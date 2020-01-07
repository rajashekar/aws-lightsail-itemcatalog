# AWS lightsail itemcatalog

The purpose of the project is to create a brand-new, bare bones, Linux server into the secure and efficient web application. In this project item catalog is deployed on AWS light sail server.

# Getting started
Item catalog app is hosted on 
- IP address - 35.155.196.149
- SSH Port - 2200
- Item catalog can be accessed through http://35.155.196.149.xip.io/

# Configurations made to enable firewall & to create new user
- Updated & upgraded all packages using `sudo apt-get update && sudo apt-get upgrade`
- Changed default SSH port from 22 to 2200
- Added ufw rule 
    * To deny all incoming - `sudo ufw default deny incoming`
    * To allow all outgoing - `sudo ufw default allow outgoing`
    * Opened port 2200 - `sudo ufw allow 2200/tcp`
    * Opened port 80 - `sudo ufw allow 80/tcp`
    * Opened port 123 - `sudo ufw allow 123/tcp`
- After adding above ufw rules enabled ufw - `sudo ufw enable`
- Verified ufw status using - `sudo ufw status verbose`   
- Removed root SSH access 
- Removed password access
- Created new user `grader`
- Gave sudo access to user `grader`
- Generated SSH keys using ssh-keygen for `grader`
- Added SSH public key to `~/.ssh/authorized_keys`

# Configurations made & libraries installed to host item-catalog app
- For Web server installed `sudo apt-get install apache2`
- To host python based application `item-catalog`, a Web server Gateway Interface is needed, so installed `sudo apt-get install libapache2-mod-wsgi python-dev`
- `item-catalog` uses sqlite3 database, to verify/query data, installed sqlite3 client - `sudo apt-get install sqlite3 libsqlite3-dev` 
- Verified data using `sqlite3 itemcatalog.db`
- `item-catalog` requres below python modules like `flask`, `requests` etc. So installed `pip` - `sudo apt-get install python-pip python-dev build-essential`
- Upgraded pip - `sudo pip install --upgrade pip`
- Installed required python modules 
    * flask - `sudo pip install -U Flask`
    * oauth - `sudo pip install --upgrade oauth2client`
    * sqlalchemy - `sudo pip install --upgrade sqlalchemy`
    * requests - `sudo pip install --upgrade requests`
- Verified item-catalog by running `python catalog_views.py`   
- Created file `/var/www/html/myapp.wsgi` to refer item-catalog
    ```
    import sys
    sys.path.append('/var/www/html/item-catalog')
    from catalog_views import app as application
    ```  
- Added myapp.wsgi to `/etc/apache2/sites-enabled/000-default.conf` at the end of `<VirtualHost *:80>` 
    ```
    WSGIScriptAlias / /var/www/html/myapp.wsgi
    ```
- Restarted apache server using `sudo apache2ctl restart`

# Errors faced & fixes done
- Saw below error in `/var/log/apache2/error.log`
    ```
    OperationalError: (sqlite3.OperationalError) unable to open database file
    ```
    When application is run through wsgi, the paths are not getting resolved correctly so used absolute paths. Changed path from `sqlite:///itemcatalog.db` to `sqlite:////var/www/html/item-catalog/itemcatalog.db`. Done the same for `client_secret_google.json` changed to `/var/www/html/item-catalog/client_secret_google.json`
    
- Saw threading excepitons like below 
    ```
    ProgrammingError: (sqlite3.ProgrammingError) SQLite objects created in a thread can only be used in that same thread.
    ```    
    Since global session is used saw above errors, so change session to session scoped, so removed 
    ```
    DBSession = sessionmaker(bind=engine)
    session = DBSession()
    ```
    Added 
    ```
    session = scoped_session(sessionmaker(bind=engine))
    ..
    @app.teardown_request
    def remove_session(ex=None):
      session.remove()
    ```

Once all above configurations are done able to run item-catalog application, below is the demo. 

# Demo
![Demo](demo.png?raw=true "Demo")    
