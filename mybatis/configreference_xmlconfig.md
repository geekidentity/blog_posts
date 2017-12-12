---
categories: mybatis

tags: 
  - Mybatis Generator
  - Mybatis
  - J2EE
  - 翻译


title: MyBatis GeneratorXML配置文件参考

date: 2017-05-06
---

# MyBatis GeneratorXML配置文件参考

在最常见的用例中，MyBatis Generator（MBG）由XML配置文件驱动。 配置文件告诉MBG：

* 如何连接到数据库
* 什么对象要生成，以及如何生成它们
* 什么表应用于生成对象

以下是一个示例MBG配置文件。 有关元素和属性值的更多信息，请参阅每个元素的各个页面。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />

  <context id="DB2Tables" targetRuntime="MyBatis3">
    <jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
        connectionURL="jdbc:db2:TEST"
        userId="db2admin"
        password="db2admin">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="test.model" targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="test.xml"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="\MBGTestProject\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

    <table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >
      <property name="useActualColumnNames" value="true"/>
      <generatedKey column="ID" sqlStatement="DB2" identity="true" />
      <columnOverride column="DATE_FIELD" property="startDate" />
      <ignoreColumn column="FRED" />
      <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
    </table>

  </context>
</generatorConfiguration>
```

关于此文件的重要注意事项如下：

* 该文件指定将使用旧版DB2 CLI驱动程序连接到数据库，并指定可以在哪里找到驱动程序。
* Java Type Resolver不应强制使用BigDecimal字段 - 这意味着如果可能，整数类型（Short，Integer，Long等）将被替换。 此功能是使数据库DECIMAL和NUMERIC列更容易处理。
* Java模型生成器应该使用子包。 这意味着在这种情况下，生成的模型对象将被放置在名为test.model.db2admin的包中（因为该表位于DB2ADMIN模式中）。 如果enableSubPackages属性设置为false，那么该包将是test.model。 Java模型生成器也应该修剪字符串。 这意味着任何String属性的setter将调用trim函数 - 如果数据库可能在字符列末尾返回空白字符，这很有用。
* SQL Map生成器应该使用子包。 这意味着在这种情况下，生成的XML文件将被放置在名为test.xml.db2admin的包中（因为表位于DB2ADMIN模式中）。 如果enableSubPackages属性设置为false，那么该包将是test.xml。
* DAO生成器应该使用子包。 这意味着在这种情况下，生成的DAO类将被放置在名为test.dao.db2admin的包中（因为表位于DB2ADMIN模式中）。 如果enableSubPackages属性设置为false，那么该包将是test.dao。 DAO生成器应生成引用MyBatis的XML配置的映射器接口。
* 该文件指定只有一个表将被内省，但更多可以指定。 关于指定表的重要注意事项包括：
    * 生成的对象将基于Customer（CustomerKey，Customer，CustomerMapper等） - 而不是表名。
    * 实际列名称将用作属性。 如果此属性设置为false（或未指定），则MBG将尝试将列名称加载。 在这两种情况下，名称可以被<columnOverride>元素覆盖
    * 该列具有生成的密钥，它是一个标识列，数据库类型是DB2。 这将导致MBG在生成的<insert>语句中生成正确的<selectKey>元素，以便可以返回新生成的密钥（使用DB2特定的SQL）。
    * 列DATE_FIELD将映射到一个名为startDate的属性。 这将覆盖在这种情况下为DATE_FIELD的默认属性，如果useActualColumnNames属性设置为false，则为dateField。
    * FRED列将被忽略。 没有SQL将列出该字段，并且不会生成Java属性。
    * 列LONG_VARCHAR_FIELD将被视为VARCHAR字段，而不考虑实际的数据类型。