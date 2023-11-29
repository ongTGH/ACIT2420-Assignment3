# Tutorial

This tutorial will assume that the user has already created their own debian server using Digital Ocean.

## Initial Setup

We will enable connecting to our droplet using Secure Shell (SSH).

First, we will generate a key pair:
1. Create a new .ssh directory using `mkdir .ssh`
2. Then input `ssh-keygen -t ed25519 -f .ssh/do-key -C <your-email-address>`

Second, 
1. In your web browser, log in to your Digital Ocean account
2. In the settings page for your Debian 12 server

Currently, you should be logged in to your Debian 12 server in your terminal. However, the root user should be your current user, which is not recommended. When logged in as the root user, every command can be run, which can lead to unintended actions. If you need higher privilege to run a commmand it is better to add `sudo` at the start of our command.

## First, we will create a new regular user and configure it so it can use `sudo`.

Instead of simply using the command `useradd <username>` where `<username>` is your preferred name for the user, we will need to add the options `-m`, which creates a new home directory for the user and `-s`, which defines the program run at login; in our case, the program will be bash.
1. Enter the command: `usermod -ms /bin/bash <username>` 
2. Next, we'll give them a password using `passwd <username>`. Follow the prompts and now your user should have a password: your main line of defense.

Now we will use the command `usermod -aG sudo <username>`. The options `-aG` will append the user to the specified group: `sudo`. When a user is added to the sudo group, they are granted the ability to use the command `sudo`.
1. Enter the command: `usermod -aG sudo <username>`

Then, we have to 
1. Change to the user using `sudo su -l <username>`

We will copy the .ssh directory and any files inside it, including our key pair, into our new user's home directory.
1. Enter the command: `sudo cp -r /root/.ssh /home/<username>`

Since we have just copied the files, they are still owned by the root user. To fix this, use `sudo chown -R <username>:<username> /home/<username>/.ssh`.
1. Enter the command: `sudo chown -R <username>:<username> /home/<username>/.ssh`

Now that we have set up our regular user, we should test out if we can connect to your Debian 12 server using `ssh -i .ssh/do-key <username>@<ip-address>`. Your ip address for the server can be found on the your Digital Ocean Droplet page.
1. In the host terminal, enter the command: `ssh -i .ssh/do-key <username>@<ip-address>`

You should now be successfully logged in with your new user! 

We will now restrict the root user from being able to login to the server using ssh. To accomplish this, we will change a line of code in the sshd_config file in the ssh directory. This ensures that no one can remotely log in to your server with full privilege accesss, which is a huge security risk. 
1. In your home directory, enter the command `cd /etc/ssh`
2. Enter the command: `sudo vim sshd_config` to open the file.
3. In the file press the `i` key on your keyboard to go into "insert mode", so we can edit the file.
4. Look for `PermitRootLogin yes` and change it to `PermitRootLogin no`.
5. Press the `esc` key to go into "normal mode" and enter: `:wq` to save and exit the file.



