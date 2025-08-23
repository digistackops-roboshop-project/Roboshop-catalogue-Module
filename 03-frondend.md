When you open our Website ==> when you click on any product ===>  it show the information by connecting with Catalogue Micro service 

So HERE in Frontend we need to Redirect our Traffic from "Frontend" to "Catalogue" Micro service 

Create Nginx Reverse Proxy Configuration.
```
vim /etc/nginx/nginx.conf
```
Add the following content

```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

        location /images/ {
          expires 5s;
          root   /usr/share/nginx/html;
          try_files $uri /images/placeholder.jpg;
        }
        location /api/catalogue/ { proxy_pass http://<catalogue-IP or DNS>:8080/; }

        location /health {
          stub_status on;
          access_log off;
        }

    }
}
```

Restart Your Nginx
```
sudo systemctl restart nginx
```
<img width="679" height="387" alt="image" src="https://github.com/user-attachments/assets/f150944e-f943-4e56-982c-f2b3c88db9e5" />


# Test our Application Showing Product List or Not

Open our Roboshop website in Browser ===> it will show the List of Products

## Check in Browser Level
```
Right Click ===> Click on "Inspect"  ===> Click on "Network"
```
For Suppose if in Click on HAL Product
```
 You can see  HERE in "Network"  Tab under HAL ===> Header
```
<img width="1015" height="433" alt="image" src="https://github.com/user-attachments/assets/5ffe99b9-cfc2-40c6-b6f6-92aa3f319319" />

When you click on "HAL Product" what happen in Backend ?

It is Redirecting or It connecting to Catalogue Server and Try to Fetch the DATA

```
You can see  HERE in "Network"  Tab under HAL ===> Response
```
<img width="885" height="406" alt="image" src="https://github.com/user-attachments/assets/757d7916-58fd-4111-a37c-0f520f8b9170" />

For Suppose in Organization sometimes you got  an Issues while Deploy the Application that time you can Troubleshoot using this Way using Inspect ==> "Network" Tab

### What we Achieve from these Method ?

We find the Issues like
   1. Is it Redirecting from Frontend to Catalogue server or Not  {if not, Issue with Frontend Nginx Reverse Proxy Configuration}
   2. Catalogue Server is Fetching DATA or Not {If not, Issue with Catalogue and MongoDB connection}

## Check at server Level

### Check the Logs in  Frontend Nginx server
```
tail -f /var/log/nginx/access.log
```
 It will show the logs are any Issues in Nginx

 ### Check the Logs in  Backend catalogue server
```
tail -f /var/log/message
```
It will show the logs are any Issues in Catalogue  or any other Module you can check HERE
<img width="1013" height="102" alt="image" src="https://github.com/user-attachments/assets/8ddd6a79-c02c-4c99-af3e-d5ed29a6e0fb" />


Using this you can Find out the Issue with Frontend and Backend Catalogue and DB Connection
