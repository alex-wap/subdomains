# Multiple Subdomains Tutorial


This tutorial teaches developers how to deploy multiple apps to the same AWS EC2 instance using Nginx as a reverse proxy and how to configure subdomains. (Node and [Pylot](https://github.com/Ketul-Patel/Pylot/tree/development) apps)

---

# Instructions


## 1. First, configure at least one application for deployment. Follow the instructions below for the corresponding type of app.

###[Node Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/node_deploy.html)

###[Pylot Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/pylot_deploy.html)

###[Rails Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/rails_deploy.html)

###[Django Deployment](https://github.com/alex-wap/DjangoDeployment)

---

## 2. All applications must be on a different port.


#### Node: Typically, the port is specified in the server.js file or a settings.js file using: 
```javascript
var server = app.listen(8001);
```
#### Pylot: The port must be modified in both manage.py and wsgi.py.
##### manage.py:
```python
manager.add_command('runserver', Server(host='127.0.0.1',port='5001'))
# NOTE: port must be a string.
```  

##### wsgi.py: 
```python
application.run(host='127.0.0.1',port=5001)
# NOTE: port must be an integer.
```  

#### Rails: You don't need to specify ports because of [Phusion Passenger](https://www.phusionpassenger.com/library/walkthroughs/basics/ruby/fundamental_concepts.html). 

#### Django: Replace your `manage.py` file (do not forget to replace PROJECTNAME and PORT with your project name and port number)
##### manage.py:
```python
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "PROJECTNAME.settings")
    
    # Different port manage.py file 
    import django
    django.setup()

    # Override default port for `runserver` command
    from django.core.management.commands.runserver import Command as runserver
    runserver.default_port = "PORT"

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
```
---

## 3. Configure a __single__ Nginx file for all apps.


#### Overwite any existing Nginx files with this format in __**one**__ file. In this example file, the first three server configs are for Node and the last two server configs are for Pylot.
  * DOMAIN.com must be replaced by the domain name
  * PROJECT#.DOMAIN.com must be replaced by the desired subdomain name
  * PORT* must be replaced by the project's port
  * PROJECT#.sock must match the configuration of the project.ini file
```
# Node example (replace PORT1)
server {
  server_name DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT1;
    }
}
# Node example 2 (replace PROJECT1 and PORT2)
server {
  server_name PROJECT1.DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT2;
    }
}
# Pylot example (replace PROJECT2 and PORT3)
server {
  server_name PROJECT2.DOMAIN.com;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass       http://127.0.0.1:PORT3;
    include uwsgi_params;
    uwsgi_pass unix:/home/ubuntu/PROJECT2/PROJECT2.sock;
    }
}
# Rails example (replace PROJECT3 and FOLDER)
server {
    listen 80;
    server_name PROJECT3.DOMAIN.com;
    passenger_enabled on;
    passenger_app_env development;
    root /var/www/FOLDER/public;
}
# Django example (replace REPONAME and PROJECT4)
server {
      listen 80;
      server_name PROJECT4.DOMAIN.com;
      location = /favicon.ico { access_log off; log_not_found off; }
      location /static/ {
            root /home/ubuntu/REPONAME;
      }
      location / {
            include proxy_params;
            proxy_pass http://unix:/home/ubuntu/REPONAME/PROJECT4.sock;
      }
}

```
---

## 4. Deploy other apps to the existing AWS EC2 instance.

#### Node:
```bash 
cd /var/www
sudo git clone {{project file path on github}}
cd /var/www/{{project_name}}
sudo npm install
pm2 start server.js
sudo service nginx reload && sudo service nginx restart
```
#### Pylot: 
follow instructions in highlighted blue lines via [Pylot Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/pylot_deploy.html) (except for Nginx section)
```bash 
sudo service nginx reload && sudo service nginx restart
```
#### Rails: 
follow instructions via [Rails Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/rails_deploy.html) (except for Nginx section)
```bash 
sudo service nginx restart
```
#### Django: 
follow instructions via [Django Deployment](https://github.com/alex-wap/DjangoDeployment) (except for Nginx section)
Ubuntu 14.04:
```bash 
sudo service gunicorn restart
sudo service nginx reload && sudo service nginx restart
```
Ubuntu 16.04:
```bash 
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo service nginx restart
```
---

## 5. Verify all the apps are running.
#### Node:
```bash 
pm2 status
```
#### Pylot:
```bash 
sudo start PROJECT
```
#### Rails:
```bash 
sudo service nginx restart
```
#### Django (Ubuntu 14.04):
```bash 
sudo service gunicorn restart
sudo service nginx restart
```
#### Django (Ubuntu 16.04):
```bash 
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo service nginx restart
```
---

## 6. Go to your domain name registrar and find *Advanced DNS.*

#### Set up an "A Record" with "Host" set to @ and "Value" set to {{IP ADDRESS OF AWS INSTANCE}}
#### Set up "CNAME Record" with "Host" set to {{SUBDOMAIN}} and Value set to {{DOMAIN NAME}}

![Subdomain Configuration](https://raw.githubusercontent.com/alex-wap/subdomains/master/Subdomains.png "Subdomain Configuration")


---

## 7. Congratulations! We're done!


Note: It may take some time for subdomains to propagate (up to 24 hours). Be sure to test out the domain and subdomains.
