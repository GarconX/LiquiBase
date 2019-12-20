## 什么是 LiquiBase

LiquiBase是针对于数据库的变更和部署管理开源工具.



## liuquibase的特性

#### 1. 数据库版本管理

它将所有数据库更改(包括结构和数据)保存在一个XML文件中，以便进行版本控制。

#### 2、支持几乎所有主流的数据库

 如MySQL, PostgreSQL, Oracle, Sql Server, DB2等

#### 3. 保存数据库变更历史 

将数据库修改历史保存在数据库中

#### 4. 数据库备份  

您可以在发布之前打开开关并备份数据库。

#### 5. 数据库比较  

它将数据库差异存储在XML中，用Liquibase可以快速的部署或升级数据库。

#### 6. 提供回滚    

已应用的更改可以通过时间、编号或标记回滚。通过这种方式，开发人员可以恢复数据库在任何时间点的状态。



## 在使用LiquiBase 之前，如何管理数据库

1. 手动更改数据库。
2. 未能与团队的其他成员共享数据库更改。
3. 使用不一致的方法来更改数据库或数据。



## 当您手动更改数据库时会发生什么?

1. 低效率。
2. 出错概率高。
3. 定位效率低的问题。
4. 没有回滚功能。



## LiquiBase的使用 

### 一、日志文件changeLog说明

changeLog是LiquiBase用来记录数据库变更的文件 , changeLog支持多种格式，主要有XML/JSON/YAML/SQL，推荐使用xml格式。

#### 1. 唯一标识

一个changeLog里面可以有多个changeSet.

一个`changeSet`标签对应一个**变更集**，由属性`id`、`author`，以及changelog的文件路径唯一标识，所以：

一个changeSet被执行之后，就不能再修改该changeSet的内容，否则下次执行会报md5加密出错。

一个changeLog日志文件在被执行之后，就不能再变更该日志文件的位置，否则可能会出错。

#### 2. 执行顺序

日志文件和日志文件中的更改集的执行顺序都是从上到下 ，如果某个日志文件执行出错，那么该日志文件以下的日志文件都不会被执行；同理，如果一个日志文件中的某个更改集执行出错，那么该更改集以下的更改集都不会被执行。

#### 3. 原子性

changeLog中的一个`changeSet`对应一个事务，在`changeSet`执行完后commit，如果出现错误则rollback



### 二、liquibase的语法

####  前置条件: `preConditions`

可以将前置条件附加到更改集，以便根据数据库的状态控制更新的执行

**preConditions**  中子标签说明：

**onFail :** 当 `sqlCheck` 中的SQL语句执行返回的值与期望值不一致时进行该操作; 

**onError :** 当 `sqlCheck` 中的SQL语句执行发生错误时的操作进行该操作。

onFail和onError的取值说明：

- `MARK_RAN` : 跳过更改集，但将其标记为已执行，以后不再执行。
- `WARN` : 输出一个警告并继续执行更改集。如果该变更集执行成功,则标记,以后每次不在会执行,否则将一直执行,并且打印报错日志和停止执行它以下的更改集。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <!--创建表news_test -->
    <changeSet author="xiejinchi" id="grab-v2.04.008_0001">
        <!-- 前置条件-->
        <preConditions onFail="MARK_RAN" onError="WARN">
            <sqlCheck expectedResult="-1">
                select count(1) from news_test;
            </sqlCheck>
        </preConditions>
        <!--创建表-->
        <createTable tableName="news_test">
            <column name="id" remarks="主键id" type="BIGINT">
                <constraints primaryKey="true"/>
            </column>
            <column name="title" remarks="标题" type="VARCHAR(64)"/>
            <column name="creator" remarks="创建者" type="VARCHAR(64)">
                <constraints nullable="false"/>
            </column>
            <column name="modifier" remarks="修改者" type="VARCHAR(64)"/>
            <column name="gmt_created" remarks="创建时间" type="datetime">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>
</databaseChangeLog>
```



#### 表的创建或删除

![1573185826868](images\1573185826868.png)



#### 列的创建和删除

![1573185945728](images\1573185945728.png)

#### 索引的创建和删除

![1573187672216](images\1573187672216.png)

#### 数据的插入和更新

![1573792977946](images\1573792977946.png)

#### 数据的删除

![image-20191109154815108](images\image-20191109154815108.png)



## LiquiBase脚本生成

#### 1、通过工具生成，这个主要是针对于相对固定的脚本生成，比如现在产品导入。

项目zatech-data-migration可以生成liquibase脚本，目前能正常使用的分支为 20191021-localuse；

关于产品的liquibase的导入说明，请参考如下文档：

> ​     Liquibase同步操作介绍 - 修改版-康家齐



#### 2、亲自手写



### 多环境配置属性: `context`

在`changeSet`变更集标签中有一个属性`context`, 可以使用该属性来指定当前变更集执行所需要的环境 , 环境在配置中心配置 ; 例如配置中心配置了 环境为 **dev3**  , 那么只有属性`context="dev3"`和没有指定属性`context`的变更集可以执行 , 如下 :

 **配置中心的配置:**![1573192068744](images\1573192068744.png)

**更改集 context属性：**

![1573192307881](images\1573192307881.png)

注意：如果配置中心没有配置 context , 则 `changeSet`中的context不起作用，所以接入新环境时，要先添加context的环境配置，再执行liquibase脚本，切记！切记！切记！

**context环境信息枚举**

配置context属性的值对应的环境如下:

dev1 : 开发环境一

dev2 : 开发环境二

dev3 : 开发环境三

test1 : 测试环境一

test2 : 测试环境二

test3 : 测试环境三

prd : 生产环境

pt : 压测环境

stg : stg环境

uat :  uat环境

## fusion接入LiquiBase配置

### 1. 使用dmds的方式

如果应用使用了**dmds**，则只需要把fusion模块需要接入LiquiBase的模块对应的appname填充进去即可

举例：dev2环境将trade模块引入

配置中心地址：http://ops-neptune-test.zhonganinfo.com/?ticket=ST-1842617-YYkbdod2mWCLgNnkKDTCyJMCETaTii2VgC0#/apps

找到应用配置（label）：zatech-liquibase-dev2，填充项如下图：

![1573196414614](images\1573196414614.png)

这里需要注意的是appNames模块名称要和脚本的命名里面的moduleName保持一致。

### 2. 使用直连数据库的方式

![1573196331889](images\1573196331889.png)



## 命名规范

### 1. 文件命名

```
|--dbscript

| |--projectName-v${version}

|    |--${systemName}

|       |--YYMMDDHHMMSS_${JIRA_No.}_${DatabaseName}_ddl.xml

|       |--YYMMDDHHMMSS_${JIRA_No.}_${DatabaseName}_dml.xml
```

**例如**

![image-20191109190922214](\images\image-20191109190922214.png)



其中version是对应测试环境应用发布最近的版本号，这个配管会发邮件，例如

![1573204136296](images\1571982053542.png)

### 2. 文件的执行顺序

liquibase是按照文件名字前面的时间戳顺序执行的，所以如果大家有执行顺序要求的，记得一定写对时间戳 ;

同一个文件里的changeset的顺序是由系统定义的，一般是自上向下执行，这里还是建议有顺序要求的要写成两个文件用时间戳区分； 

对同一个库的操作，不同文件名时间戳要区分开来。



## 更多信息参考 

内部wiki：https://wiki.zhonganonline.com/pages/viewpage.action?pageId=36047162 

liquibase官网：http://www.liquibase.org/documentation/index.html