***Sparky Linux Services: Web Hosting***

Here we will be going over how we can setup web hosting withh our Sparky Linux VM.

Sparky Linux is a lightweight Debian-based distro that we will be using for setting-up the majority of our services.
It has a really clean GUI, and doesn't take much space on our system to spin up.

First after installing Sparky Linux, let's set it up in Oracle Virtual Box. Here is the configuration I used for my setup:
 - Name: Sparky Services
 - Type: Linux
 - Subtype: Debian
 - Version: Debian (64-bit)
 - Base Memory: 5053 MB
 - Processors: 2 Cores
 - Storage: 60 GB

After finishing the first set of options, select your VM and go into the following:
Settings >> Network >> Adapter 2 >> Attached to Internal Network

*Make sure the name of the internal adapter is the same as the one we created before with our router*

Once Sparky boots up, we will have our user called "live" which we can use to then create our designated users for each of our services.

Let's create our user for our webservices using the following commands:
 - sudo useradd webmaster
 - sudo usermod -aG www-data webmaster
 - sudo usermod -s /bin/bash webmaster
 - sudo mkdir -p /var/www/webmaster
 - sudo mkdir -p /home/webmaster
 - sudo chown -R webmaster:www-data /var/www/webmaster
 - sudo chown -R webmaster:webmaster /home/webmaster
 - sudo chmod -R 775 /var/www/webmaster
 - sudo chmod -R 700 /home/webmaster

Now that user has been created let's install the services we will be using for this lab.
We will be installing Apache, PHP, and PostgreSQL.

Let's first start by updating and upgrading our system.
 - sudo apt update && sudo apt upgrade -y

After updating our system, let's install apache using *systemctl*:
 - sudo apt install apache2 -y

After installing apache, let's start, enable, then check the status of our service to ensure that is fully operational.
 - sudo systemctl enable apache2
 - sudo systemctl start apache2
 - sudo systemctl status apache2

After seeing that the apache service is running, let's now go and check out our webpage using the Firefox browser. Best way to go about this is by using another VM and accessing the site via Spark's IP.

As we can see, the default Apache web page was loaded successfully from our Kali VM. 

Now that our initial apache setup is complete, lets move onto PHP.

Installing php can be done with the following command:
 - sudo apt install php libapache2-mod-php php-mysql

After running the apt install command from above, we can now test to see if our PHP is funcitonal.
We can attempt this by creating a test php file, and accesssing it from our web browser.

Let's first run this command, then access this site to check if PHP is working (http://YourIP/info.php)

As we can see by the screenshot, PHP is functional for our webpage, so we can move onto installing MySQL.

To install PostgreSQL, we can use the following command:
 - sudo apt install postgresql postgresql-contrib

After installing postgresql, we need to start, enable, and check the status of our service to ensure it is functional.
 - sudo systemctl enable postgresql
 - sudo systemctl start postgresql
 - sudo systemctl status postgresql

With everything now installed, let's restart our apache2 service to ensure everything is set.
 - sudo systemctl restart apache2




