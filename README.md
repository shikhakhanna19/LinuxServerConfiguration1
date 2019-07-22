# Linux Server Configuration

**Server Details**

IP address : 54.169.103.230

SSH port : 2200

EC2 URL : [http](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)[s](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)[://ec2-54-169-103-230.ap-southeast-1.compute.amazonaws.com/](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)

**Note: **
**HTTPS Required for Facebook Login that&#39;s why I am using URL as  &quot;https://" insted of & "http://" for more details see here: [https://developers.facebook.com/blog/post/2018/06/08/enforce-https-facebook-login/](https://developers.facebook.com/blog/post/2018/06/08/enforce-https-facebook-login/)**

IMP NOTE: That&#39;s why I have to change the firewall setting (allow port 443) at aws instence and also at SSH server

**Configuration steps**

**Create an instance in AWS Lightsail**

- Go to AWS Lightsail and create a new account / sign in with your account.
- Click Create instance and choose Linux/Unix,OS only Ubuntu 16.04LTS
- Choose a payment plan (the cheapest plan is enough for now and it&#39;s free for first month)
- Click Create button to create an instance.

**Reference**

- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).

**Set up SSH key**

- Go to account page from your AWS account. You will find your SSH key there.
- Download your SSH key, the file name will be like LightsailDefaultPrivateKey-\*.pem
- Navigate to the directory where your file is stored in your terminal.(/c/Users/Shikha/.ssh)
- Run chmod 600 LightsailDefaultPrivateKey-\*.pem to restrict the file permission.
- Change name to lightsail\_key1.rsa.
- Run a command  ssh -i ~/.ssh/lightsail\_key1.rsa ubuntu@54.169.103.230 in your terminal to cnnect to the instance via the terminal, where 54.169.103.230 is the public IP address of the instance.

**Secure the server**

- **Update and upgrade installed packages**

sudo apt-get update

sudo apt-get upgrade

**Change the SSH port from 22 to 2200**

- Edit the /etc/ssh/sshd\_config file: sudo nano /etc/ssh/sshd\_config.
- Change the port number from 22 to 2200.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: sudo service ssh restart.

**Configure the Uncomplicated Firewall (UFW)**

 - Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
    - sudo ufw status                  # The UFW should be inactive.
   
    - sudo ufw default deny incoming   # Deny any incoming traffic.
   
    - sudo ufw default allow outgoing  # Enable outgoing traffic.
    - sudo ufw allow 2200/tcp              # Allow incoming tcp packets on port  2200.
    -  sudo ufw allow www                   # Allow HTTP traffic in.
    -   sudo ufw allow 123/udp             # Allow incoming udp packets on port  123.
    -   sudo ufw deny 22                        # Deny tcp and udp packets on port 22.

 - Turn UFW on: sudo ufw enable. 

 - Check the status of UFW to list current roles: sudo ufw status.

               The output should be like this:

               Status: active

                To                         Action      From

                --                         ------      ----

              2200/tcp                   ALLOW       Anywhere

               80/tcp                     ALLOW       Anywhere

               123/udp                    ALLOW       Anywhere

               22                         DENY        Anywhere

               2200/tcp (v6)              ALLOW       Anywhere (v6)

               80/tcp (v6)                ALLOW       Anywhere (v6)

               123/udp (v6)               ALLOW       Anywhere (v6)

               22 (v6)                    DENY        Anywhere (v6)
Exit the SSH connection: exit.

**Update Firewall Setting at AWS instence**

- Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above.

Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

From your local terminal, run:

	ssh -i ~/.ssh/lightsail\_key1.rsa -p 2200 [ubuntu@54.169.103.230](mailto:ubuntu@54.169.103.230)

(where 54.169.103.230 is the public IP address of the instance)

**References**

- Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).
- TechRepublic, [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).

**Use Fail2Ban to ban attackers**

- Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks.

	- Install Fail2Ban: sudo apt-get install fail2ban.
	- Install sendmail for email notice: sudo apt-get install sendmail iptables-persistent.
	- Create a copy of a file: sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local.
	- sudo nano /etc/fail2ban/jail.local and added my mail to the destmail
	- Under [sshd] change port = ssh by port = 2200.
	- Restart the service: sudo service fail2ban restart.

**References**

