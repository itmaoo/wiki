#### 简介
Mycat是分布式数据库管理中间件, 用户使用起来无感,如使用MySQL部署集群, 和单独用一个MySQL数据一样的用法

#### 特性
1. 支持分库分表, 纵向和横向数据切分
2. 独立服务,类似数据库代理
3. 支持读写分离


#### 功能模块

|模块|描述|
|-|-|
|Schema| schema.xml独立标签, 定义逻辑库|
|Table|schema.xml从属Schema标签, 定义逻辑表|
|DataNode|schema.xml独立标签, 分片是库的别名, 被tabl引用|
|DataHost|schema.xml独立标签, 配置物理数据库,被DataNode引用|
|tableRule|rule.xml独立标签, 配置分片规则,被table引用|
|function|rule.xml独立标签, 配置分片算法,被tableRule引用|

schema.xml 配置样例

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">

<mycat:schema  xmlns:mycat="http://org.opencloudb/">
  <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" dataNode="dn1">
      <table name="t_user" dataNode="dn1,dn2" rule="sharding-by-mod2"/>
      <table name="ht_jy_login_log" primaryKey="ID" dataNode="dn1,dn2" rule="sharding-by-date_jylog"/>
  </schema>
  <dataNode name="dn1" dataHost="localhost1" database="mycat_node1"/>
  <dataNode name="dn2" dataHost="localhost1" database="mycat_node2"/>
  
  <dataHost name="localhost1" writeType="0" switchType="1" slaveThreshold="100" balance="1" dbType="mysql" maxCon="10" minCon="1" dbDriver="native">
    <heartbeat>show status like 'wsrep%'</heartbeat>
    <writeHost host="hostM1" url="127.0.0.1:3306" user="root" password="root" >
    </writeHost>  
  </dataHost>
</mycat:schema >
```
server.xml 配置样例
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://org.opencloudb/">
   <system>
	    <property name="defaultSqlParser">druidparser</property>
    </system>
    <user name="mycat">
	    <property name="password">mycat</property>
	    <property name="schemas">TESTDB</property>
    </user>
 </mycat:server>
```
rule.xml配置样例
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:rule SYSTEM "rule.dtd">

<mycat:rule  xmlns:mycat="http://org.opencloudb/">
  <tableRule name="sharding-by-hour">
    <rule>
      <columns>createTime</columns>
      <algorithm>sharding-by-hour</algorithm>
    </rule>
  </tableRule>
  
  <function name="sharding-by-hour" class="org.opencloudb.route.function.LatestMonthPartion">
    <property name="splitOneDay">24</property>
  </function>
   
</mycat:rule >
```
