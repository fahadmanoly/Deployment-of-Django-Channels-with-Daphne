# Deployment-of-Django-Channels-with-Daphne in AWS Instance

**1)Install Server Dependencies**

sudo apt update

sudo apt install python3-pip python3-dev nginx curl

sudo -H pip3 install --upgrade pip

sudo -H pip3 install virtualenv

sudo apt install git-all

sudo apt install libgl1-mesa-glx

mkdir < project directory1 >

cd < project directory1 >

virtualenv venv

source venv/bin/activate

mkdir  < project directory2 > 

cd < project directory2 >

git init
git pull https://github.com/your git id/your repository.git branch 

pip install -r requirements.txt

pip install django gunicorn

python manage.py makemigrations

python manage.py migrate

python manage.py runserver 0.0.0.0:8000

visit http://<your_ip_address>:8000/  --in order to check whether its working or not


**2) Gunicorn configuration**

only do this if first is working properly

Navigate to /etc/systemd/system/

#Create a file named: gunicorn.socket and paste the code below. for that use sudo nano gunicorn.socket


            [Unit]
            Description=gunicorn socket
            
            [Socket]
            ListenStream=/run/gunicorn.sock
            
            [Install]
            WantedBy=sockets.target

#Create another file named: gunicorn.service and paste the code below. for that use sudo nano gunicorn.service


            [Unit]
            Description=gunicorn daemon
            Requires=gunicorn.socket
            After=network.target
            
            [Service]
            User=ubuntu
            Group=www-data
            WorkingDirectory=/home/ubuntu/dir 1/dir 2/projectname(where we can find manage.py)
            ExecStart=/home/ubuntu/dir 1/dir 2/venv/bin/gunicorn \
                      --access-logfile - \
                      --workers 3 \
                      --bind unix:/run/gunicorn.sock \
                      CodingWithMitchChat.wsgi:application
            
            [Install]
            WantedBy=multi-user.target

sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

**3) Nginx configuration**

Navigate to /etc/nginx/sites-available

#Create a file named: any_name(example - gunicorn_name) and paste the code below. for that use sudo nano gunicorn_name


              server {
                  server_name <your_ip_address from AWS instance>;
                  location = /favicon.ico { access_log off; log_not_found off; }
                  location /static/ {
                      root /home/ubuntu/dir 1/dir 2/projectname(where we can find manage.py);
                  }
               
                   location / {
                      include proxy_params;
                      proxy_pass http://unix:/run/gunicorn.sock;
                  }
              }

#Update Nginx config file at /etc/nginx/nginx.conf so we can upload large file images-(do the folowing for this)

#Navigate to /etc/nginx/

#if you type 'ls' you can see a file namely 'nginx.conf' whcih we cant access directly. so to access this we need to do the following command

sudo bash

#now we will be logged to another terminal of bash which we have permission to access all files , so from there type the following

sudo nano nginx.conf (#and paste the following at the end)


            http{
            	...
            	client_max_body_size 10M; (# we need to enter only this line . http and brackets is already there with lots of other codes)
            }

**4) Configure the firewall**

sudo ln -s /etc/nginx/sites-available/gunicorn_name /etc/nginx/sites-enabled

sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 8000

sudo ufw allow 'Nginx Full'

sudo systemctl restart gunicorn

Set DEBUG=False in settings.py if it isn't already.

sudo shutdown -r now (# Restarting the server. connection will be lost. you need to connect again after 45 seconds)

#Now visit http://<your_ip_address>/ to checkbits working.(If it is showing the gunicorn page, it means that the default server is working, not the custom server(gunicorn_name))

 In order to make the custom server gunicorn_name to work we can use the following helpful command
 
                  sudo systemctl daemon-reload                 -   Must be executed if you change the gunicorn.service file.
                  
                  sudo systemctl restart gunicorn              -   If you change code on your server you must execute this to see the changes take place.
                  
                  sudo systemctl status gunicorn
                  
                  sudo journalctl                              -   All the logs are consolidated here
                  
                  sudo tail -F /var/log/nginx/error            -   log View the last entries in the error log
                  
                  sudo journalctl -u nginx                     -   Nginx process logs
                  
                  sudo less /var/log/nginx/access.log          -   Nginx access logs
                  
                  sudo less /var/log/nginx/error.log           -   Nginx error logs
                  
                  sudo journalctl -u gunicorn                  -   gunicorn application logs
                  
                  sudo journalctl -u gunicorn.socket           -   check gunicorn socket logs
                  
                  sudo shutdown -r now                         -   Restart the server

                  
**5) Install and Configure Redis**

  sudo apt install redis-server
  
  Navigate to /etc/redis/
  
  #if you type 'ls' you can see a file namely 'redis.conf' whcih we cant access directly. so to access this we need to do the following command
  
  sudo bash
  
  #now we will be logged to another terminal of bash which we have permission to access all files , so from there type the following
  
  sudo nano redis.conf 
  
  #type cntrl+F t find the word 'supervised no' and replace that 'supervised no' to 'supervised systemd' the save and exit
  
  sudo systemctl restart redis.service
  
  sudo systemctl status redis   (should see the status as active after entering this code)
  
  # CTRL+C to exit.
  
  sudo apt install net-tools   -   Confirm Redis is running at 127.0.0.1. Port should be 6379 by default.
  
  sudo netstat -lnp | grep redis
  
  sudo systemctl restart redis.service


**6) ASGI for Hosting Django Channels as a Separate Application**



                  



