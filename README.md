# DB8-ReplicationAndTransactions
Database Assignment 8 regarding Mysql, Replication and Transactions

Assignment: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment8.md

Slides: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/lecture_notes/08-TransactionsAndReplication.ipynb

------

## VERY IMPORTANT

- This guide for setting up a new Slave, has been created using a **Ubuntu Droplet 18.04**. If you are using an older Ubuntu-version, there might be some discrepancies!

- You will need login information to connect your slave-droplet with the master droplet. Please see the Peergrade-handin for a file containing the information. The guide will tell you need to use the information.

-----

## Hand-in
You need to document enough of your server for your reviewer to be able to set-up a new slave of your server. You need to make a database user which allow your reviewer to be able to make an update to your master database to verify that the slave updates correctly.

- Put password in a special file for peergrade, so it is not on github.

-----

## Setup

### Update and upgrade Droplet
```shell
apt-get update
```
```shell
apt-get upgrade
```

### Install MySQL
```shell
sudo apt install mysql-server
```

Setup basic Mysql informaiton using the following command:
```shell
sudo mysql_secure_installation
```

### Setup Database

Copy the `classicmodels.sql`-file found in this github repo to your droplet.
Download to your desktop and use the following command to copy to your droplet, remember to replace the placeholder information with the correct one:
```shell
scp classicmodels.sql [username]@[dropletIP]:classicmodels.sql 
```

### Access Mysql

Use the following command to access your mysql:

```shell
mysql -u [username] -p 
```
You should be prompted for at password. 



source ./CreateTables.sql;

### Droplet



In Ubuntu 18.04, you will locate your config-file for MySQL a bit different than earlier versions. 
Open your droplet and enter the following command:
```shell
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

You should now be placed inside the file and be able to edit the files using Nano.

In your mysql.conf located at ```/etc/mysql/mysql.conf.d/mysqld.cnf``` change the following:

Remember to replace [INSERT YOUR DROPLETS IP ADDRESS] with your droplets own ip. 
```shell
bind-address	=[INSERT YOUR DROPLETS IP ADDRESS]
server-id     = 3
log_bin       = /var/log/mysql/mysql-bin.log
```

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


After you have done all of the above changes to the ```mysqld.cnf```-file, restart mysql and its services with the following command:

```shell
sudo service mysql restart
```


### MySQL

Create a new user and replace the username and password with something of your own choice:

```mysql
CREATE USER '[USERNAME]'@'%' IDENTIFIED BY '[PASSWORD]';
```

Run the following command to setup the slave droplet with the slave-user:

***Important**: Check the Peergrade-handin for the file containing the correct script with the correct information! The command below is just a placeholder to show you what the command actually looks like*

```mysql
CHANGE MASTER TO MASTER_HOST='[CHECKPEERGRADEFORIP]',MASTER_USER='[CHECKPEERGRADEFORUSERNAME]', MASTER_PASSWORD='[CHECKPEERGRADEFORPASSWORD]', MASTER_LOG_FILE='[CHECKPEERGRADEFORMASTERLOGFILE]', MASTER_LOG_POS= [CHECKPEERGRADEFORPOS#];

```


### Slave

### Test Queries

#### Assignment 4

```mysql
INSERT INTO customers (customerNumber, customerName, contactLastName, contactFirstName, phone, addressLine1, city, state, postalCode, country, creditLimit) VALUES (497 'Ceo', 'Olsen', 'Christian', '31431716', 'Peter Fabers Gade 26 th', 'Denmark', 'Copenhagen', '2200', 'Denmark', '100000.00'); 
```

#### Assignment 5

-----

## Review

- Verify that you are able to set up your own slave of the database you review.
- Verify that you can update the master database and see the changes to your database.

