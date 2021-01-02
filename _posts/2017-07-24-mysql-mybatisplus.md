---
layout: post
title: Mysql 07 Mybatis-plus
categories: [Mysql]
description: Mybatis-plus
keywords: mysql, Mybatis-plus
topmost: false
---



Mybatis-plus 是 Mybatis Mysql 重要增强，详细细节大家可以看官网，这里会记录一些实际使用经验

#### 1.理念

只做增强不做改变，引入不会对现有工程造成任何影响  
简单配置，快速CRUD操作  
热加载、代码生成、分页、性能分析 功能全面

![mybatisplus1](/images/posts/2017-07-24-mysql-mybatisplus/mybatisplus1.png)

#### 2.特性

- 无侵入，只增强不改变，损耗小，强大CRUD   

- 支持Lambda形式调用

- 支持主键自动生成：内含分布式唯一ID生成器-Sequence

- 支持ActiveRecord模式，实体类只需继承Model类可进行强大CRUD操作 

- 自定义全局通用操作

- 内置代码生成器

- 内置分页插件

- 分页插件支持多种数据库

- 内置性能分析插件

- 内置全局拦截插件

#### 3.QueryWrapper

案例1：

```java
WHERE
    (
        (
            (year = ? AND week >= ?)
            OR (year = ? AND week <= ?)
        )
        AND city_name = ?
        AND user_net_type = ?
        AND module_name = ?
        AND (
            subject_cname LIKE ?
            OR subject_ename LIKE ?
        )
    )
ORDER BY
    year DESC,
    week DESC
```

```java
/**
     * @param pageNum     当前页
     * @param pageSize    每页条数
     * @param moduleName  产品
     * @param userNetType 运营商
     * @param cityName    城市名称
     * @param beginTime   开始周的任意一天日期（例如：2019-12-20）
     * @param endTime     结束周的任意一天日期（例如：2020-01-20）
     * @param keyWord     查询条件（专题英文名或者中文名）
     * @return
     */
        //开始年份
        String beginYear = null;
        //结束年份
        String endYear = null;
        //开始周数
        String beginWeek = null;
        //结束周数
        String endWeek = null;
        /*这部分内容忽略，调用了其他的方法，
         反正就是为了获取开始日期所在的年份、周数以及结束日期所在的年份、周数*/
        if (StringUtils.isBlank(beginTime) || StringUtils.isBlank(endTime)) {
            DateTime dateTime = DateUtil.lastWeek();
            //格式化日期，结果：yyyyMMdd
            beginTime = DateUtil.formatDate(dateTime);
            beginYear = TimeUtils.getDateOfYearWeek(beginTime).get("year");
            endYear = beginYear;
            beginWeek = TimeUtils.getDateOfYearWeek(beginTime).get("week");
            endWeek = beginWeek;
        } else {
            beginYear = TimeUtils.getDateOfYearWeek(beginTime).get("year");
            endYear = TimeUtils.getDateOfYearWeek(endTime).get("year");
            beginWeek = TimeUtils.getDateOfYearWeek(beginTime).get("week");
            endWeek = TimeUtils.getDateOfYearWeek(endTime).get("week");
        }
        Page<DwSubjectDataInfoWw> page = new Page<>(pageNum, pageSize);
        LambdaQueryWrapper<DwSubjectDataInfoWw> queryWrapper = Wrappers.<DwSubjectDataInfoWw>lambdaQuery();
        if (beginYear.equals(endYear)) {
            queryWrapper.eq(DwSubjectDataInfoWw::getYear, beginYear);
            queryWrapper.between(DwSubjectDataInfoWw::getWeek, beginWeek, endWeek);
        } else {
            //因为Java8 Lambda表达式中最终变量问题，重新赋值一个参数解决
            String year1 = beginYear;
            String year2 = endYear;
            String week1 = beginWeek;
            String week2 = endWeek;
            queryWrapper
	.and( wrapper -> wrapper.and( wrapper1 -> wrapper1.eq(DwSubjectDataInfoWw::getYear, year1).ge(DwSubjectDataInfoWw::getWeek, week1))
      .or(wrapper2 -> wrapper2.eq(DwSubjectDataInfoWw::getYear, year2).le(DwSubjectDataInfoWw::getWeek, week2)));
        }
        queryWrapper.orderByDesc(DwSubjectDataInfoWw::getYear);
        queryWrapper.orderByDesc(DwSubjectDataInfoWw::getWeek);

        if (StrUtil.isNotEmpty(cityName)) {
            queryWrapper.eq(DwSubjectDataInfoWw::getCityName, cityName);
        }
        if (StrUtil.isNotEmpty(userNetType)) {
            queryWrapper.eq(DwSubjectDataInfoWw::getUserNetType, userNetType);
        }
        if (StrUtil.isNotEmpty(moduleName)) {
            queryWrapper.eq(DwSubjectDataInfoWw::getModuleName, moduleName);
        }
        //搜索条件可以是专题中文名或英文名
        if (StrUtil.isNotEmpty(keyWord)) {
            queryWrapper.and(wrapper -> wrapper.like(DwSubjectDataInfoWw::getSubjectCname, keyWord).or().like(DwSubjectDataInfoWw::getSubjectEname, keyWord));
        }
        try {
            Page<DwSubjectDataInfoWw> list = dwSubjectDataInfoWwService.page(page, queryWrapper);
            return AjaxResult.success(list);
        } catch (Exception e) {
            logger.error("获取分周专题数据列表错误,错误信息为:", e);
            return AjaxResult.error();
        }
```

案例2：

```
WHERE is_deleted = 0 AND (is_global = ? OR (tenant_id = ?))

queryWrapper.lambda()
	.eq(Role::getIsGlobal, BladeTenantProperties.globalColumnPublicValue)
	.or(wrapper -> wrapper.eq(Role::getTenantId, bladeUser.getTenantId()));
```

#### 4.${ew.customSqlSegment} 自定义SQL

1.注解

```java
${ew.customSqlSegment} 
@Param(Constants.WRAPPER)

@Select("select * from mysql_data ${ew.customSqlSegment}")
List<MysqlData> getAll(@Param(Constants.WRAPPER) Wrapper wrapper);
```
2.xml--重点

```
1.mapper.java定义接口
public List<User> getUserHasRole(
	@Param(Constants.WRAPPER) QueryWrapper wrapper);

2.mapper.xml定义SQL
<select id="getUserHasRole" 
	resultType="czc.superzig.modular.system.model.User">
	select * FROM sys_user <where>${ew.sqlSegment}</where>
</select>
```

![mysql1](/images/posts/2017-07-24-mysql-mybatisplus/mysql1.png)

![mysql2](/images/posts/2017-07-24-mysql-mybatisplus/mysql2.png)

![mysql3](/images/posts/2017-07-24-mysql-mybatisplus/mysql3.png)













## 参考：

https://github.com/MiniPa/mybatis-plus 
https://github.com/baomidou/mybatis-plus-doc  
https://mp.baomidou.com/guide/
