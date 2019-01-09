# linux-server
<p>Udacity  Linux server configuration project</p>

<h2>Server Information</h2>
<p> IP address : 104.196.133.132 </p>
<p>SSH port : 2200</p>
<p>Application URL : http://catalog.104.196.133.132.xip.io</p>

<h2>Configuration Summary</h2>

<h5>1. Start a new Ubuntu Linux server instance on Google Cloud Platform.</h5>

<h5>2.  Update all currently installed packages.</h5>
1.<code> sudo apt-get update</code> <br>
2.<code>sudo apt-get upgrade</code>

<h5>3.  Change the SSH port from 22 to 2200.</h5>
<code>sudo nano /etc/ssh/sshd_config</code>

<h5>change "port 22" to "port 2200".</h5>
<code>sudo service ssh restart</code>

<h5>4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).</h5>
1.<code>sudo ufw allow 2200</code><br>
2.<code>sudo ufw allow 80</code><br>
3.<code>sudo ufw allow 123</code>
<h5>Then we activate ufw</h5>
<code>sudo ufw enable</code>

<h5>5. Create a new user account named (grader).</h5>
<code>sudo adduser grader</code>
<h5>6. Give (grader) the permission to sudo.</h5>
1.<code>sudo touch /etc/sudoers.d/grader</code><br>
2.<code>sudo nano /etc/sudoers.d/grader</code><br>
3.type in file <code>grader ALL=(ALL:ALL) NOPASSWD:ALL</code><br>
4.save and quit

<h5>7. Create an SSH key pair for (grader) using the ssh-keygen tool.</h5>
<h5>1.using ssh-keygen and create a new ssh key at .ssh/grader on local machine</h5>
<h5>2.Copy the contents of grader.pub file.</h5>
<code>su - grader</code><br>
<code>mkdir .ssh</code><br>
<code>nano .ssh/authorized_keys</code><br>
paste the contents and save the file<br>
<code>chmod 700 .ssh</code><br>
<code>chmod 644 .ssh/authorized_keys</code><br>
Restart the ssh service<br>
<code>sudo service ssh restart</code><br>
Now we can connect to the server through<br>
<code>ssh -p 2200 -i ~/.ssh/grader grader@104.196.133.132</code>

<h5>8. Configure the local timezone to UTC.</h5>
1.<code>sudo dpkg-reconfigure tzdata</code><br>
2.Select "None of the above" and then select UTC.

<h5>9. Install and configure Apache to serve a Python mod_wsgi application.</h5>
1.<code>sudo apt-get install apache2</code><br>
2.<code>sudo apt-get install libapache2-mod-wsgi</code><br>
3.<code>sudo apt-get install python</code><br>
4.<code>sudo apt-get install python-setuptools</code><br>
5.Restart Apache <code>sudo service apache2 restart</code>

<h5>10. Install and configure PostgreSQL.<h5>
1.<code>sudo apt-get install postgresql</code><br>
2.Do not allow remote connections.<br>
Check if no remote connections are allowed <code>sudo nano /etc/postgresql/9.5/main/pg_hba.conf</code><br>
3.Create a new database user named (catalog) that has limited permissions to your catalog application database.<br>
<code>sudo -u postgres psql</code><br>
<code>create user catalog with password 'your-password';</code><br>
<code>create database catalog with owner catalog;</code><br>
<code>\q</code>

<h5>11. Install git.</h5>
<code>sudo apt-get install git</code>

<h5>12. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.</h5>
<code>cd /var/www/</code><br>
<code>sudo git clone https://github.com/amjadzaghloul/catalog.git</code><br>
<code>cd catalog</code><br>
<code>sudo mv application.py __init__.py</code><br>
<code>sudo nano __init__.py</code><br>
Make the necessary changes to connect to the postgresql database <br>
<code>sudo nano /etc/apache2/sites-available/catalog.conf</code><br>

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
<code>cd /var/www/catalog/</code><br>
<code>sudo nano wsgi.py</code><br>
<pre><code>import sys
sys.path.insert(0, '/var/www/catalog')
from __init__ import app as application
application.secret_key = 'super_secret_key'
application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'catalog:yuor-password@localhost/catalog')
</code></pre>

Restart Apache <code>sudo service apache2 restart</code>

<h2>References:</h2>

http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/