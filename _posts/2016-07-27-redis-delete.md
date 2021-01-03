---
layout: post
title: Redis 09 淘汰机制-3 删除策略-2 
categories: [Redis]
description: Redis 淘汰机制-3 删除策略-2 
keywords: Redis
topmost: false
---

#### 1.Redis 过期时间存储

redis的过期时间存储在 **db->expires** 的对象当中

##### 设置过期时间的过程：

- 1.从db-dict获取原来存储数据，保证key的存在性 

- 2.从db->expires获取旧的过期事件并重新计算过期时间dictReplaceRaw 

- 3.将过期时间重新保存到DictEntry当中，也就是db->expires中的某个对象

```java
/*
 * 将键 key 的过期时间设为 when
 */
void setExpire(redisDb *db, robj *key, long long when) {

    dictEntry *kde, *de;

    /* Reuse the sds from the main dict in the expire dict */
    // 取出键
    kde = dictFind(db->dict,key->ptr);

    redisAssertWithInfo(NULL,key,kde != NULL);

    // 根据键取出键的过期时间
    de = dictReplaceRaw(db->expires,dictGetKey(kde));

    // 设置键的过期时间
    // 这里是直接使用整数值来保存过期时间，不是用 INT 编码的 String 对象
    dictSetSignedIntegerVal(de,when);
}
```

##### redis 数据淘汰过程：

- 1.遍历所有的db进行数据的释放 

- 2.根据不同的策略选择从db.dict还是从db.expires选择**待过期数据** 

- 3.区分不同的淘汰策略选择不同的key，主要分为**随机**淘汰、**LRU**淘汰、**TTL**时间淘汰

```java
int freeMemoryIfNeeded(void) {
    /* Compute how much memory we need to free. */
    // 计算需要释放多少字节的内存
    mem_tofree = mem_used - server.maxmemory;

    // 初始化已释放内存的字节数为 0
    mem_freed = 0;

    // 根据 maxmemory 策略，
    // 遍历字典，释放内存并记录被释放内存的字节数
    while (mem_freed < mem_tofree) {
        int j, k, keys_freed = 0;

        // 遍历所有字典
        for (j = 0; j < server.dbnum; j++) {
            long bestval = 0; /* just to prevent warning */
            sds bestkey = NULL;
            dictEntry *de;
            redisDb *db = server.db+j;
            dict *dict;

            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM)
            {
                // 如果策略是 allkeys-lru 或者 allkeys-random 
                // 那么淘汰的目标为所有数据库键
                dict = server.db[j].dict;
            } else {
                // 如果策略是 volatile-lru 、 volatile-random 或者 volatile-ttl 
                // 那么淘汰的目标为带过期时间的数据库键
                dict = server.db[j].expires;
            }

            // 跳过空字典
            if (dictSize(dict) == 0) continue;

            /* volatile-random and allkeys-random policy */
            // 如果使用的是随机策略，那么从目标字典中随机选出键
            if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_RANDOM)
            {
                de = dictGetRandomKey(dict);
                bestkey = dictGetKey(de);
            }

            /* volatile-lru and allkeys-lru policy */
            // 如果使用的是 LRU 策略，
            // 那么从一集 sample 键中选出 IDLE 时间最长的那个键
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
                server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
            {
                struct evictionPoolEntry *pool = db->eviction_pool;

                while(bestkey == NULL) {
                    evictionPoolPopulate(dict, db->dict, db->eviction_pool);
                    /* Go backward from best to worst element to evict. */
                    for (k = REDIS_EVICTION_POOL_SIZE-1; k >= 0; k--) {
                        if (pool[k].key == NULL) continue;
                        de = dictFind(dict,pool[k].key);

                        /* Remove the entry from the pool. */
                        sdsfree(pool[k].key);
                        /* Shift all elements on its right to left. */
                        memmove(pool+k,pool+k+1,
                            sizeof(pool[0])*(REDIS_EVICTION_POOL_SIZE-k-1));
                        /* Clear the element on the right which is empty
                         * since we shifted one position to the left.  */
                        pool[REDIS_EVICTION_POOL_SIZE-1].key = NULL;
                        pool[REDIS_EVICTION_POOL_SIZE-1].idle = 0;

                        /* If the key exists, is our pick. Otherwise it is
                         * a ghost and we need to try the next element. */
                        if (de) {
                            bestkey = dictGetKey(de);
                            break;
                        } else {
                            /* Ghost... */
                            continue;
                        }
                    }
                }
            }

            /* volatile-ttl */
            // 策略为 volatile-ttl ，从一集 sample 键中选出过期时间距离当前时间最接近的键
            else if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_TTL) {
                for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    thisval = (long) dictGetVal(de);

                    /* Expire sooner (minor expire unix timestamp) is better
                     * candidate for deletion */
                    if (bestkey == NULL || thisval < bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
            }

            /* Finally remove the selected key. */
            // 删除被选中的键
            if (bestkey) {
                long long delta;

                robj *keyobj = createStringObject(bestkey,sdslen(bestkey));
                propagateExpire(db,keyobj);
                /* We compute the amount of memory freed by dbDelete() alone.
                 * It is possible that actually the memory needed to propagate
                 * the DEL in AOF and replication link is greater than the one
                 * we are freeing removing the key, but we can't account for
                 * that otherwise we would never exit the loop.
                 *
                 * AOF and Output buffer memory will be freed eventually so
                 * we only care about memory used by the key space. */
                // 计算删除键所释放的内存数量
                delta = (long long) zmalloc_used_memory();
                dbDelete(db,keyobj);
                delta -= (long long) zmalloc_used_memory();
                mem_freed += delta;
                
                // 对淘汰键的计数器增一
                server.stat_evictedkeys++;

                notifyKeyspaceEvent(REDIS_NOTIFY_EVICTED, "evicted",
                    keyobj, db->id);
                decrRefCount(keyobj);
                keys_freed++;

                /* When the memory to free starts to be big enough, we may
                 * start spending so much time here that is impossible to
                 * deliver data to the slaves fast enough, so we force the
                 * transmission here inside the loop. */
                if (slaves) flushSlavesOutputBuffers();
            }
        }

        if (!keys_freed) return REDIS_ERR; /* nothing to free... */
    }

    return REDIS_OK;
}
```



