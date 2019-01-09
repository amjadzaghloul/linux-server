# linux-server
<p>Udacity  Linux server configuration project</p>

<h2>Server Information</h2>
<p> IP address : 104.196.133.132 </p>
<p>SSH port : 2200</p>
<p>Application URL : http://catalog.104.196.133.132.xip.io</p>

<h2>Configuration Summary</h2>

<h5>1. Start a new Ubuntu Linux server instance on Google Cloud Platform.</h5>

<h5>2.  Update all currently installed packages.</h5>
1.`sudo apt-get update` <br>
2.`sudo apt-get upgrade`

<h5>3.  Change the SSH port from 22 to 2200.</h5>
`sudo nano /etc/ssh/sshd_config`

<h5>change "port 22" to "port 2200".</h5>
`sudo service ssh restart`

<h5>4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).</h5>
1.`sudo ufw allow 2200`<br>
2.`sudo ufw allow 80`<br>
3.`sudo ufw allow 123`
<h5>Then we activate ufw</h5>
`sudo ufw enable`

<h5>5. Create a new user account named (grader).</h5>
`sudo adduser grader`
<h5>6. Give (grader) the permission to sudo.</h5>
1.`sudo touch /etc/sudoers.d/grader`<br>
2.`sudo nano /etc/sudoers.d/grader`<br>
3.type in file `grader ALL=(ALL:ALL) NOPASSWD:ALL`<br>
4.save and quit

<h5>7. Create an SSH key pair for (grader) using the ssh-keygen tool.</h5>
<h5>1.using `ssh-keygen` and create a new ssh key at .ssh/grader on local machine</h5>
<h5>2.Copy the contents of grader.pub file.</h5>
`su - grader`<br>
`mkdir .ssh`<br>
`nano .ssh/authorized_keys`<br>
paste the contents and save the file<br>
`chmod 700 .ssh`<br>
`chmod 644 .ssh/authorized_keys`<br>
Restart the ssh service<br>
`sudo service ssh restart`<br>
Now we can connect to the server through<br>
`ssh -p 2200 -i ~/.ssh/grader grader@104.196.133.132`

<h5>8. Configure the local timezone to UTC.</h5>
1.`sudo dpkg-reconfigure tzdata`<br>
2.Select "None of the above" and then select UTC.

<h5>9.  Install and configure Apache to serve a Python mod_wsgi application.</h5>
1.`sudo apt-get install apache2`<br>
2.`sudo apt-get install libapache2-mod-wsgi`<br>
3.`sudo apt-get install python`<br>
4.`sudo apt-get install python-setuptools`<br>
5.Restart Apache `sudo service apache2 restart`

<h5>10. Install and configure PostgreSQL.<h5>
1.`sudo apt-get install postgresql`<br>
2.Do not allow remote connections.<br>
Check if no remote connections are allowed `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`<br>
3.Create a new database user named (catalog) that has limited permissions to your catalog application database.<br>
`sudo -u postgres psql`<br>
`create user catalog with password 'your-password';`<br>
`create database catalog with owner catalog;`<br>
`\q`

<h5>11. Install git.</h5>
`sudo apt-get install git`

<h5>12. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.</h5>
`cd /var/www`<br>
`sudo git clone https://github.com/amjadzaghloul/catalog.git`<br>
`cd catalog`<br>
`sudo mv application.py __init__.py`<br>
`sudo nano __init__.py`<br>
Make the necessary changes to connect to the postgresql database <br>
`sudo nano /etc/apache2/sites-available/catalog.conf`<br>

<pre><code>&lt;VirtualHost *:80&gt;
    ServerName 104.196.133.132
    WSGIScriptAlias / /var/www/catalog/wsgi.py
    &lt;Directory /var/www/catalog&gt;
        Order allow,deny
        Allow from all
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;
</code></pre>

Create the wsgi file<br>
`cd /var/www/catalog/`<br>
`sudo nano wsgi.py`<br>
<pre><code>import sys
sys.path.insert(0, '/var/www/catalog')
from __init__ import app as application
application.secret_key = 'super_secret_key'
application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'catalog:yuor-password@localhost/catalog')
</code></pre>

Restart Apache `sudo service apache2 restart`

<h2>References:</h2>

http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/