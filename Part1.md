# Setting Up a Secure Debian 12 Server with Nginx

This tutorial will assume that the user has already created their own Debian 12 server using Digital Ocean.

## Before We Begin

Currently, you should be logged in to your Debian 12 server in your terminal. However, the root user should be your current user, which is not recommended. When logged in as the root user, every command can be run, which can lead to unintended actions. If you need higher privilege to run a commmand it is better to add `sudo` at the start of it.

**Note**: For more details on a command, you can use `man <your-command>`. For example: `man sudo` opens the manual page for `sudo`.

## Establishing a Secure Regular User with sudo Privileges

Instead of simply using the command `useradd <username>` where `<username>` is your preferred name for the user, we will need to add the options `-m`, which creates a new home directory for the user and `-s`, which defines the program run at login; in our case, the program will be bash.

1. Enter the command: `useradd -ms /bin/bash <username>` 
2. Next, we'll give them a password using `passwd <username>`. Follow the prompts and now your user should have a password: your main line of defense.

Now we will use the command `usermod -aG sudo <username>`. The options `-aG` will append the user to the specified group: `sudo`. When a user is added to the sudo group, they are granted the ability to use the command `sudo`.

3. Enter: `usermod -aG sudo <username>`

Then, we have to change to the user using `sudo su -l <username>`

4. Enter: `sudo su -l <username>`

We will copy the .ssh directory and any files inside it, which includes our key pair, into our new user's home directory.

5. Enter: `sudo cp -r /root/.ssh /home/<username>`

Since we have just copied the files, they are still owned by the root user. To fix this, use `sudo chown -R <username>:<username> /home/<username>/.ssh`.

6. Enter: `sudo chown -R <username>:<username> /home/<username>/.ssh`

Now that we have set up our regular user, we should test out if we can connect to your Debian 12 server using `ssh -i .ssh/do-key <username>@<ip-address>`. Your ip address for the server can be found on the your Digital Ocean Droplet page.

7. In the host terminal, enter the command: `ssh -i .ssh/do-key <username>@<ip-address>`

You should now be successfully logged in with your new user! 

## Restricting Root Login

We will now restrict the root user from being able to login to the server using ssh. To accomplish this, we will change a line of code in the sshd_config file in the ssh directory. This ensures that no one can remotely login to your server with full privilege access, which is a huge security risk. 

1. In your home directory, enter the command `cd /etc/ssh`
2. Enter the command: `sudo vim sshd_config` to open the file.
3. In the file, press the "i" key on your keyboard to go into "insert mode", so we can edit it.
4. Look for `PermitRootLogin yes` and change it to `PermitRootLogin no`.
5. Press the "esc" key to go into "normal mode" and enter: `:wq` to save and exit the file.
6. Enter: `sudo systemctl restart ssh.service`

If you test the configuration by attempting to login as the root user via SSH — access should now be denied.

## Installing and Configuring Nginx





