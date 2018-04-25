# Setup for Workshop

## Downloading our components

[Download DSE 6.0.0](https://portal.datastax.com/downloads.php?dsedownload=tar/enterprise/dse.tar.gz)

[Download Apache Zeppelin 0.7.3](http://mirrors.gigenet.com/apache/zeppelin/zeppelin-0.7.3/zeppelin-0.7.3-bin-netinst.tgz)

## Let's start by Setting up DSE

    tar -xvf tar -xvf dse-6.0.0-bin.tar.gz
    cd dse-6.0.0
    
### Turn on AOSS
    
    # AlwaysOn SQL options have dependence on workpool setting of resource_manager_options. Set workpool configuration if you
    # enable alwayson_sql_options.
    alwayson_sql_options:
    #     # Set to true to enable the node for AlwaysOn SQL. Only an Analytics node
    #     # can be enabled as an AlwaysOn SQL node.
          enabled: true

### Start up DSE

    ./bin/dse -k -g
    
### Test out our DSE Connection

    ./bin/cqlsh
    
    CREATE KEYSPACE test WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1 };
    use test;
    CREATE TABLE tab ( k int, c int, v int, PRIMARY KEY (k,c));
    INSERT INTO tab (k , c , v ) VALUES ( 1, 1, 1) ;
    SELECT * FROM test.tab;
    
     k | c | v
    ---+---+---
     1 | 1 | 1
     
 ### Test out our Always On SQL Server Connection
 
     ./bin/dse beeline
     !connect jdbc:hive2://127.0.0.1:10000
     
     # When it asks for a user type in anything, leave password blank
     
      1: jdbc:hive2://127.0.0.1:10000> SELECT * FROM test.tab;
     +----+----+----+--+
     | k  | c  | v  |
     +----+----+----+--+
     | 1  | 1  | 1  |
     +----+----+----+--+
     
  Congratulations you have just run a distributed Spark Job.

----

Additional Details for Cassandra Novices
 
### What does our Cassandra Table look like?

    k :: partition key
    c :: clustering key
    v :: a value

On disk this looks like

    k1 -> (c1,v1) , (c2,v2), (c3,v3)
    k2 -> (c1,v1) , (c2,v2), (c3,v3)
   
    
#### Important Cassandra Concepts
For more information on Cassandra and Data Layout
* Tokens : Where data lives
* DataModeling : How data is laid out on disk
* Replication : How many copies of the data will there be on the Server
* Consistency Level : How many acknowledgements the Client needs for success  

Study more later!
[Datastax Academy](https://academy.datastax.com/)
