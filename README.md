# Linux-Server

Project 7 - Setting up Linux Server under the Full Stack Web Developer Nanodegree at Udacity


## Notes for reviewer:
* public Ip: `34.205.43.91`
* SSH PORT: `2200`
* Full project URL:[link](http://ec2-34-205-43-91.compute-1.amazonaws.com/)



### Tasks given and method for completion:

* Start a new Ubuntu Linux server instance on Amazon Lightsail.

* Launch Amazon Lightsail terminal
    * loggin into your Amazon Web Services account.
    * Visit this [link](https://lightsail.aws.amazon.com/) and Create new instance of Ubuntu.
    * You will get your respective public IP address.
    * Download the default key-pair and copy to /.ssh folder.
    * Now in terminal type chmod 600 ~/.ssh/key.pem
    * Type the command `ssh -i ~/.ssh/key.pem ubuntu@34.205.43.91` to create the instance on your terminal

* Create a new user named grader
    * type `sudo adduser grader`
    * install finger `sudo apt-get install finger` to check user has been added or not.
    * type `finger grader` to check.


* Give the permission of sudo to grader
    * Type `sudo visudo`
    * inside the opened file add `grader   ALL=(ALL:ALL) ALL` below the root user under "#User privilege specification"
    * Now the save file(nano: `ctrl+x`, `Y`, Enter)
    * Add grader to `/etc/sudoers.d/` by type in `grader   ALL=(ALL:ALL) ALL` by command `sudo nano /etc/sudoers.d/grader`
    * Add root to `/etc/sudoers.d/` by type in `root   ALL=(ALL:ALL) ALL` by command `sudo nano /etc/sudoers.d/root`


* Update all currently installed packages
    * Find updates:`sudo apt-get update`
    * Install updates:`sudo sudo apt-get upgrade` Hit Y for "Yes".

* Change the SSH port from 22 to 2200 and other SSH configurations required from [grading rubic](https://www.udacity.com/course/viewer#!/c-nd004/l-3573679011/m-3608778867)
    * Now type `sudo nano /etc/ssh/sshd_config` add `port 2200` below `port 22`
    * while in the file also change `PermitRootLogin prohibit-password` to `PermitRootLogin no` to disallow root login
    * Change `PasswordAuthentication` from `no` to `yes`. We will change back after finishing SHH login setup
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * restart ssh service by command `sudo service ssh reload`


* Create SSH keys and copy to server manually:
    * On your local machine generate SSH key pair with command: `ssh-keygen`
    * save your keygen file in your ssh directory `/home/ubuntu/.ssh/` example full file path that could be used: `/home/ubuntu/.ssh/item-catalog`
    * You can add a password to use incase your keygen file gets compromised(you will be prompted to enter this password when you connect with key pair)
    * Change the SSH port number configuration in Amazon lightsail in networking tab to 2200.
    * login into grader account using password set during user creation by command `ssh -v grader@34.205.43.91 -p 2200`
    * Now make .ssh directory `mkdir .ssh`
    * then make a file to store key `touch .ssh/authorized_keys`
    * On your local machine read contents of the public key `cat .ssh/project5.pub`
    * Copy the key and paste in the file you just created in grader `nano
.ssh/authorized_keys` paste contents(ctr+v)
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Set permissions for files:
        * `chmod 700 .ssh`
        * `chmod 644 .ssh/authorized_keys`
    * Now change `PasswordAuthentication` from `yes` back to `no`. by typing command `sudo nano /etc/ssh/sshd_config`
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Now login with key pair: `ssh grader@34.205.43.91 -p 2200 -i ~/.ssh/item-catalog`


* Configure the Uncomplicated Firewall (UFW) to only allow  incoming connections for SSH (port 2200), HTTP (port 80),  and NTP (port 123)
    * Check UFW status to make sure its inactive by command `sudo ufw status`
    * Deny all incoming by default `sudo ufw default deny incoming`
    * Allow outgoing by default `sudo ufw default allow outgoing`
    * Allow SSH `sudo ufw allow ssh`
    * Allow SSH on port 2200 `sudo ufw allow 2200/tcp`
    * Allow HTTP on port 80 `sudo ufw allow 80/tcp`
    * Allow NTP on port 123 `sudo ufw allow 123/udp`
    * Now Turn on firewall `sudo ufw enable`


* Configure the local timezone to UTC
    * run `sudo dpkg-reconfigure tzdata`
    * from prompt:
      * Select none of the above.
      * Then select UTC.


* Install and configure Apache to serve a Python mod_wsgi application
    * run to install apache `sudo apt-get install apache2` Check if "It works!" at you public IP address given during setup.
    * install mod_wsgi: run `sudo apt-get install libapache2-mod-wsgi`
    * configure Apache to handle requests using the WSGI module `sudo nano /etc/apache2/sites-enabled/000-default.conf`
    * Now add `WSGIScriptAlias / /var/www/html/myapp.wsgi` before `</VirtualHost>` closing line
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Restart the Apache run `sudo apache2ctl restart`


* Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

* install git
    * run `sudo apt-get install git`

* install python dev and verify WSGI is enabled
    * Install python-dev package run `sudo apt-get install python-dev`
    * Verify wsgi is enabled `sudo a2enmod wsgi`
* Create flask app taken from [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
    * `cd /var/www`
    * `sudo mkdir catalog`
    * `cd catalog`
    * `sudo mkdir catalog`
    * `cd catalog`
    * `sudo mkdir static templates`
    * `sudo nano __init__.py `

    ```
    from flask import Flask
    app = Flask(__name__)
    @app.route("/")
    def hello():
        return "Hello, world (Testing!)"
    if __name__ == "__main__":
        app.run()
    ```

* install flask by running following commands
    * `sudo apt-get install python-pip`
    * `sudo pip install virtualenv `
    * `sudo virtualenv venv`
    * `sudo chmod -R 777 venv`
    * `source venv/bin/activate`
    * `pip install Flask`
    * `python __init__.py`
    * `deactivate`

* Configure And Enable New Virtual Host
    * Create host config file run `sudo nano /etc/apache2/sites-available/catalog.conf`
    * paste the following:

    ```
    <VirtualHost *:80>
      ServerName 34.205.43.91
      ServerAdmin admin@34.205.43.91
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
```
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Enable it by running `sudo a2ensite catalog`

* Create the wsgi file by running following commands
    * `cd /var/www/catalog`
    * `sudo nano catalog.wsgi`

    ```
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")

        from catalog import app as application
        application.secret_key = 'Add your secret key'
  ```

  * save file(nano: `ctrl+x`, `Y`, Enter)

  * run `sudo service apache2 restart`

* Clone Github Repo
    * `sudo git clone https://github.com/thakur295/item_catalog`
    * make sure you get hidden files iin move `shopt -s dotglob`. Move files from clone directory to catalog ` sudo mv /var/www/catalog/item-catalog/* /var/www/catalog/catalog/`
    * remove clone directory `sudo rm -r item_catalog`

* make .git inaccessible
    * from `cd /var/www/catalog/` create .htaccess file `sudo nano .htaccess`
    * paste in `RedirectMatch 404 /\.git`
    * save file(nano: `ctrl+x`, `Y`, Enter)

* install dependencies by running following commands:
    * `source venv/bin/activate`
    * `pip install httplib2`
    * `pip install requests`
    * `sudo pip install oauth2client`
    * `sudo pip install sqlalchemy`
    * `pip install SQLAlchemy`
    * `sudo pip install psycopg2`
    * If you used any other packages in your project be sure to install those as well.


* Install and configure PostgreSQL:
    * Install postgres`sudo apt-get install postgresql`
    * install additional models`sudo apt-get install postgresql-contrib`
    * config database_setup.py `sudo nano database_setup.py`
    * `engine = create_engine('postgresql://catalog:qwerty@localhost/catalog')`
    * repeat for project.py
    * copy your main project.py file into the __init__.py file by `sudo mv project.py __init__.py`
    * Add catalog user `sudo adduser catalog`
    * login as postgres super user by running `sudo su - postgres`
    * enter postgres `psql`
    * Create user catalog`CREATE USER catalog WITH PASSWORD 'qwerty';`
    * Change role of user catalog to creatDB` ALTER USER catalog CREATEDB;`
    * List all users and roles to verify `\du`
    * Create new DB "catalog" with own of catalog`CREATE DATABASE catalog WITH OWNER catalog;`
    * Connect to database `\c catalog`
    * Revoke all rights `REVOKE ALL ON SCHEMA public FROM public;`
    * Give accessto only catalog role`GRANT ALL ON SCHEMA public TO catalog;`
    * Quit postgres `\q`
    * logout from postgres super user `exit`
    * Setup your database schema by running `python database_setup.py`



* fix OAuth to work with hosted Application
    * Google wont allow the IP address to make redirects so we need to set up the host name address to be usable.
    * go to [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi) in order to get your host name by entering your public IP address that you get above.
    * open apache configbfile `sudo nano /etc/apache2/sites-available/catalog.conf`
    * below the `ServerAdmin` paste `ServerAlias 34.205.43.91`
    * make sure the virtual host is enabled `sudo a2ensite catalog`
    * restart apache server `sudo service apache2 restart`
    * in your google developer console add your host name and IP address to Authorized Javascript origins. And add http://34.205.43.91/ouath2callback to the Authorized redirect URIs.
