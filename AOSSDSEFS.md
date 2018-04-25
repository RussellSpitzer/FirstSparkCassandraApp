# DSEFS

## DSE FS Commands

DSEFS is compatible with Hadoop Style FS commands

First make sure DSEFS Metadata is being replicated correctly



cqlsh
     
     ALTER KEYSPACE dsefs WITH replication = {'class': 'NetworkTopologyStrategy', 'GraphAnalytics':1};
     ALTER KEYSPACE dse_analytics WITH replication = {'class': 'NetworkTopologyStrategy', 'GraphAnalytics':1};
     ALTER KEYSPACE dse_security WITH replication = {'class': 'NetworkTopologyStrategy', 'GraphAnalytics':1};
     ALTER KEYSPACE dse_leases WITH replication = {'class': 'NetworkTopologyStrategy', 'GraphAnalytics':1};
     
You may need to reboot at this point

Make a directory and put a file from this repo into DSEFS

    ./bin/dse fs "mkdir /data"
    #Change the Directory in the below line to match your system
    ./bin/dse fs 'put /Users/russellspitzer/repos/FirstSparkCassandraApp.git/data/important.csv /data/important.csv'

## DSEFS Shell

Using raw commands is fun but DSE also comes with a mini shell for exploring
dsefs

    ./bin/dse fs
    ls
    # data  spark  tmp 
    cat data/important.csv
    # Sundance,dog,10,brown
    # Cara,dog,10,white
    # Mr. Pants,cat,12,brown
    # HD,dog,2,brown
    # Benny,dog,6,white
    # Tony,tiger,45,orange
    
Lets make a new directory in the shell to hold parquet (a columnar data format) files

    mkdir my_parquet
    
## Always Online SQL Server 

Making parquet files (or changing between any formats) is easy in Spark. Just about
anything you can do in Spark programattically can also be done via Spark SQL and 
the AOSS. 

Go back to beeline and reconnect if you lost the connection

    ./bin/dse beeline
    !connect jdbc:hive2://127.0.0.1:10000
    
We saw previously that our C* tables were automatically registered,
unfortunately files in DSEFS/HDFS are not. So lets register our CSV
file as a [table](https://docs.databricks.com/spark/latest/spark-sql/language-manual/create-table.html).

     CREATE TABLE important_csv USING csv OPTIONS (path='/data/important.csv');
     select * from important_csv;
     
    +------------+--------+------+---------+--+
    |    _c0     |  _c1   | _c2  |   _c3   |
    +------------+--------+------+---------+--+
    | Name       | Type   | Age  | Color   |
    | Sundance   | dog    | 10   | brown   |
    | Cara       | dog    | 10   | white   |
    | Mr. Pants  | cat    | 12   | brown   |
    | HD         | dog    | 2    | brown   |
    | Benny      | dog    | 6    | white   |
    | Tony       | tiger  | 45   | orange  |
    +------------+--------+------+---------+--+0
       
     describe important_csv;
     +-----------+------------+----------+--+
     | col_name  | data_type  | comment  |
     +-----------+------------+----------+--+
     | _c0       | string     | NULL     |
     | _c1       | string     | NULL     |
     | _c2       | string     | NULL     |
     | _c3       | string     | NULL     |
     +-----------+------------+----------+--+
     
When we ran that describe it looks like all of our data types were marked
as strings ... Isn't Spark a super smart system that could tell the types
of my data? It can! But you have to tell it explicitly. What we actually
did in the above code is tell Spark that we have a [DataSource](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/DataSource.scala)
of type "csv". This could also be any fully defined class name which implements
the right interfaces. Each *DataSource* Is also allowed to have it's own 
set of options and modifiers which change it's behavior.

### Challenge: Getting Spark to Learn Data Types in CSV Files

Get Spark to read in this table with appropriate data types by telling the
CSV Datasource to *infer* something about the *Schema*. You may want to check out
the documentation [here](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrameReader@csv(paths:String*):org.apache.spark.sql.DataFrame)
Don't forget the *headers*!

You may have to *drop* the previous table we made before you remake your table
with the correct schema.
   
### Challenge: Write a Parquet files our CSV Data

Once a Source has been registered we can read or write to it. To change our
CSV data to a Parquet file we need to use some [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML) 
syntax for copying.

Here is an example of us writing to DSE using an insert statement in 
JDBC

    INSERT INTO test.tab SELECT 10 as k, 10 as c, 10 as v;
    #Note this is an incredibly expensive way of inserting 1 row :)
    #Use the "explain" command to see the amount of work which will go into this insert
    
If we similarly define a Parquet source we can insert values into it
by selecting rows from another source. 

So try the following

* Create a parquet source located in our parquet directory we made above using the CSV data
    * hint: C.T.A.S
* Use the DSE FS shell to look at the results
* Check the UI's we went over to see the actual Spark Work that was done.
* Check out the SQL Tab as well (for fun)

### Challenge: Do some fun non-cql calls on our DSE and Parquet Data

* Join test.tab and your parquet table on k = age

        +-----+-----+-----+-----------+-------+------+--------+--+
        |  k  |  c  |  v  |   Name    | Type  | Age  | Color  |
        +-----+-----+-----+-----------+-------+------+--------+--+
        | 10  | 10  | 10  | Cara      | dog   | 10   | white  |
        | 10  | 10  | 10  | Sundance  | dog   | 10   | brown  |
        +-----+-----+-----+-----------+-------+------+--------+--+
 
 Congratulations you just joined a Distributed Database with a Columnar
 Distributed File Format.
 
 * Filter your C* table on a non-partition key column
 * Count stuff
 
 ## Conclusion
 
 This is the bare minimum required to get into working with Spark, 
 to learn more about what's happening under the hood continue on to
 the Zeppelin section.
