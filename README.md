# Linux-Assignment-3 Part 2
---

# Introduction
- Welcome to the second part of my Linux Assignment #3 where I will be creating two new servers with the same
setup configuration as the part 1 of this assignment. The goal is to create a system that is both reliable and
be able to handle more traffic by using a load balancer to evely distribute requests between th two servers.

---

## Tasks:
- New Droplets and Load Balancing Server
- Clone the repository 
- Creating a system user and giving ownership
- Setting up new servers and configuring nginx
- Our Load balancer Website

---

# *Disclamer* 
- Everything "Tasks" I'm doing, I'm doing for both servers
- I also created 2 different branches for server 1 and server 2 for pushing any changes into github
- I went in more depth in the video at the bottom  explaining my commands and action for this part of the assignment 

---

# Task 1 Server & Load Balancer
We first have to create 2 new digital ocean droplets running Arch Linux with the tag "web" 

! This README wont show how to create a droplet in digital ocean 

- Server 1
<img width="651" alt="Screenshot 2024-12-01 at 10 47 00 AM" src="https://github.com/user-attachments/assets/08cf7cb3-0e1c-4807-bd94-4c32d6cac003">

- Server 2
<img width="639" alt="Screenshot 2024-12-01 at 10 47 07 AM" src="https://github.com/user-attachments/assets/a77ce738-6641-4b89-ad57-d00246b247fb">


After creating the new server with the tag "web" -> Create a load balancer server to handle both (Using the tag "web" to connect to both)


- Load balancer

<img width="601" alt="Screenshot 2024-12-01 at 10 47 25 AM" src="https://github.com/user-attachments/assets/220be73b-0388-408e-b3b1-ed708fb137f5">

By using the tag "web", the load balancer will be able to balance the traffic between server 1 and server 2

---



# Task 2 System User - File Structure

In order to get started, clone the following repository 
```
https://git.sr.ht/~nathan_climbs/2420-as3-p2-start
```

By cloning this, you will be given a updated generate-index HTMl script. This scirpt will now show the public IP address of the servers.

### Create System User 

Part 1 goes more in depth with explaining the comamnds we are using, so in here I'll only explain in slight details

- System User
  ```
  sudo useradd -r -d /var/lib/webgen
  ```
- User Ownership
  ```
  sudo chown -R webgen:webgen /var/lib/webgen 
  ```
### File Structure
We will be using the same file structuer as part 1 with the new addition of the updated generate_index and a new directory named **documents** with file-one and file-two inside of it

```
── bin/
│ └── generate_index
├── documents/
│ ├── file-one
│ └── file-two
└── HTML/
  └── index.html
```

Make the genereate_index executable and run it to create the new index.html script
```
sudo chmod 700 /var/lib/webgen/bin/generate_index
```

# Task 3 Service and Timer
We wont be changing anything inside the Service and Timer file, will be the same in part 1

### Service file
We will be able to access our Service file inside -> `/etc/systemd/system/generate-index.service`

```
[Unit]
Description=Index
After=network-online.target
# This will delay the service untill the network is up 

[Service]
User=webgen
Group=webgen
ExectStart=var/lib/webgen/bin/generate_index
WorkingDirectory=/var/lib/webgen

[Install]
WantedBy=multi-user.target
```

### Timer File
We will be able to access our Timer file inside -> `/etc/systemd/system/generate-index.timer`

```
[Unit] 
Description=Timer For Index Service

[Timer]
OnCalendar=*-*-* 05:00:00
Persistent=true

[Install] 
WantedBy=timers.target
```
Before moving on double again if everything is working
```
systemctl status generate-index.timer
```

# Task 4 Nginx Configuration and Service Block 
We wont be changing much of the configuration, only adding two lines in our service block file to be able to see our files inside the /documents/ directory in the web 
### Nginx Configuration
```
user webgen; <-- this line 
worker_process 1; 

#error_log  logs/error.log
#error_log  logs/error.log notice;
#error_log  logs/error.log info;

events {
 worker_connection 1024;
}

http {
  types_hash_max_size 4096;  <-- this line
  types_hash_bucket_zie 64;  <-- this line
  include      mime.types;
  include    /etc/nginx/sites-enabled/*; <-- this line
}
```

### Server Block File
This will be inside the `/etc/nginx/website/server_block`

```
server {
  listen 80
  listen [::]: 80;
  # This will tell nginx to listen for HTTP requests on port 80

  server_name webgen;

  root /var/lib/webgen/HTML;
  # when a request is received, Nginx will look for files to serve in /var/lib/webgen/HTML
  index index.html

  locaton /documents/ {
    root /var/lib/webgen; 
    autoindex on; # New line 
    try_files $uri $uri/ =404
    # if there was an error loading it in 
  }
}
```
- `autoindex on;` Will enable directory listing, when this directive is enabled, Nginx will display a list of files and directories in directory if there is no index file (documents)
  
Ref = https://www.keycdn.com/support/nginx-directory-index


Before moving one we want to double check the new stuff we added 
```
# Check our syntax
sudo nginx -t

# Reload our nginx
sudo systemctl reload nginx

# if not started 
sudo systemctl start nginx

# Check status
sudo systemctl status nginx
```

# Task 5 Load Balancer Website
Completing all the previous steps, we will now be access our load balancer site handling both servers.

Here are my IP Address for my Load Balancer and both servers
### Load Balancer 
<img width="624" alt="Screenshot 2024-12-01 at 11 18 55 AM" src="https://github.com/user-attachments/assets/60db27fc-c29c-4579-9c60-2398e325bc19">

### Server 1
<img width="645" alt="Screenshot 2024-12-01 at 11 19 08 AM" src="https://github.com/user-attachments/assets/a428da54-04b8-4e19-b031-d72372d1a6da">

### Server 2
<img width="636" alt="Screenshot 2024-12-01 at 11 19 18 AM" src="https://github.com/user-attachments/assets/b8f97588-6938-4341-9981-0ebd0c16bf51">

## Acessing our Load Balancer site
We now want to visit our load balancer site -> vist `http://<loadbalancerIpadd>`

### Server 1
<img width="820" alt="Screenshot 2024-12-01 at 11 21 36 AM" src="https://github.com/user-attachments/assets/6ea1009d-6e3a-4907-bbb0-c8f819eccb23">

### Server 2
<img width="815" alt="Screenshot 2024-12-01 at 11 21 56 AM" src="https://github.com/user-attachments/assets/e433de77-b10f-4792-948a-abaabffcb721">

---
---
# Video 
https://youtu.be/qOafv8baYzA

