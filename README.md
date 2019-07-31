# Isilon Multi Protocol Batch Analytics Demo
This is a demonstration to show that data can be ingested into Isilon via different protocols such as SFTP, NFS and SMB, after that we could use Spark to analyze the data from a Hadoop cluster connected to Isilon via HDFS.

![Diagram](/Isilon-Multi-Protocol-Batch-Analytics.png)

### Setup:
#### 1. Setup Hortonworks and Isilon with this [guide](https://www.emc.com/collateral/TechnicalDocument/docu71396.pdf).
#### 2. Create a new user in Isilon.
	isi auth groups create hduser2 --zone hdpzonename --provider local
	isi auth users create hduser2 --primary-group hduser2 \
	--zone hdpzonename --provider local \
	--home-directory /ifs/hdpzonename/hdp/hadoop/user/hduser2
#### 3. Create user in Hadoop node.
	useradd -u 2004 hduser2
	sudo -u hdfs hdfs dfs -mkdir -p /user/hduser2
	sudo -u hdfs hdfs dfs -chown hduser2:hduser2 /user/hduser2
	sudo -u hdfs hdfs dfs -chmod 755 /user/hduser2
#### 4. Create a new database in Hive.
	SSH to Hadoop node.
	su - hduser2
	hadoop fs -mkdir -p sales_data/transactiontable

	beeline -u jdbc:hive2://pnode1.pgen6.local:10000/appview  -n hduser2
	create database appdb;
	use appdb;
	
	
	DROP TABLE IF EXISTS transactiontable;
	CREATE EXTERNAL TABLE transactiontable
	(
	   transdate Timestamp,
	   ipaddr STRING,
	   platform INT,
	   itemid INT,
	   quantity INT
	)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
	LOCATION '/user/hduser2/sales_data/transactiontable';
	
