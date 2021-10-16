# Implementing Client Server Architecture using MySQL

Before we start we need to have an environemnt to work with. I will we using my AWS account to create **two** EC2 instances with an Ubuntu OS.
Make sure both instances are in the same **subnet**

* First thing I'm going to do when I log in to AWS is look for the **EC2** services. There are various methods to navigate to it, here I'm using the **search bar** <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/ec2search.png) <br>

* Once you navigate to the **EC2** page look for a **Launch instance** button <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/launchInstance.png) <br>

* You will then be prompted to pick an OS Image. I will be using **Ubuntu Server 20.04 LTS (HVM), SSD Volume**. Once done click **Select**

* I will pick the **t2.micro** instace type <br /> 
 ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/t2micro.png) <br>

* I will leave default settings and click **Review and Launch** <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/reviewLaunch.png) <br>

* As you can see we have a Security Group applied to the instance by default which allows SSH connections <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/sshDefault.png) <br>

* After reviewing you can launch your instancing by clicking <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/launch.png) <br>

* We are prompted to create or use an existing Key Pair. I will be creating a new one. I will use this .pem key to SSH into the instance later on. <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/keyPair.png) <br>

* Once you have downloaded your key launch intance <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/LaunchInstances.png) <br>

* To go to the instances dashboard <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/ViewInstances.png) <br>

* If your instance is up and running you will see something like this <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/Running.png) <br>

* To find information on how to connect click on your **Instance ID**

* In the top-right corner you should see the button **Connect**, click on it <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/Connect.png) <br>

* Look for the **SSH client** tab <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/SSHclient.png) <br>

* Under **Example** you'll find an **ssh** command with eveything you need to connect to the instance from a terminal

**Examaple:**
```bash
ssh -i "daro.io.pem" ubuntu@ec2-3-216-90-84.compute-1.amazonaws.com
```
Make sure when you run the command that your current working directory in the terminal is where your KeyPair/.pem is located because in the above example I'm using a relative path to point to my key

* A successful log-in <br /> 
![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/ubunutuLogIn.png) <br>


# INSTALLING MYSQL

First, we run update on both instances
```bash
sudo apt update
```
Then we choose which instnace is the **client** and which one is the **server**

On the client side we run
```perl
      #Client IP
ubuntu@ip-172-31-61-156:~$ sudo apt install mysql-client -y
```

On the server side we run
```perl
      #Server IP
ubuntu@ip-172-31-52-209:~$ sudo apt install mysql-server -y
```

# PREP SERVER SIDE

 First, we need to configure MySQL to allow connections from remote hosts. Edit file **mysql.cnf** and make sure **bind-address** equals **0.0.0.0**
 ```perl
 #Using grep to filter output of cat
 ubuntu@ip-172-31-52-209:~$ sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
ubuntu@ip-172-31-52-209:~$ sudo cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep bind
bind-address            = 0.0.0.0
mysqlx-bind-address     = 127.0.0.1
ubuntu@ip-172-31-52-209:~$
```

Second, lets create a user in MySQL to connect to

```sql
ubuntu@server-172-31-52-209:~$ sudo mysql
mysql>
mysql> CREATE USER 'hector'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
Query OK, 0 rows affected (0.02 sec)
mysql>
```

* Now we are going to add a rule to our **Security Group** to open **TCP** port **3306**
    * Navigate to your intances dashboard and select your instance by cliking the empty box <br /> 
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/checkMark.png) <br>
    * Look for the **Security** tab <br /> 
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/security.png) <br>
    * Click on top of your **Security Group**  <br />
    Keep in mind yours might look different <br/>
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/securityGroup.png) <br> <br />
    
    * Under the tab **Inbound Rules** click **Edit inbound rules** button <br />
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/editInboundRules.png) <br> <br />
    * Click **Add rule** <br />
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/addRule.png) <br> 
    * Pick type **Custom TCP** and source **0.0.0.0/0** meaning all IPs, you can also use drop-down menu **Anywhere-IPv4**
    * Save it <br />
    ![Markdown Logo](https://raw.githubusercontent.com/hectorproko/LAMP_STACK/main/images/SaveRules.png) <br> 


    # CLIENT SIDE

    Now we'll attempt to make a connection to the MySQL server using user **hector**

    ```perl
    ubuntu@ip-172-31-61-156:~$ sudo mysql -u hector -p -h  172.31.52.209
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 15
    Server version: 8.0.26-0ubuntu0.20.04.2 (Ubuntu)

    Copyright (c) 2000, 2021, Oracle and/or its affiliates.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> SHOW DATABASES;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    +--------------------+
    1 row in set (0.00 sec)

    mysql>
    ```
We can display remote databases hence a successful connection