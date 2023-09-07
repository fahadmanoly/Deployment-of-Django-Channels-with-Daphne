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
  
  #CTRL+C to exit.
  
  sudo apt install net-tools   -   Confirm Redis is running at 127.0.0.1. Port should be 6379 by default.
  
  sudo netstat -lnp | grep redis
  
  sudo systemctl restart redis.service


**6) ASGI for Hosting Django Channels as a Separate Application**

    Your asgi.py file inside your project should be like below. It will come automatically like this. Need to ensure only here. dont edit


                        import os
                        import django
                        from channels.routing import ProtocolTypeRouter, URLRouter
                        from django.core.asgi import get_asgi_application
                        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'bandhanback.settings')
                        os.environ["DJANGO_ALLOW_ASYNC_UNSAFE"] = "true"
                        django.setup()
                        from chat.routing import websocket_urlpatterns
                        
                        application = ProtocolTypeRouter({
                            'http': get_asgi_application(),
                            'websocket': URLRouter(websocket_urlpatterns),
                        })


**7) Deploying Django Channels with Daphne & Systemd**

  #Navigate to root, I mean Ubuntu and install Daphne(It should be on Ubuntu, not inside the project directory)

  Sudo apt install daphne

  Navigate to /etc/systemd/system/

  Create daphne.service. Notice the port is 8001. To do this use the following command and should be inside /etc/systemd/system

  Sudo nano daphne.service and paste the follwoing

                        [Unit]
                        Description=WebSocket Daphne Service
                        After=network.target
                        
                        [Service]
                        Type=simple
                        User=root
                        WorkingDirectory=/home/ubuntu/dir 1/dir 2/your project directory where we can find the manage.py file
                        ExecStart=/home/ubuntu/dir/1/project dir where we can find venv file/venv/bin/python /home/ubuntu/dir 1/project dir where we can find venv file/venv/bin/daphne -b 0.0.0.0 -p 8001 your project name.asgi:application
                        Restart=on-failure
                        
                        [Install]
                        WantedBy=multi-user.target


 systemctl daemon-reload

 systemctl start daphne.service

 systemctl status daphne.service

 #Check the status and exit by typing cntr+c


**8) Starting the daphne Service when Server boots**

 Create the script to run daphne in the root folder. Please make sure that it is in the root folder. To Navigate to root after reaching the ubuntu folder we should press cd .. twice to reach to root. after reaching root if we type 'ls' then we would be able to see the boot files. DO teh following after reaching there
 
 #create boot.sh(here we wont have permission to get access. so too get access we should login to the terminal of root by typing 'sudo bash' 

 #!/bin/sh
 
 sudo systemctl start daphne.service
 
 Save and close.

 Type exit

 #we will reach to ubuntu again when we exit from bash, there

 chmod u+x /root/boot.sh

 Now Navigate to /etc/systemd/system

 #there create on_boot.service using the following command

 sudo bash - get into that terminal again for hte permission to access and edit

 sudo nano _boot.service and paste the following

 
                         [Service]
                        ExecStart=/root/boot.sh
                        
                        [Install]
                        WantedBy=default.target

Save and close.

type exit and exit from bash to the ubuntu

sudo systemctl daemon-reload       -  Reloading the boot service

sudo systemctl start on_boot       -  Starting the boot service

sudo systemctl enable on_boot      -  Enabling it to run at boot

ufw allow 8001                     -  Allowing daphne service through firewall

sudo shutdown -r now               -  Restarting the server

systemctl status on_boot.service   -  Check the status of on_boot.service

sudo journalctl -u on_boot.service -  Taking journals to see the logs(here we should see that completed and successfully deactivated)

systemctl status daphne.service    -  Check if the daphne service started when the server started

#**Purchase a domain and make https to give it on domain name.

 

 









                  



