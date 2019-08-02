# Isilon Multi Protocol Batch Analytics Demo
This is a demonstration to show that data can be ingested into Isilon via different protocols such as SFTP, NFS and SMB, after that we could use Spark to analyze the data from a Hadoop cluster connected to Isilon via HDFS.

![Diagram](/Isilon-Multi-Protocol-Batch-Analytics.png)
[![Analytics](https://ga-beacon.appspot.com/UA-145007542-1/welcome-page)](https://github.com/igrigorik/ga-beacon)

### Setup Environment:
#### 1. Setup Hortonworks and Isilon with this [guide](https://www.emc.com/collateral/TechnicalDocument/docu71396.pdf).
During the Hortonworks HDP installation, include Hive, Spark, NiFi and Zeppelin.
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

	beeline -u jdbc:hive2://<hadoopnode>:10000/appview  -n hduser2
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
#### 5. Create a NiFi Flow with this template (HDP-NiFi):
	HDP-500_Ingest_data_into_Isilon_via_SFTP_and_NFS.xml
#### 6. Configure FTP access in Isilon.
#### 7. Configure NFS export in Isilon.
Example NFS path -> /ifs/<accesszone>/hadoop/user/hduser2/sales_data/transactiontable
	SSH to Hadoop node to mount the NFS.
	mount -t nfs <smartconnectaccesszone>:/transactiontable /mnt/hdp-nfs
#### 8. Create SMB share in Isilon.
Example SMB path -> /ifs/<accesszone>/hadoop/user/hduser2/sales_data/transactiontable
	
#### 9. Install NiFi in a Windows server (Win-NiFi).
	Start NiFi -> C:\nifi-xxx\bin\run-nifi.bat
Map Network drive to Z: with the SMB share from Isilon.

Create a NiFi Flow with this template:
	HDP-H500_ingest_data_into_Isilon_via_SMB.xml
	
Execute this batch file: moveFileToShareFolder.bat
#### 10. Configure Zeppelin to analyze the data.
Open Zeppelin WebUI: http://hadoopnode:9995

Import note: Isilon Data Lake Demo.json

### Demo:
#### 1. Start HDP-NiFi Flow and Win-NiFi Flow:
	http://<hadoopnode>:8080/nifi
	http://<windowserver>:8080/nifi
#### 2. Open Zeppelin and run the note: Isilon Data Lake Demo
