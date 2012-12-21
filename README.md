NoMapReduce Data Queries (SELECT & JOIN) in HBase.
======
Effective Data Queries (SELECT &amp; JOIN) in a HBase store without the use of MapReduce.

Feature List (23/10/12)
---------------

 * A Hadoop Job that imports document data from files on HDFS to a HBase table, using bulk loading (HFileOutputFormat).
 * A Hadoop Job that imports TPC-H benchmark data from files on HDFS to a HBase table, using bulk loading (HFileOutputFormat).
 * A Hadoop Job that imports synthetic benchmark data from files on HDFS to a HBase table, using bulk loading (HFileOutputFormat).
 * A Hadoop Job that creates an Index HBase table for a given Data HBase table, using bulk loading (HFileOutputFormat).
 * SELECT query using HBase's Filter API.
 * SELECT query using the produced Index HBase table.
 * EQUI-JOIN query using single-table Index HBase tables.
 * EQUI-JOIN query using multi-table Index HBase tables. 
  

Requirements
---------------
 * Java Runtime Environment (1.6).
 * Java Development Kit (1.6).
 * Maven (3.0.3).
 * Apache Hadoop (1.0.3).
 * Apache HBase (0.92.1).

Modules (23/10/12)
---------------
This project is organized in several different modules. So far only 2 have been implemented. They are:

 * import-document-data-to-hbase : A self-contained Hadoop Job capable of importing data from file on HDFS to a HBase table. 
Specifically it imports terms and stores their term and document frequency in a HBase Table. It requires the user 
to provide several info such us where to find it's input on HDFS, a working HBase table and fully qualified columns. It uses the Bulk Loading process 
(http://hbase.apache.org/book.html#arch.bulk.load) to fast load produced HFiles in order to make the appropriate HBase data Table.
 * import-tpch-data-to-hbase : unimplemented!
 * import-synthetic-data-to-hbase : unimplemented!
 * create-index-for-hbase : A self-contained Hadoop Job capable of creating an in-store index table based on the data found in a pre-existed HBase table.
It requires the user to provide the source table along with the fully qualified column (ColumnFamily:ColumnQualifier) aimed for indexing.
This index is an HBase table is being maintained (create/update/truncated) via the use of this module. It uses the Bulk Loading process to fast load produced HFiles in order to make the appropriate HBase data Table.
 * select-query-using-api : A Java Application using standard HBase 0.92 API calls, including Filters, to implement a SELECT-like query over an existing HBase table. 
 * select-query-using-index : A Java Application using standard HBase 0.92 API calls, to implement a SELECT-like query over an existing HBase table and it's respected Index HBase table. 
 * equi-join-query-using-index : A Java Application using standard HBase 0.92 API calls and the CoProcessors extentions, to implement a EQUI-JOIN like query over two existing HBase tables and their Index tables.
 * equi-join-query-using-multitable-index : A Java Application using standard HBase 0.92 API calls, to implement a EQUI-JOIN like query over two existing HBase tables and their common index table(multi-table).

Data/Index Model
---------------
HBase is a powerful distributed key-value store heavily using Hadoop's Apache HDFS and Apache Zookeeper.
Data in form of rows,columns and tables is distributed in files on HDFS shared by the supporting node cluster. 
A generic row looks like this `<row_key : [column_family:column_qualifier = value,...,column_familyM:column_qualifierN = value]>`.
This project aim is to use efficiently key-value store's capabilities and provide an in-store index (index HBase table) 
in order to execute SQL-like queries, such as SELECT and JOIN, without using MapReduce techniques which show increased 
system overhead for a relatively small number of data. The index rows must adhere to the following format,
 `<value : [column_family_column_qualifier:row_key = NULL>`.
	
So far our experimenting data source is a collection of documents. The HBase store holds two tables :

`document_table`

	deck 		column=documents_family:documents, timestamp=1342439733261, value=[1.txt, 3.txt, 4.txt]
	deck        column=documents_stats_family:count, timestamp=1342439733261, value=\x00\x00\x00\x03
	deck        column=documents_stats_family:tf, timestamp=1342439733261, value={3.txt=7, 4.txt=1, 1.txt=2}

`index_document_table`

	\x00\x00\x00\x03		column=documents_stats_family_count:decent, timestamp=1342441001480, value=
	\x00\x00\x00\x03        column=documents_stats_family_count:decisive, timestamp=1342441001480, value=
	\x00\x00\x00\x03        column=documents_stats_family_count:deck, timestamp=1342441001480, value=
	\x00\x00\x00\x03        column=documents_stats_family_count:declaration, timestamp=1342441001480, value=
	...
	[1.txt, 3.txt, 4.txt] 	documents_family_documents:deck, timestamp=1342440701717, value=
	[1.txt, 3.txt, 4.txt] 	documents_family_documents:defects, timestamp=1342440701717, value=
	[1.txt, 3.txt, 4.txt] 	documents_family_documents:depths, timestamp=1342440701717, value=

How to Compile
---------------
To compile :

	cd nomapreduce-queries-hbase
	mvn clean package

Inside the modules now lies a target directory containing the necessary jobs jars. Use those who end in `*-job.jar`.
Run the .jars
---------------
* Before running you should export the system path of the Apache Hadoop & HBase distributions to your system variables. `$ export $HADOOP_HOME=/path/to/hadoop` , `$ export $HBASE_HOME=/path/to/habase` 
* Start all Apache Hadoop Services (NameNode,JobTracker,DataNodes,etc) in all your cluster nodes appropriately.
* Start all Apache HBase Services (HBaseMaster, RegionServers) in all your cluster nodes appropriately.
* Make sure you have your input data in place on the HDFS.
* Consult the --help argument explaining the command line arguments for the complied job jars.
* Consult the shell scripts found in nomapreduce-queries-hbase/scripts directory. Their names are quite self-descriptive of what they do.
