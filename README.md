# Configure Linux Web Server
This project is about how to set up a Ubuntu Linux server to host an Item Catalog application built with Python, Flask and SQLAlchemy.

## 1. Amazon Lightsail VM instance details
The public IP address used is `100.24.2.187` and SSH port used is `2200`.

The URL to the hosted web page is:
http://100.24.2.187/ or http://ec2-100-24-2-187.compute-1.amazonaws.com/

## 2. Software to install during the configuration
- Apache2
- Flask
- Flask-SQLAlchemy
- git
- httplib2
- libpq-dev
- mod_wsgi
- oauth2client
- pip
- PostgreSQL
- Psycopg2
- Python Requests
- virtualenv


## 3. Configuration steps
### I) Create an instance with Amazon Lightsail
1. Sign in to **[Amazon Lightsail](https://amazonlightsail.com)** using an Amazon Web Services account

2. Follow the 'Create an instance' link.

3. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options.

4. Choose $3.50/month payment plan. Deactivate when this web app no longer is needed.

5. Give the instance a unique name and click 'Create'.

6. Wait for the instance to start up.


### II) Connect to the instance on a local machine
1. Download **[PuTTY](http://www.putty.org)** to your local machine.

2. Download your default private key from the Amazon Lightsail Account page.

3. Save LightsailDefaultPrivateKey-us-east-1.pem to local machine.

4. Download PuttyGen to your local machine.

5. Generate amazonSSHKey.ppk from LightsailDefaultPrivateKey-us-east-1.pem

6. From Putty, enter Host Name(or IP Address) as `100.24.2.187` and port `22`. Expand SSH and select Auth and for Private Key file authentication, Browse to amazonSSHKey.ppk and Click Open and again click on Open. For login, enter user name as ubuntu.


### III) Upgrade currently installed packages
1. `sudo apt-get update`

2. `sudo apt-get upgrade`


### IV) Configure the firewall
1. First change the SSH port from `22` to `2200` (sudo vim /etc/ssh/sshd_config), and then restart SSH by running `sudo service ssh restart`)

2. Check to see if pre-installed ubuntu firewall is active by running `sudo ufw status`

3. Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in

4. Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing

5. Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH

6. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200`, so that SSH will work

7. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

8. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP

9. Run `sudo ufw deny 22` to deny port `22`

10. Run `sudo ufw enable` to enable the ufw firewall

11. Run `sudo ufw status` to check which ports are open

12. Update the Amazon Lightsail firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP), `123`(UDP), and `2200`(TCP) should be allowed; make sure to deny the default port `22`)

13. Now, to login from your local machine, open up the Putty Terminal. Change the port from 22 to 2200 and login as ubuntu.


### V) Create a new user named `grader`
1. Run `sudo adduser grader`

2. Enter in a new UNIX password (twice) when prompted

3. Fill out information for the new `grader` user

4. To switch to the `grader` user, run `su - grader`, and enter the password


### VI) Give `grader` user sudo permissions
1. Run `sudo visudo`

2. Search for a line that looks like this:

	`root    ALL=(ALL:ALL) ALL`

3. Add the following line below above line:

	`grader	   ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

5. To verify that `grader` has sudo permissions, run `su - grader`, enter the password, and run `sudo -l`; after entering in the password (again), a line like the following should appear, meaning `grader` has sudo permissions:

		Matching Defaults entries for grader on
	    ip-172-26-6-110.ec2.internal:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

		User grader may run the following commands on
		ip-172-26-6-110.ec2.internal:
	    (ALL : ALL) ALL


### VII) Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

2. Choose a file name as `graderKey` for the key pair.

3. Enter in a passphrase twice. (Two files will be generated; the second one should be graderKey.pub)

4. Log in to the virtual machine

5. Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

6. Run `touch .ssh/authorized_keys`

7. On the local machine, run `cat ~/.ssh/graderKey.pub`

8. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

9. Run `chmod 700 .ssh` on the virtual machine

10. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

11. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and set 'PasswordAuthentication to 'no'; save and exit the file.

12. Run `sudo service ssh restart`

13. From local machine, log in as the grader using the following command:

	`ssh -i ~/.ssh/graderKey -p 2200 grader@100.24.2.187`


### VIII) Configure the local timezone to UTC
1. Run `sudo dpkg-reconfigure tzdata`, and follow the instructions.

2. To verify the timezone is configured correctly, run `date` command.


### IX) Install and configure Apache
1. Run `sudo apt-get install apache2` to install Apache

2. To check Amazon Lightsail public port is opened and Apache is installed correctly, copy & paste `http://100.24.2.187:80` on your browser. If a page with the title 'Apache2 Ubuntu Default Page' opened, means Apache is installed successfully.


### X) Install mod_wsgi
1. Install the mod_wsgi package along with python-dev:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

2. To verify mod_wsgi is enabled, run: `sudo a2enmod wsgi`


### XI) Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running `sudo apt-get install postgresql`

2. Open the `sudo vi /etc/postgresql/9.5/main/pg_hba.conf` file and verify the following entries matches:

	local   all             postgres                                peer

	local   all             all                                     peer

	host    all             all             127.0.0.1/32            md5

	host    all             all             ::1/128                 md5


### XII) Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. Something like the following should appear:

	Python 2.7.12 (default, Dec  4 2017, 14:50:18)
	[GCC 5.4.0 20160609] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> 

### XIII) Create a new PostgreSQL user named `catalog` with limited permissions
1. For security reason, it is important to only use the `postgres` user for accessing the PostgreSQL software. Run `sudo su - postgres` 

2. Connect to psql by running `psql`

3. Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

4. Give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

5. Finally, give the `catalog` user a password by running `\password catalog`

6. Verify the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of 
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

7. Exit psql by running `\q`.

8. Switch back to the `ubuntu` user by running `exit`.


### XIV) Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- Run `sudo adduser catalog`
	- Enter in a new UNIX password (twice) when prompted
	- Fill out information for `catalog`

2. Give the `catalog` user sudo permissions:
    
	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below the above line: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file
	- to verify that `catalog` has sudo permissions, run `sudo su - catalog`, and then run `sudo -l`
	- after entering catalog user's password, a line like the following should appear (meaning `catalog` has sudo permissions):

		```
		User catalog may run the following commands on
			ip-172-26-6-110.ec2.internal:
		    (ALL : ALL) ALL
		```

3. Create a database called `catalog` by running `createdb catalog`

4. Run `psql` and then run `\l` to see that the new database has been created

5. To exit from psql, run `\q`.

6. Switch back to the `ubuntu` user by running `exit`.


### XV) Install git and clone the catalog project that was submitted before.
1. Run `sudo apt-get install git`

2. cd /var/www/ and create itemCatalog directory by running `sudo mkdir itemCatalog`

3. cd itemCatalog, and clone the catalog project:

	`sudo git clone https://github.com/usahoo/upendrasahoo.git itemCatalog`

4. cd /var/www

5. Change the ownership of the 'itemCatalog' directory to `ubuntu` by running:

	`sudo chown -R ubuntu:ubuntu itemCatalog/`

6. Change to the /var/www/itemCatalog/itemCatalog directory

7. Rename application.py file to _\_init_\_.py by running `mv application.py __init__.py`

8. In \_\_init__.py, go to line# 433:

	`app.run(host='0.0.0.0', port=8000)`

	Change this line to:

	`app.run()` and save the file.


### XVI) Create/update client_secrets.json
- Create/update a new project on the **[Google API Console](https://console.developers.google.com)**.

- Create/update an OAuth Client ID (under the Credentials tab), and make sure to add http://100.24.2.187 and http://ec2-100-24-2-187.compute-1.amazonaws.com as authorized JavaScript origins

- Add http://ec2-100-24-2-187.compute-1.amazonaws.com/login, http://ec2-100-24-2-187.compute-1.amazonaws.com/gconnect, and http://ec2-100-24-2-187.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs.

	- Create a file called client_secrets.json in the /var/www/itemCatalog/itemCatalog/ directory 

	- Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

	- Add the client ID to line 11 of the /var/www/itemCatalog/itemCatalog/templates/login.html file

	- Add the complete file path for the client_secrets.json file in lines 34 and 79 in the \_\_init__.py file; change it from 'client_secrets.json' to '/var/www/itemCatalog/itemCatalog/client_secrets.json'


### XVII) Set up a virtual environment and install dependencies
1. Start by installing pip:

	`sudo apt-get install python-pip`

2. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

3. Change to the /var/www/itemCatalog/itemCatalog/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running `virtualenv venv`.

4. Activate the new environment, `venv`, by running `. venv/bin/activate`

5. With the virtual environment active, install the following dependencies:

	`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev`

	`pip install psycopg2`

6. In order to make sure everything was installed correctly, run `python __init__.py`; the following should be returned:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

7. Deactivate the virtual environment by running `deactivate`


### XVIII) Set up and enable a virtual host
1. Create `itemCatalog.conf` file in /etc/apache2/sites-available directory.

2. Add the following into the file:	

		<VirtualHost *:80>
			ServerName 100.24.2.187
			ServerAdmin upendra.sahoo68@gmail.com
			WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
			<Directory /var/www/itemCatalog/itemCatalog/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/itemCatalog/itemCatalog/static
			<Directory /var/www/itemCatalog/itemCatalog/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>
	

3. Run `sudo a2ensite itemCatalog` to enable the virtual host

	The following prompt will be returned:

	```
	Enabling site itemCatalog.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

4. Run `sudo service apache2 reload`


### XIV) Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called itemCatalog.wsgi in /var/www/itemCatalog

2. Add the following to the file:
	
		activate_this = '/var/www/itemCatalog/itemCatalog/venv/bin/activate_this.py'

		execfile(activate_this, dict(__file__=activate_this))

		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/itemCatalog/")

		from itemCatalog import app as application
		application.secret_key = '20181105'
	
3. Restart Apache: `sudo service apache2 restart`


### XV) Switch the database in the application from SQLite to PostgreSQL
Replace line 37 in \_\_init__.py, line 59 in database_setup.py, and line 6 in mock_data.py with the following:

	engine = create_engine('postgresql://catalog:catalog@localhost/catalog')


### XVI) Disable the default Apache site
1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

2. Run `sudo service apache2 reload`  


### XVII) Change the ownership of the project directories
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

	sudo chown -R www-data:www-data itemCatalog/

**Note:** Since ownership has changed, if anything needs to be changed, run as  

	sudo -u www-data nano [filename]


### XVIII) Set up the database schema and populate the database
1. cd /var/www/itemCatalog/itemCatalog, activate the virtualenv by running `. venv/bin/activate`

2. Then run `python mock_data.py`

3. Deactivate the virtualenv (run `deactivate`)

4. Restart Apache again: `sudo service apache2 restart`

5. Now open up a browser and check to make sure the app is working by going to http://100.24.2.187 or http://ec2-100.24.2.187.compute-1.amazonaws.com

6. App log can be found under /var/log/apache2/access.log


<br>

## 4. Please do write your feedback to:
**Upendra Sahoo** <us9452@att.com><br>
Currently pursuing a Nanodegree program with Udacity<br>

_Company: AT&T Inc._<br>
200 S. Laurel Ave.<br>
Room A4-4C09<br>
Middletown, NJ 07748<br>
(732) 420-8618 (Office)


