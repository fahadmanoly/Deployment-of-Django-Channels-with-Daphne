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



**9) Domain Setup**

 #Purchase a domain and setup the ip address of the aws instance in it

 Update Nginx config. do the following to do it

 Navigate to /etc/nginx/sites-available

 update the gunicorn whcih was setup earlier.(follow the below steps to do that)

 sudo nano gunicorn_name  and the give your domain name after the ip address where it is specified

                           server {
                              server_name ip_address of AWS instance  purchased_domain_name.xyz(or whatever)  www.purchased_domain_name.xyz(or whatever)
                              location = /favicon.ico { access_log off; log_not_found off; }
                              location /static/ {
                                  root /home/ubuntu/dir 1/dir 2/projectname(where we can find manage.py);
                              }
                           
                               location / {
                                  include proxy_params;
                                  proxy_pass http://unix:/run/gunicorn.sock;
                              }
                          }

 sudo systemctl reload nginx

 sudo nginx -t     -  Make sure nginx configuration is still good.

 #Update the allowed host in settings.py in the project file

              ALLOWED_HOSTS =  [ "ip_addresss", "purchased_domain_name.xyz", "www.purchased_domain_name.xyz" ]


#Apply the changes

service gunicorn restart

#Now its time to wait to get into effect. It may long upto maximum 1 hour


**10) Making HTTPS**

 sudo apt install certbot python3-certbot-nginx

 sudo systemctl reload nginx

 sudo ufw allow 'Nginx Full'

 sudo ufw delete allow 'Nginx HTTP'

 sudo certbot --nginx -d <your-domain.whatever> -d www.<your-domain.whatever>

 sudo systemctl status certbot.timer

 sudo certbot renew --dry-run

**Update Nginx**

 Navigate to /etc/nginx/sites-available

 sudo nano nginx_name

                        server {
                        
                            ...
                            
                            location /ws/ {
                                proxy_http_version 1.1;
                                proxy_set_header Upgrade $http_upgrade;
                                proxy_set_header Connection "upgrade";
                                proxy_redirect off;
                                proxy_pass http://127.0.0.1:8001;
                            }
                        
                            ...
                        }

**Update Daphne**

 Navigate to /etc/systemd/system

 sudo nano daphne.service

 
 [Unit]
                        Description=WebSocket Daphne Service
                        After=network.target
                        
                        [Service]
                        Type=simple
                        User=root
                        WorkingDirectory=/home/ubuntu/proj directory/project directory 2 where we can find manage.py
                        ExecStart=/home/ubuntu/project folder where venv is /venv/bin/python /home/ubunto/project folder where venv is /venv/bin/daphne -e ssl:8001:privateKey=/etc/letsencrypt/live/your domain name.whatever/privkey.pem:certKey=/etc/letsencrypt/live/your domain name.whatever/fullchain.pem your_project_name.asgi:application  
                        Restart=on-failure
                        
                        [Install]
                        WantedBy=multi-user.target


**Reload everything**

 sudo systemctl daemon-reload

 sudo systemctl restart redis.service

 sudo systemctl start gunicorn.socket

 sudo systemctl enable gunicorn.socket

 sudo systemctl restart gunicorn

 sudo systemctl start daphne.service
 
 sudo service daphne restart

 sudo systemctl restart nginx


**Check Status is everything working**
 
 systemctl status daphne.service
 
 systemctl status on_boot.service

 sudo systemctl status redis
 
 sudo systemctl status certbot.timer

 sudo nginx -t

 sudo systemctl status gunicorn

 sudo systemctl status daphne.service


**11) Changes in AWS instance**

**Add Security Group Rule for Port 443:**

            Go to the AWS Management Console.
            
            Navigate to the EC2 dashboard.
            
            Select the EC2 instance associated with your web server.
            
            In the "Description" tab, scroll down to the "Security groups" section and click on the linked security group (e.g., 'launch-wizard-1').
            
            In the security group details, click the "Inbound rules" tab.
            
            Click the "Edit inbound rules" button.
            
            Click the "Add rule" button.
            
            Set the following values:
            
            Type: HTTPS (Port 443)
            
            Source: 0.0.0.0/0 (or limit to specific IP ranges if needed)
            
            Click the "Save rules" button to add the rule allowing incoming traffic on port 443.

 **Update Nginx**

  Navigate to /etc/nginx/

  sudo nano nginx.conf

  #after the line "ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE", paste the follwoing
  
              ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
              
              ssl_prefer_server_ciphers off;

  save and exit

  Navigate to etc/nginx/sites-available

  sudo nano nginx_name

  after the line "ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot", paste the follwoing

                  ssl_session_cache shared:SSL:10m;
                  
                  ssl_stapling on;

 save and exit

 wait for 5 minutes and check whether the site is working

  




  
             

 

 









                  



