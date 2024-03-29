# DB8-ReplicationAndTransactions
Database Assignment 8 regarding Mysql, Replication and Transactions

Assignment: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment8.md

Slides: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/lecture_notes/08-TransactionsAndReplication.ipynb

------

## Hand-in
You need to document enough of your server for your reviewer to be able to set-up a new slave of your server. You need to make a database user which allow your reviewer to be able to make an update to your master database to verify that the slave updates correctly.

- Put password in a special file for peergrade, so it is not on github.

-----

## Review

- Verify that you are able to set up your own slave of the database you review.
- Verify that you can update the master database and see the changes to your database.

-----

## VERY IMPORTANT

- This guide for setting up a new Slave, has been created using a **Ubuntu Droplet 18.04**. If you are using an older Ubuntu-version, there might be some discrepancies!

- You will need login information to connect your slave-droplet with the master droplet. Please see the Peergrade-handin for a file containing the information. The guide will tell you when to use the information from the file.

- In the same file as mentioned above, you will also find the login information for the Master droplet, in which you need to execute the various queries to see the effect in your droplet (Please be nice to it, don't mess around).

- Login information has also been provided to my own Slave-droplet, please also be nice to it, should you take a look.

- The guide was created with the goal of making the user able to connect and perform action on the database using [MySQL Workbench](https://dev.mysql.com/downloads/workbench/), but you are also able to use the Terminal or Bash directly if you prefer.

------
## Tools and other information

Some basic information about the creation of this guide:
- **Master Server**: Singapore
- **Slave Server**: Frankfurt
- Tested on both Windows and Mac
- Connected to both the Master and Slave through [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) to perform queries.
- Users for the MySQL part of the Master and Slave was created directly through the droplet and a Terminal/Bash

-----

## Setup

The following is a guide on how to setup your own fresh droplet as a Slave to my Master Database. You might want to have a notepad open to note information such as usernames and passwords for later.

### Update and upgrade Droplet

Login to your Droplet and run the following commands: 

```shell
sudo apt-get update
```
```shell
sudo apt-get upgrade
```

### Install MySQL
```shell
sudo apt install mysql-server
```

### Setup MySQL
Setup basic Mysql informaiton using the following command:
```shell
sudo mysql_secure_installation
```
After the installation is complete, login to MySQL `mysql -u [username] -p` (You should be prompted for at password) and create a new user (replace the username and password with something of your own choice):

```mysql
CREATE USER '[USERNAME]'@'%' IDENTIFIED BY '[PASSWORD]';
```

Grant priviliges to the new user:

`GRANT ALL PRIVILEGES ON *.* TO '[UserName]'@'%' WITH GRANT OPTION;`

**Suggestions from reviwer:**

Creat a special *replicate slave*-user instead 
`GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'%' IDENTIFIED BY 'password';`

Flush the priviligies to confirm the new changes:

```mysql
FLUSH PRIVILEGES;
```

Check that the user was created (this does not check that the privilges was flushed!)

```mysql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

The reason we are creating a new user and not just stick with the 'root' user provided with mysql, is because we want to be able to remotely connect to the slave (hence why the '[username]'@'%' and not '[username]'@'localhost') through a program like [MySQL Workbench](https://dev.mysql.com/downloads/workbench/), and you don't want to expose the 'root' user to remote access in that way.

Furthermore, one could argue about giving the mysql user privileges to basically "everything" in the slave, but that discussion is for another day (:  

Now that MySQL has been installed and very roughly setup, exit MySQL and lets get on setting up the database in MySQL.

```mysql
exit
```

----------------

### Setup Database

Having now setup the basic things in your droplets, we now need to download and setup the actual Database in Mysql.

#### CreateTables.sql

In your droplet (preferable the place you are placed right when you login and  connect to your droplet), download the 'CreateTables'-file file:

`wget https://raw.githubusercontent.com/radeonxray/DB-Assignment6/master/CreateTables.sql`

#### Classicmodels.sql

There are 3 options to download the Database to your Droplet (you only need to chose and do one of them!):

##### 1: Scp
Copy the `classicmodels.sql`-file found in this github repo to your droplet.
Download to your desktop and use the following command to copy to your droplet, remember to replace the placeholder information with the correct one:
```shell
scp classicmodels.sql [username]@[dropletIP]:classicmodels.sql 
```

##### 2: Wget (the easiest)

Download the `classicmodels.sql`-file directly from the repo and onto your droplet, using the following command:

`wget https://raw.githubusercontent.com/radeonxray/DB8-ReplicationAndTransactions/master/classicmodels.sql`

#### 3: Copy directly from droplet to droplet

`ssh -A -t root@<old server ip> scp /root/classicmodels.sql root@<new server ip>:/root/classicmodels.sql`

#### Create the DB

With the `classicmodels.sql`- and `CreateTables.sql`-file on your droplet, login to your MySQL and execute the following command to create the DB in your MySQL setup:

```mysql
source ./CreateTables.sql;
```

Select the Schema to be used

```mysql
use classicmodels
```

You should now have the `classicmodels` database setup in MySQL. Next, we'll look at making the final configuration to your MySQL setup, in order for your droplet to act and perform as an actual Slave to my Master Droplet and Database, so exit MySQL.

```mysql
exit
```

-------

### Edit the mysqld.cnf

Now that we have setup the Database in MySQL and a created a MySQL user within the slave-droplet, we need to configure MySQL a bit more, in order to set it up as a slave correctly.

In Ubuntu 18.04, you will locate your config-file for MySQL a bit different than earlier versions. 
Open your droplet and enter the following command:
```shell
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

You should now be placed inside the file and be able to edit the files using Nano.

In your mysql.conf located at ```/etc/mysql/mysql.conf.d/mysqld.cnf``` change the following:

*Remember to replace [INSERT YOUR DROPLETS IP ADDRESS] with your droplets own ip.* 
```shell
bind-address	=[INSERT YOUR DROPLETS IP ADDRESS]
server-id     = 3
log_bin       = /var/log/mysql/mysql-bin.log
```
*Note: There might be mutiple testers/reviewers of this project and since the **server-id** has to be unique, you might want to use a random number between 3-999 if you get an error*

Locate the following line:
```shell
binlog_do_db  = classicmodels
```
Change it to:
```shell
replicate_do_db 	= classicmodels
```

Add the following line to the file (I personally placed it in the "Logging and Replication" section, but it makes no difference ):
```shell
relay-log  = /var/log/mysql/mysql-relay-bin.log
```


After you have done all of the above changes to the ```mysqld.cnf```-file, save and close the file, then restart mysql and its services with the following command:

```shell
sudo service mysql restart
```

### Start Slave in MySQL

After the restart is complete, run the following command in MySQL to setup the slave droplet with the slave-user:

***Important**: Check the Peergrade-handin for the file containing the correct script with the correct information! The command below is just a placeholder to show you what the command actually looks like*

```mysql
CHANGE MASTER TO MASTER_HOST='[CHECKPEERGRADEFORIP]',MASTER_USER='[CHECKPEERGRADEFORUSERNAME]', MASTER_PASSWORD='[CHECKPEERGRADEFORPASSWORD]', MASTER_LOG_FILE='[CHECKPEERGRADEFORMASTERLOGFILE]', MASTER_LOG_POS= [CHECKPEERGRADEFORPOS#];

```

Having executed the correct query with the right information/data, run the following mysql command to start the slave:

`start slave;`

Then check the status of the slave with the following mysql command:

`SHOW SLAVE STATUS\G`

You really want to check that ` Slave_IO_State` states *Waiting for Master to send event*, and both `Slave_IO_Running` and ` Slave_SQL_Running` states ´Yes´.

If there is an issue in connecting, you can try starting slave with a command to skip over it:

```mysql
SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
```

Followed by:

`start slave;`

If all the above has been successfully executed, you are now ready to run queries to the Master DB and see the effects in your slave Database! You can run the queries directly your Terminal (Mac) or Bash (Windows), or connect to your slave using [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)
 
-------

### Test Queries

#### Assignment 4

*Make an insert in one of the tables in singapore, and see how long it takes for the tables in Europe to update.*

Tested the speed of communication between the Master and Slave using the following query:

```mysql
INSERT INTO customers (customerNumber, customerName, contactLastName, contactFirstName, phone, addressLine1, city, state, postalCode, country, creditLimit) VALUES (497 'Ceo', 'Olsen', 'Christian', '31431716', 'Peter Fabers Gade 26 th', 'Denmark', 'Copenhagen', '2200', 'Denmark', '100000.00'); 
```
After having worked out the various setup kinks, the current speed seems like a fraction of a seconds, so no real impact (as of now)

-----

#### Assignment 5

*Make a transaction of several updates on the Singapore database, and verify that no changes happens to the European database until after the commit of the transaction.*

Using the following transaction on the Master droplet, will not affect the Slave droplet, since the transaction is not commited:

```mysql
START TRANSACTION;
	UPDATE customers SET customerName = "RadeonXRay" WHERE customerNumber=497;
```

However, the following transaction below will be afecting both the Master droplet and the Slave droplet:

```mysql
START TRANSACTION;
	UPDATE customers SET customerName = "RadeonXRay" WHERE customerNumber=497;
COMMIT;
```

------

# Other information - NOT IMPORTANT! Read only if you want bonus info!

Following information is just to inform you (and better help me remember) how to recreate other aspects of this assignment. These steps are not required in order to create your slave, since the information is already provided in the guide

### See the Master Status

Run the following command in the Master-droplet to create a Replication user, that will be used for connecting the slave droplet with the master droplet:

`GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'%' IDENTIFIED BY 'password';`


If you want to create the slave user from the ground up, connect to my Master Droplet (info can be found in the peergrade-file), login to the MySQL-part of the droplet and enter the following command:

`SHOW MASTER STATUS;`

This will show you:

`File             | Position | Binlog_Do_DB | Binlog_Ignore_DB`

Take that structure into consideration when looking at the statement run in the slave:

`CHANGE MASTER TO MASTER_HOST='[CHECKPEERGRADEFORIP]',MASTER_USER='[CHECKPEERGRADEFORUSERNAME]', MASTER_PASSWORD='[CHECKPEERGRADEFORPASSWORD]', MASTER_LOG_FILE='[CHECKPEERGRADEFORMASTERLOGFILE]', MASTER_LOG_POS= [CHECKPEERGRADEFORPOS#];`

Breaking the info down:

`CHANGE MASTER TO MASTER_HOST` is the IP to the Master/Host Droplet

`MASTER_USER` is actually the Slave User in our example

`MASTER_PASSWORD` Password for the Slave User

`Master_Log_File` is the whole name found under `File` in th `SHOW MASTER STATUS`

`Master_Log_POS` is the number under `Position` in th `SHOW MASTER STATUS`

----------

#### Guides used for this assignment

https://www.digitalocean.com/community/tutorials/how-to-set-up-master-slave-replication-in-mysql 
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04
https://dba.stackexchange.com/questions/21119/how-do-i-completely-disable-mysql-replication
https://dba.stackexchange.com/questions/151680/master-slave-mysql-error-in-replication
http://lasanthals.blogspot.com/2012/09/mysql-replication-server-configuration.html
https://dev.mysql.com/doc/refman/8.0/en/commit.html 
