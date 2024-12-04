# 2420-Assignment-3
### Assignment 3 Linux

##### Click on the link to visit the working load balancer [HERE](http://209.38.5.95/)

##### If you want to see the documents click on this link [HERE](http://209.38.5.95/documents) 

## Purpose

The purpose of this README is to document the process of setting up and configuring a web server using Nginx, and a load balancer. It provides clear instructions for creating a system user, organizing file structures, and automating services with timers. This README also explains why specific choices were made, such as the use of a system user and the purpose of a load balancer.


## Create a Project on DigitalOcean


## Create Droplet
Select the following settings for both droplets:

    Image: Arch Linux
    Region: SFO3
    Plan: Basic, 1 vCPU, 1GB RAM (minimum required)
    Add a tag: web

Repeat this process for the second droplet.

We will also add these droplets into our localmachines `.ssh` folder.

```
Host drop1
  HostName 143.198.111.224
  User arch
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/do-key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null

Host drop2
  HostName 143.198.109.183
  User arch
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/do-key
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```


## Create a Load Balancer in DigitalOcean
    Go to Networking > Load Balancers and click "Create Load Balancer".

    Set the following options:
        Region: SFO3 (same as your droplets).
        Forwarding Rule:
            Load Balancer: HTTP, Port 80.
            Droplet: HTTP, Port 80.

        Add Droplets:
            Use the web tag to automatically include all droplets with the web tag.

Click "Create Load Balancer" to finalize the setup.

> [!Note]
> You may get an error, on your load-balancer this is due to nginx not yet running on your droplets.

## Initial Step

In each of the droplets, we will be doing the following commands.

First we'll sync, refresh and update all packages.

``` linux
sudo pacman -Syu
```

Then we can install the required packages.

``` linux
sudo pacman -S neovim nginx git ufw
```

We will be creating a user, using the following command in both of the droplets:

``` linux
sudo useradd --system -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

This will simply create a system user for you called `webgen`

System users are created by the system during installation and are used to run system services and applications. When creating the --system user we set the /usr/sbin/nologin preventing direct logins to the account. It limits the privileges and also access to sensitive parts of the system

## Create Directories and Changing Ownership

Now we'll make directories for the necessary files inside of webgens home directory
``` linux
sudo mkdir /var/lib/webgen/HTML
sudo mkdir /var/lib/webgen/bin
```
We will run the following command
``` linux
sudo chown -R webgen:webgen /var/lib/webgen
```

-R ensures that the change applies recursively to all files and subdirectories

webgen:webgen is for OWNER:GROUP


Now you will ensure that the ownership is changed to our new user `webgen` for all directories we've created and other directories inside of `/var/lib/webgen`

If you want to clone this repo, and move the appropriate directories and files into the correct spots. That is also an option using the git clone method.

Then  using `mv` commands to move the files into their correct spots.

Your `webgen` user tree should look like the following:

``` linux
/var/lib/webgen/
├── bin/
│   └── generate_index   # Copy the generate_index script here
├── HTML/
│   └── index.html       # Place the main index.html here
└── documents/
    ├── file-one         # Example file for directory listing
    └── file-two         # copy the index.html into HTML folder
```

Ensure that the generate_index script is given executable permissions, as we will be using the .service file to run the generate_index script inside of /bin

``` linux
chmod +x /var/lib/webgen/bin/generate_index
```

#### Move .service and .timer Files
Now you will take the .service and .timer files and move them to the following directory using the command below.
These .service and .timer files have to be moved to the /system because all .service and .timer are run there.

``` linux
mv generate-index.service /etc/systemd/system/
mv generate-index.timer /etc/systemd/system/
```
Once these commands are run, you will need to enable the .service and .timer files. 

``` linux
sudo systemctl start generate-index.service
sudo systemctl enable generate-index.service
sudo systemctl enable --now generate-index.timer
```

Now after we have started our .service and .timer we can check if the .timer file and .service file are active, by running 

``` linux
sudo systemctl status generate-index.timer
sudo systemctl status generate-index.service
```

#### Nginx Install and Conf File
Next, we'll ensure that you have nginx installed on your machine by running the command below.

Now we can move into the .conf file to set webgen as the user and create the server block configuration as included.

Go into your nginx.conf file and update the `user` so it says `user webgen;`, it is usually commented out in the .conf file.

Run the next command, this is where you will be pasting the server.conf file. Ensure that you change the server_name and check that the root is pointing to the correct HTML path so it can find the index.html to show on screen. The location {  } is used so that it checks

``` linux 
sudo nvim /etc/nginx/sites-available/webgen
```

After pasting the server.conf file and changing the server_name to your own, run the command below.

``` linux
sudo ln -s /etc/nginx/sites-available/webgen /etc/nginx/sites-enabled/webgen
sudo nginx -t
sudo systemctl restart nginx
```

This will enable the server block, and it should be up and running now.

The next steps will install UFW which is a friendly way to create a firewall for your IPv4 or IPv6 host.

``` linux
sudo pacman -S ufw
```

Next we will ensure that you allow ssh and http, this is so we can access the server using ssh and http so that the files can be hosted on the site.

``` linux
sudo ufw allow http
sudo ufw allow ssh
```

We can also limit the ssh to deter brute force attacks by running

``` linux
sudo ufw limit ssh
```

Now run the following to enable the firewall
``` linux
sudo ufw enable
```

You can verify the status by running
``` linux
sudo ufw status verbose
```

#### Troubleshooting

If you ran into any trouble, run the following command to troubleshoot tips to check logs

``` linux 
sudo systemctl status <program>
sudo journalctl -u <program>
```
This will give you a list of logs for any program.