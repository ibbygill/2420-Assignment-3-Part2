# 2420-Assignment-3
### Assignment 3 Linux

#### Click on the link to visit the working server [164.92.88.167](http://164.92.88.167/)

If you also want to also setup a new server, this is a quick guide to help you do that.


## Setting Up a New Server

#### Intial Step
We will be creating a user, using the following command:

``` linux
sudo useradd --system -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

This will simply create a system user for you called `webgen`

#### Create Directories and Changing Ownership
Now we'll make directories for the necessary files inside of webgens home directory
``` linux
sudo mkdir /var/lib/webgen/HTML
sudo mkdir /var/lib/webgen/bin
```
We will run the following command
``` linux
sudo chown -R webgen:webgen /var/lib/webgen
```

This ensures that the ownership is changed to our new user `webgen` for all directories we've created and other directories inside of `/var/lib/webgen`

`webgen` user tree should look like the following:

``` linux
/var/lib/webgen/

├── bin/
│   └── generate_index   # copy the generate_index into bin
└── HTML/
    └── index.html       # copy the index.html into HTML folder
```

#### Move .service and .timer Files
Now you will take the .service and .timer files and move them to the following directory using the command below.
These .service and .timer files have to be moved to the /system because all .service and .timer are run there.

``` linux
mv generate-index.service /etc/systemd/system/
mv generate-index.timer /etc/systemd/system/
```
Once these commands are run, you will need to enable the .service and .timer files. 

```
sudo systemctl enable generate-index.service
sudo systemctl enable generate-index.timer
```

#### Nginx Install and Conf File
Next, we'll ensure that you have nginx installed on your machine by running the command below.

``` linux 
sudo pacman -S nginx
```
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