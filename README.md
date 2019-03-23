# DB8-ReplicationAndTransactions
Database Assignment 8 regarding Mysql, Replication and Transactions

Assignment: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment8.md

Slides: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/lecture_notes/08-TransactionsAndReplication.ipynb

------

## VERY IMPORTANT

- This guide for setting up a new Slave, has been created using a **Ubuntu Droplet 18.04**. If you are using an older Ubuntu-version, there might be some discrepancies!

- You will need login information to connect your slave-droplet with the master droplet. Please see the Peergrade-handin for a file containing the information. The guide will tell you need to use the information.

- In the same file as mentioned above, you will also find the login information for the Master droplet, in which you need to execute the various queries to see the effect in your droplet

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

Flush the priviligies to confirm the new changes:

```mysql
FLUSH PRIVILEGES;
```

----------------

### Setup Database

Having now setup the basic things in your droplets, we now need to download and setup the Database.

#### CreateTables.sql

In your droplet, download the 'CreateTables'-file file:

`wget https://raw.githubusercontent.com/radeonxray/DB-Assignment6/master/CreateTables.sql`

#### Classicmodels.sql

There are 2 options to download the Database to your Droplet (you only need to chose and do one of them!):

##### 1: Scp
Copy the `classicmodels.sql`-file found in this github repo to your droplet.
Download to your desktop and use the following command to copy to your droplet, remember to replace the placeholder information with the correct one:
```shell
scp classicmodels.sql [username]@[dropletIP]:classicmodels.sql 
```

##### 2: Wget

Download the `classicmodels.sql`-file directly from the repo and onto your droplet, using the following command:

`wget https://github.com/radeonxray/DB8-ReplicationAndTransactions/blob/master/classicmodels.sql`


#### Create the DB

With the `classicmodels.sql`- and `CreateTables.sql`-file on your droplet, login to your MySQL and execute the following command to create the DB in your MySQL setup:

```mysql
source ./CreateTables.sql;
```

Select the Schema to be used

```mysql
use classicmodels
```
-------

### Edit the mysqld.cnf

Now that we have setup the Database and a MySQL user, we need to configure MySQL a bit, in order to set it up as a slave.

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
*Note: There might be mutiple tester of this code and since the **server-id** has to be unique, you might want to use a random number between 3-999 if you get an error*

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

Run the following command in MySQL to setup the slave droplet with the slave-user:

***Important**: Check the Peergrade-handin for the file containing the correct script with the correct information! The command below is just a placeholder to show you what the command actually looks like*

```mysql
CHANGE MASTER TO MASTER_HOST='[CHECKPEERGRADEFORIP]',MASTER_USER='[CHECKPEERGRADEFORUSERNAME]', MASTER_PASSWORD='[CHECKPEERGRADEFORPASSWORD]', MASTER_LOG_FILE='[CHECKPEERGRADEFORMASTERLOGFILE]', MASTER_LOG_POS= [CHECKPEERGRADEFORPOS#];

```

Having executed the correct query, run the following mysql command to start the slave:

`start slave;`

Then check the status of the slave with the following mysql command:

`SHOW SLAVE STATUS\G`

You really want to see ` Slave_IO_State` stating *Waiting for Master to send event*, and both `Slave_IO_Running` and ` Slave_SQL_Running` stating ´Yes´.

If all the above has been successfully executed, you are now ready to run queries to the Master DB and see the effects in your slave Database!
 
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