- [Fail2Ban Official website](http://www.fail2ban.org/wiki/index.php/Main_Page).

**Automatically install updates**

- The unattended-upgrades package can be used to automatically install important system updates.
- Enable automatic (security) updates: sudo apt-get install unattended-upgrades
- Edit /etc/apt/apt.conf.d/50unattended-upgrades

	sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

- Uncomment the line ${distro\_id}:${distro\_codename}-updates and save it.
- Modify /etc/apt/apt.conf.d/20auto-upgrades file so that the upgrades are downloaded and  installed every day:

           APT::Periodic::Update-Package-Lists &quot;1&quot;;

           APT::Periodic::Download-Upgradeable-Packages &quot;1&quot;;

           APT::Periodic::AutocleanInterval &quot;7&quot;;

           APT::Periodic::Unattended-Upgrade &quot;1&quot;;

- Enable it: sudo dpkg-reconfigure --priority=low unattended-upgrades.

           sudo apt-get install apache2

- Restart Apache: sudo service apache2 restart.

**References**

- Official Ubuntu Documentation, [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).
- Ubuntu Wiki, [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates).

**Updated packages to most recent versions**

- Some packages have not been updated because the server need to be rebooted.

           sudo apt-get update

           sudo apt-get dist-upgrade

           sudo shutdown -r now

- Logged back in ssh -i ~/.ssh/lightsail\_key1.rsa -p 2200 [ubuntu@54.169.103.230](mailto:ubuntu@54.169.103.230)

**Give grader access**

**Create a new user account named grader**

 - While logged in as ubuntu, add user: sudo adduser grader.
 - Enter a password (twice) and fill out information for this new user.

**Give grader the permission to sudo**

- Edits the sudoers file:

		   sudo touch /etc/sudoers.d/grader

		   sudo nano /etc/sudoers.d/grader

		   Edit the file with following:

		   grader ALL=(ALL) NOPASSWD:ALL

- Save and exit using CTRL+X and confirm with Y.
- Verify that grader has sudo permissions. Run
- su - grader, enter the password,
- Run sudo -l and enter the password again. 

**Resources**
- DigitalOcean, [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

**Create an SSH key pair for grader using the ssh-keygen tool**

   **On the local machine:**

    -Run ssh-keygen
    -Enter file in which to save the key (I gave the name grader\_key1) in the local directory ~/.ssh
    - Enter in a passphrase twice. Two files will be generated ( ~/.ssh/grader\_key1 and ~/.ssh/grader\_key1.pub)
    -Run cat ~/.ssh/grader\_key.pub and copy the contents of the file
    -Log in to the grader&#39;s virtual machine

   **On the grader&#39;s virtual machine:**

    -Create a new directory called ~/.ssh (mkdir .ssh)
    -touch .ssh/authorized\_keys
    -Run sudo nano ~/.ssh/authorized\_keys and paste the content into this file, save and exit
    -Give the permissions: chmod 700 .ssh and chmod 644 .ssh/authorized\_keys
    -Check in /etc/ssh/sshd\_config file if PasswordAuthentication is set to no
    -Restart SSH: sudo service ssh restart

- **On the local machine, run: **

		ssh -i ~/.ssh/grader\_key1 -p 2200 grader@54.169.103.230.


**References**
- DigitalOcean, [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).
- Ubuntu Wiki, [SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys).

**Disable root login**

- $ sudo nano /etc/ssh/sshd\_config
- Find the PermitRootLogin line and edit to no
- $ sudo service ssh restart

**Configure the local timezone to UTC**

- While logged in as grader, configure the time zone:

		sudo dpkg-reconfigure tzdata.


**References**

- Ubuntu Wiki, [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- Ask Ubuntu, [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)

**Install and configure Apache to serve a Python mod\_wsgi application**

- Install Apache sudo apt-get install apache2
- Install mod\_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
- Restart Apache sudo service apache2 restart
- Enable mod\_wsgi

		 $ sudo a2enmod wsgi
		 $ sudo service apache2 start

- Clone the Catalog app from Github

- Install git using: sudo apt-get install git
- cd /var/www
- sudo mkdir catalog
- Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
- cd catalog
- Clone your project from github
- git clone [https://github.com/shikhakhanna19/Product-Catalog.git](https://github.com/shikhakhanna19/Product-Catalog.git) catalog
- Create a catalog.wsgi file, then add this inside:

		import sys

		import logging

		logging.basicConfig(stream=sys.stderr)

		sys.path.insert(0, &quot;/var/www/catalog/&quot;)

		from catalog import app as application

		application.secret_key = "supersecretkey"

- Rename application.py to  **init**.py :

		mv application.py  __init__.py

**Install virtual environment**

- Install pip: sudo apt-get install python-pip
- Install the virtual environment sudo pip install virtualenv
- Create a new virtual environment with sudo virtualenv venv
- Activate the virutal environment source venv/bin/activate
- Change permissions sudo chmod -R 777 venv

- Install Flask and other dependencies

		pip install httplib2

		pip install requests

		pip install --upgrade oauth2client

		pip install sqlalchemy

		pip install flask

		Pip install sqlalchemy\_utils

		pip install psycopg2

- Update path of client\_secrets.json file

		nano __init__.py

- Change client\_secrets.json path to

		/var/www/catalog/catalog/client_secrets.json

- Configure and enable a new virtual host

		Run this: sudo nano /etc/apache2/sites-available/catalog.conf

Paste the following code in this open file:

		<VirtualHost *:80>

		ServerName 54.169.103.230

		ServerAlias ec2-54-169-103-230.ap-southeast-1.compute.amazonaws.com

	        ServerAdmin admin@54.169.103.230

	        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages

	        WSGIProcessGroup catalog

	        WSGIScriptAlias / /var/www/catalog/catalog.wsgi

	        <Directory /var/www/catalog/catalog/>

	        Order allow,deny

	        Allow from all

	       </Directory>

	       Alias /static /var/www/catalog/catalog/static

	       <Directory /var/www/catalog/catalog/static/>

	        Order allow,deny

	        Allow from all

	      </Directory>

               ErrorLog ${APACHE_LOG_DIR}/error.log

              LogLevel warn

              CustomLog ${APACHE_LOG_DIR}/access.log combined

	</VirtualHost>

- Enable the virtual host sudo a2ensite catalog

**Install and configure PostgreSQL**

- sudo apt-get install libpq-dev python-dev
- sudo apt-get install postgresql postgresql-contrib
- sudo su - postgres
- psql
- CREATE USER catalog WITH PASSWORD &#39;password&#39;;
- ALTER USER catalog CREATEDB;
- CREATE DATABASE catalog WITH OWNER catalog;
- \c catalog
- REVOKE ALL ON SCHEMA public FROM public;
- GRANT ALL ON SCHEMA public TO catalog;
- \q
- Exit
- Change create engine line in\_\_init\_\_.py,listofcatalog.py and database\_setup.py to

		engine = create\_engine(&#39;postgresql://catalog:password@localhost/catalog&#39;)

		Run python /var/www/catalog/catalog/database\_setup.py

- Put the client\_id value:

		"502197887292-069vartk0ej9l0qga7mhvel1p6vale40.apps.googleusercontent.com"

In /var/www/catalog/catalog/templates/login.html file in following function

	function start() {

        gapi.load(&#39;auth2&#39;, function() {

        auth2 = gapi.auth2.init({

            client_id:'502197887292-069vartk0ej9l0qga7mhvel1p6vale40.apps.googleusercontent.com';

**NOTE** : Change the following line in /var/www/catalog/catalog/templates/login.html

&lt;link href=&#39;http://fonts.googleapis.com/css?family=Roboto:400,300,700&#39; rel=&#39;stylesheet&#39;  type=&#39;text/css&#39;\&gt;
To
&lt;link href=&#39;https://fonts.googleapis.com/css?family=Roboto:400,300,700&#39; rel=&#39;stylesheet&#39; type=&#39;text/css&#39;\&gt;


**Steps To Create a Self-Signed SSL Certificate for Apache in Ubuntu 16.04**

All the steps that i have done for this step mentioned in **LinuxConfiguration.docx** under same heading

**Reference:**

[https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04)

#Launch the Web Application

- Restart Apache again: sudo service apache2 restart.
- Open your browser type URL

[http](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)[s](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)[://ec2-54-169-103-230.ap-southeast-1.compute.amazonaws.com/](http://ec2-18-196-167-64.us-west-2.compute.amazonaws.com/)

Note: Test user for checking App with facebook

 login : [shikha\_rnijomn\_testuser@tfbnw.net](mailto:shikha_rnijomn_testuser@tfbnw.net) password: awstestuser
