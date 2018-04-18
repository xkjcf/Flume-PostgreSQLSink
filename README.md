Flume-PostgreSQLSink
=============
A library of Flume sinks for postgreSQL

## Getting Started
----
1. Clone the repository
2. Install latest Maven and build source by 'mvn package'
3. Generate classpath by 'mvn dependency:build-classpath'
4. Append classpath in $FLUME_HOME/conf/flume-env.sh
5. Add the sink definition according to **Configuration**

## Source data format:
- - - 
    Source data format must be JSON string, for example:
          
    {
        "data": {
            "field_name1": value1,
            "field_name2": value2,
            "field_name3": value3
        } 
    }
    
## Configuration
- - - 
    type: org.frank.flume.sink.PostgresSink
    host: db host IP [localhost]
    port: db port [5432]
    db  : postgresql db name
    username: = db username 	[]
    password: = db user password []
    batch: batch size of insert opertion [100]
    model: = auto/by_field/custom
             [auto]: Means every event will specify the table name by the field 'log_usr' in the event JSON string.
             [by_column_name]: Means every event will specify the table name by the field which specify by 'table_name_by'.
             [custom]: Means every event will specify the table name as you like.
    table_name_by: Required  when "model" is 'by_field', and the table name will be specifed by the value of this field.
    table_name: Required when "model" is 'custom', you can specifed table name as you like.
    columns: The number of columns write to postgresql. 
    nameN (N = 1,2,3...): The column name.
    typeN (N = 1,2,3...): The column type. The value can be: {text, varchar, char, numberic, integer, int, smallint, bigint, real, float, double, double precison, date, time, timestamp}
   
## flume.conf sample
- - - 
    agent.sources = source1  
    agent.sinks = sink1  
    agent.channels = channel1  
     
    agent.sources.source1.type = netcat  
    agent.sources.source1.bind = 192.168.1.2
    agent.sources.source1.port = 9000  
      
    agent.sinks.sink1.type = org.frank.flume.sink.PostgresSink
    agent.sinks.sink1.host = 192.168.1.1
    agent.sinks.sink1.port = 5432
    agent.sinks.sink1.batch = 100
    agent.sinks.sink1.db = postgres
    agent.sinks.sink1.username = test	
    agent.sinks.sink1.password = testpassword
    agent.sinks.sink1.model = by_column_name
    agent.sinks.sink1.table_name_by = log_usr
      
    agent.sinks.sink1.num.columns = 3
    agent.sinks.sink1.column.name1 = time
    agent.sinks.sink1.column.type1 = timestamp
    agent.sinks.sink1.column.name2 = log_usr
    agent.sinks.sink1.column.type2 = varchar
    agent.sinks.sink1.column.name3 = src_ip
    agent.sinks.sink1.column.type3 = bigint
      
    agent.channels.channel1.type = memory  
    agent.channels.channel1.capacity = 1000  
    agent.channels.channel1.transactionCapacity = 100 
      
    agent.sources.source1.channels = channel1  
    agent.sinks.sink1.channel = channel1  

## 执行数据类型时的映射关系（对大小写不敏感）

```
    数据类型          ==>  指定值
    Types.VARCHAR    ==>  VARCHAR
    Types.VARCHAR    ==>  TEXT
    Types.CHAR       ==>  CHAR
    Types.NUMERIC    ==>  NUMERIC
    Types.INTEGER    ==>  INTEGER
    Types.INTEGER    ==>  INT
    Types.BIGINT     ==>  SMALLINT
    Types.REAL       ==>  BIGINT
    Types.FLOAT      ==>  REAL
    Types.FLOAT      ==>  FLOAT
    Types.DOUBLE     ==>  DOUBLE
    Types.DOUBLE     ==>  DOUBLE PRECISION
    Types.DATE       ==>  DATE
    Types.TIME       ==>  TIME
    Types.TIMESTAMP  ==>  TIMESTAMP
```

## 指定表明的方法

1. 最简单的方案，手动配置表明

```
    agent.sinks.sink1.model = custom
    agent.sinks.sink1.table_name = data
```

 2. 还支持在数据中指定该数据需要插入的表名

```
    agent.sinks.sink1.model = by_column_name
    agent.sinks.sink1.table_name_by = check_table_name
```
这这种方法之下，需要传入的数据格式是：

```
    {
        "data": {
            "check_table_name": "data{{ 修改为你要存放的表名 }}",
            ……
        } 
    }
```
## 注意

传入的数据中，即使是int类型等，也需要用引号引起来，程序中会以字符串的形式提取，在强制转换成配置文件中配置的数据类型。
我的实验样例
```
    { "data": {"field1": "value1", "field2": "121", "createdtime": "2018-12-12 12:12:12", "time": "1235623545" } }
```
## About the director jar_libs
- - - 
    Copy the json-simple-1.1.1.jar and postgresql-9.4.1208.jre7.jar to the path 'xxx/flume/lib/'.And you can update these jar as necessary.
