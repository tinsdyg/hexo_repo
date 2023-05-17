---
title: Mysql之JSON操作
tags: study
---
# 一. 通过Mybatis存储JSON

> json的存储本质上还是一种字符串,至少针对这种特殊的字符串mysql提供了一系列的函数进行了支持,如增加,追加,修改,删除等操作.

存储Mysql不支持的类型时需要自定义实现一个TypeHandler的类,实现存取时的类型转换,常见的基本类型Mybatis已经帮我们实现了,代码如下

```java
public class JsonTypeHandler extends BaseTypeHandler<List<QuestInfo>> {  

    @Override  
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, List<QuestInfo> questInfos, JdbcType jdbcType) throws SQLException {  
        preparedStatement.setString(i, JSONObject.toJSONString(questInfos));    
    }  
  
    @Override  
    public List<QuestInfo> getNullableResult(ResultSet resultSet, String s) throws SQLException {  
        if(resultSet==null) return null;  
        return JSONObject.parseArray(resultSet.getString(s),QuestInfo.class) ;  
    }  
  
    @Override  
    public List<QuestInfo> getNullableResult(ResultSet resultSet, int i) throws SQLException {  
  
        if(resultSet==null) return null;  
  
        return JSONObject.parseArray(resultSet.getString(i),QuestInfo.class);  
    }  
  
    @Override  
    public List<QuestInfo> getNullableResult(CallableStatement callableStatement, int i) throws SQLException {  
        return JSONObject.parseArray(callableStatement.getString(i),QuestInfo.class);  
    }  
}
```



# 二. Mysql中内置的JSON操作函数

## 1. JSON的添加
> 指定JSON对象或者数组的下标即可向该对象或者数组添加对象或值
> 当没有这个属性时会创建

```sql
update 表名 
set 字段x = JSON_SET(字段x, '$[下表].属性', 值)  
where 字段y = 参数;
```

## 2.JSON的追加
> 指定JSON对象或者数组的下标即可向该对象或者数组追加对象或值

```sql
update 表名  
set 字段x = JSON_ARRAY_APPEND(字段x, '$ <被添加的地方,这里是根>', cast('{"no":  4,"qid": 60} <被添加的json>' as json))  
where 字段y = 参数;

```


## 3.JSON的修改
> 传入JSON属性名或者是下标,即可修改该属性的值
> 当被修改的地方不存在时,不会有任何操作,返回成功

```sql
update 表名  
set 字段x = JSON_REPLACE(字段x, '$被修改的地方', '修改后的值')  
where 字段y = 参数;

```


## 4.JSON的删除
> 指定JSON的属性或者数组的下标,即可删除改属性和值或者是该对象

```sql
update 表名  
set 字段x = JSON_REMOVE(字段x, '$[下标]')  
where 字段y = 参数;

```