#### 2.Redis 淘汰策略

##### 1.**随机淘汰**

获取待删除key的策略，随机找hash桶再次hash指定位置的dictEntry即可

```java
/*
 * 随机返回字典中任意一个节点。
 *
 * 可用于实现随机化算法。
 *
 * 如果字典为空，返回 NULL 。
 *
 * T = O(N)
*/
dictEntry *dictGetRandomKey(dict *d)
{
    dictEntry *he, *orighe;
    unsigned int h;
    int listlen, listele;

    // 字典为空
    if (dictSize(d) == 0) return NULL;

    // 进行单步 rehash
    if (dictIsRehashing(d)) _dictRehashStep(d);

    // 如果正在 rehash ，那么将 1 号哈希表也作为随机查找的目标
    if (dictIsRehashing(d)) {
        // T = O(N)
        do {
            h = random() % (d->ht[0].size+d->ht[1].size);
            he = (h >= d->ht[0].size) ? d->ht[1].table[h - d->ht[0].size] :
                                      d->ht[0].table[h];
        } while(he == NULL);
    // 否则，只从 0 号哈希表中查找节点
    } else {
        // T = O(N)
        do {
            h = random() & d->ht[0].sizemask;
            he = d->ht[0].table[h];
        } while(he == NULL);
    }

    /* Now we found a non empty bucket, but it is a linked
     * list and we need to get a random element from the list.
     * The only sane way to do so is counting the elements and
     * select a random index. */
    // 目前 he 已经指向一个非空的节点链表
    // 程序将从这个链表随机返回一个节点
    listlen = 0;
    orighe = he;
    // 计算节点数量, T = O(1)
    while(he) {
        he = he->next;
        listlen++;
    }
    // 取模，得出随机节点的索引
    listele = random() % listlen;
    he = orighe;
    // 按索引查找节点
    // T = O(1)
    while(listele--) he = he->next;

    // 返回随机节点
    return he;
}
```

##### 2.**LRU**

- 1.dictGetRandomKeys随机获取指定数目的dictEntry 
- 2.将获取的的dictEntry进行下sort按照最近时间进行排序 
- 3.选择最近使用时间最久远的数据进行过期 
- 4.每次过期的数据其实是采样的结果数据中的最近未被访问数据而非全局的

