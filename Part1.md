# Setting Up a Secure Debian 12 Server with Nginx

This tutorial will assume that the user has already created their own Debian 12 server using Digital Ocean.

## Before We Begin

Currently, you should be logged in to your Debian 12 server in your terminal. However, the root user should be your current user, which is not recommended. When logged in as the root user, every command can be run, which can lead to unintended actions. If you need higher privilege to run a commmand it is better to add `sudo` at the start of it.

**Note**: For more details on a command, you can use `man <your-command>`. For example: `man sudo` opens the manual page for `sudo`.

## Establishing a Secure Regular User with sudo Privileges

Instead of simply using the command `useradd <username>` where `<username>` is your preferred name for the user, we will need to add the options `-m`, which creates a new home directory for the user and `-s`, which defines the program run at login; in our case, the program will be bash.

1. Enter the command: `useradd -ms /bin/bash <username>` 
2. Next, we'll give them a password using `passwd <username>`. Follow the prompts and now your user should have a password.

Now we will use the command `usermod -aG sudo <username>`. The options `-aG` will append the user to the specified group: `sudo`. When a user is added to the sudo group, they are granted the ability to use the command `sudo`.

3. `usermod -aG sudo <username>`

Then, we have to switch to the new user.

4. `sudo su -l <username>`

We will copy the .ssh directory and any files inside it, which includes our key pair, into our new user's home directory.

5. `sudo cp -r /root/.ssh /home/<username>`

Since we have just merely copied the files, they are still owned by the root user. To fix this, we'll use `chown`.

6. `sudo chown -R <username>:<username> /home/<username>/.ssh`

Now that we have set up our regular user, we should test out if we can connect to your Debian 12 server. Your ip address for the server can be found on the your Digital Ocean Droplet page.

7. In the host terminal, enter the command: `ssh -i .ssh/do-key <username>@<ip-address>`

You should now be successfully logged in with your new user! 

## Restricting Root Login

We will now restrict the root user from being able to login to the server using ssh. To accomplish this, we will change a line of code in the sshd_config file in the ssh directory. This ensures that no one can remotely login to your server with full privilege access, which is a huge security risk. 

1. Go to the home directory using `cd`, then enter the command: `cd /etc/ssh`
2. Enter the command: `sudo vim sshd_config` to open the file.
3. In the file, press the "i" key on your keyboard to go into "insert mode", so we can edit it.
4. Look for `PermitRootLogin yes` and change it to `PermitRootLogin no`
5. Press the "esc" key to go into "normal mode" and enter: `:wq` to save and exit the file.
6. Enter: `sudo systemctl restart ssh.service`

If you test the configuration by attempting to login as the root user via SSH — access should now be denied.

## Installing and Configuring Nginx

Nginx is an open-source web server, widely used for serving web content. We are going to use it to set up a simple server that will display an HTML file.

### Installation

To install nginx, we will be using `apt`: a package manager pre-installed with Debian.

1. `sudo apt install nginx`

By default, nginx will be disabled, so we have to enable it.

2. `sudo systemctl enable nginx`

### Setting Up the HTML File

On the Debian 12 server we're working with, the default directory is `/var/www`, so we will put our HTML file there.

1. Change to the directory using `cd /var/www`
2. Create a new folder called "my-site" using `sudo mkdir my-site` and navigate to the new folder using `cd my-site`
3. Enter the command: `sudo vim index.html` to create and edit the file.
4. Copy this into the index.html file:

        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>2420</title>
            <style>
                body {
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    height: 100vh;
                    margin: 0;
                }
                h1 {
                    text-align: center;
                }
            </style>
        </head>
        <body>
            <h1>Hello, World</h1>
        </body>
        </html>

5. Press the "esc" key to go into "normal mode" and enter: `:wq` to save and exit the file.

### Creating the Nginx Configuration File

1. In the home directory, enter: `cd /etc/nginx/sites-available`
2. Enter the command: `sudo vim my-site.conf` to create and edit the file.
3. Copy this into the my-site.conf file:

        server {
	        listen 80 default_server;
	        listen [::]:80 default_server;
	
	        root /var/www/my-site/;
	
	        index index.html index.htm index.nginx-debian.html;
	
	        server_name _;
	
	        location / {
		      # First attempt to serve request as file, then
		      # as directory, then fall back to displaying a 404.
		      try_files $uri $uri/ =404;
	        }
        }

### Enabling the Site

1. In the home directory, enter: `cd /etc/nginx/sites-enabled`
    
Nginx won't direct to our my-site.conf until we first unlink it from the default.

2. `sudo unlink default`

And then create a symbolic link.

3. `sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled`

Finally, we check for any errors in our configuration.

4. `sudo nginx -t`

Now we should restart nginx so that our changes take effect.

5. `sudo systemctl restart nginx`

### Well done! 
Your Debian 12 server is now securely configured with a regular user, restricted root login, and setup with an nginx web server serving a sample HTML page.

To see your web server in action, enter: `curl <ip-address>`