```java
void evictionPoolPopulate(dict *sampledict, dict *keydict, struct evictionPoolEntry *pool) {
    int j, k, count;
    dictEntry *_samples[EVICTION_SAMPLES_ARRAY_SIZE];
    dictEntry **samples;

    /* Try to use a static buffer: this function is a big hit...
     * Note: it was actually measured that this helps. */
    if (server.maxmemory_samples <= EVICTION_SAMPLES_ARRAY_SIZE) {
        samples = _samples;
    } else {
        samples = zmalloc(sizeof(samples[0])*server.maxmemory_samples);
    }

#if 1 /* Use bulk get by default. */
    count = dictGetRandomKeys(sampledict,samples,server.maxmemory_samples);
#else
    count = server.maxmemory_samples;
    for (j = 0; j < count; j++) samples[j] = dictGetRandomKey(sampledict);
#endif

    for (j = 0; j < count; j++) {
        unsigned long long idle;
        sds key;
        robj *o;
        dictEntry *de;

        de = samples[j];
        key = dictGetKey(de);
        /* If the dictionary we are sampling from is not the main
         * dictionary (but the expires one) we need to lookup the key
         * again in the key dictionary to obtain the value object. */
        if (sampledict != keydict) de = dictFind(keydict, key);
        o = dictGetVal(de);
        idle = estimateObjectIdleTime(o);

        /* Insert the element inside the pool.
         * First, find the first empty bucket or the first populated
         * bucket that has an idle time smaller than our idle time. */
        k = 0;
        while (k < REDIS_EVICTION_POOL_SIZE &&
               pool[k].key &&
               pool[k].idle < idle) k++;
        if (k == 0 && pool[REDIS_EVICTION_POOL_SIZE-1].key != NULL) {
            /* Can't insert if the element is < the worst element we have
             * and there are no empty buckets. */
            continue;
        } else if (k < REDIS_EVICTION_POOL_SIZE && pool[k].key == NULL) {
            /* Inserting into empty position. No setup needed before insert. */
        } else {
            /* Inserting in the middle. Now k points to the first element
             * greater than the element to insert.  */
            if (pool[REDIS_EVICTION_POOL_SIZE-1].key == NULL) {
                /* Free space on the right? Insert at k shifting
                 * all the elements from k to end to the right. */
                memmove(pool+k+1,pool+k,
                    sizeof(pool[0])*(REDIS_EVICTION_POOL_SIZE-k-1));
            } else {
                /* No free space on right? Insert at k-1 */
                k--;
                /* Shift all elements on the left of k (included) to the
                 * left, so we discard the element with smaller idle time. */
                sdsfree(pool[0].key);
                memmove(pool,pool+1,sizeof(pool[0])*k);
            }
        }
        pool[k].key = sdsdup(key);
        pool[k].idle = idle;
    }
    if (samples != _samples) zfree(samples);
}
```

通过random() & d->ht[j].sizemask方法随机获取从某个hash桶开始随机获取dictEntry。获取的待淘汰的数据通过count进行指定

```java
int dictGetRandomKeys(dict *d, dictEntry **des, int count) {
    int j; /* internal hash table id, 0 or 1. */
    int stored = 0;

    if (dictSize(d) < count) count = dictSize(d);
    while(stored < count) {
        for (j = 0; j < 2; j++) {
            /* Pick a random point inside the hash table 0 or 1. */
            unsigned int i = random() & d->ht[j].sizemask;
            int size = d->ht[j].size;

            /* Make sure to visit every bucket by iterating 'size' times. */
            while(size--) {
                dictEntry *he = d->ht[j].table[i];
                while (he) {
                    /* Collect all the elements of the buckets found non
                     * empty while iterating. */
                    *des = he;
                    des++;
                    he = he->next;
                    stored++;
                    if (stored == count) return stored;
                }
                i = (i+1) & d->ht[j].sizemask;
            }
            /* If there is only one table and we iterated it all, we should
             * already have 'count' elements. Assert this condition. */
            assert(dictIsRehashing(d) != 0);
        }
    }
    return stored; /* Never reached. */
}
```

##### 3.**TTL** 基于采样结果进行选择然后选择距离过期时间最近的数据进行过期

```java
for (k = 0; k < server.maxmemory_samples; k++) {
                    sds thiskey;
                    long thisval;

                    de = dictGetRandomKey(dict);
                    thiskey = dictGetKey(de);
                    thisval = (long) dictGetVal(de);

                    /* Expire sooner (minor expire unix timestamp) is better
                     * candidate for deletion */
                    if (bestkey == NULL || thisval < bestval) {
                        bestkey = thiskey;
                        bestval = thisval;
                    }
                }
```



#### 3.删除策略

- 1.惰性删除：redis处理读写请求时，如get/set

- 2.定期删除：redis内部定时任务执行过程中，限制占用cpu时间  
  定时：定时任务会循环调用serverCron方法，然后定时检查过期数据的方法是databasesCron   
  特点：每次执行databasesCron的时候会限制cpu的占用**不超过25%**

[Blog](https://www.jianshu.com/p/35a579c43d4f)


















## 参考：

