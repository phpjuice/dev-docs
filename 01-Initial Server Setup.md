# Initial Server Setup with Ubuntu 18.04

- [Initial Server Setup with Ubuntu 18.04](#initial-server-setup-with-ubuntu-1804)
  - [Create a Deployment User](#create-a-deployment-user)
    - [Step 1 : Logging in as Root](#step-1--logging-in-as-root)
    - [Step 2 : Creating a New User](#step-2--creating-a-new-user)
    - [Step 3 : Granting Administrative Privileges](#step-3--granting-administrative-privileges)
    - [Step 4 : Setting Up a Basic Firewall](#step-4--setting-up-a-basic-firewall)
    - [Step 5 : Enabling External Access for Your Regular User](#step-5--enabling-external-access-for-your-regular-user)
    - [Step 6 : Connect as Deployer user](#step-6--connect-as-deployer-user)
    - [Step 7 : Disable Password Authentication (Recommended)](#step-7--disable-password-authentication-recommended)
    - [Step 8 : Generate SSH key pair for deployer user](#step-8--generate-ssh-key-pair-for-deployer-user)

## Create a Deployment User

### Step 1 : Logging in as Root

If you are not already connected to your server, go ahead and log in as the root user using the following command (substitute the highlighted portion of the command with your server’s public IP address):

```bash
ssh root@server_ip
```

_The root user is the administrative user in a Linux environment that has very broad privileges. Because of the heightened privileges of the root account, you are discouraged from using it on a regular basis. This is because part of the power inherent with the root account is the ability to make very destructive changes, even by accident._

### Step 2 : Creating a New User

Once you are logged in as root, we’re prepared to add the new user account that we will use to log in from now on.

```bash
adduser deployer
```

You will be asked a few questions, starting with the account password.

_Note: we used deployer as username here just as a convention._

### Step 3 : Granting Administrative Privileges

Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as “superuser” or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word sudo before each command.

As root, run this command to add your new user to the sudo group (substitute the highlighted word with your new user):

```bash
usermod -aG sudo deployer
```

Now, when logged in as your regular user, you can type sudo before commands to perform actions with superuser privileges.

Laravel needs some writable directories to store cached files and uploads, so the directories created by the deployer user must be writable by the Nginx web server. Add the user to the www-data group to do this:

```bash
usermod -aG www-data deployer
```

The default permission for files created by the deployer user should be 644 for files and 755 for directories. This way, the deployer user will be able to read and write the files, while the group and other users will be able to read them.

Do this by setting deployer’s default umask to 022:

```bash
chfn -o umask=022 deployer
```

### Step 4 : Setting Up a Basic Firewall

Ubuntu 18.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall very easily using this application.

```bash
ufw app list
```

We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:

```bash
ufw allow OpenSSH
```

### Step 5 : Enabling External Access for Your Regular User

Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

If you logged in to your root account using SSH keys, then password authentication is disabled for SSH. You will need to add a copy of your local public key to the new user’s `~/.ssh/authorized_keys` file to log in successfully.

```bash
su - deployer
```

On your local machine run the following command, then copy the command’s output which contains your public key:

```bash
cat ~/.ssh/id_rsa.pub
```

On your server as the deployer user run the following:

```bash
nano ~/.ssh/authorized_keys
```

Paste the public key to the editor and hit CTRL-X, Y, then ENTER to save and exit.

Restrict the permissions of the file:

```bash
chmod 600 ~/.ssh/authorized_keys
```

Finally, exit the server:

```bash
exit
```

### Step 6 : Connect as Deployer user

On your local machine run the following command

```bash
ssh deployer@server_ip
```

### Step 7 : Disable Password Authentication (Recommended)

Now that your new user can use SSH keys to log in, you can increase your server’s security by disabling password-only authentication. Doing so will restrict SSH access to your server to public key authentication only. That is, the only way to log in to your server (aside from the console) is to possess the private key that pairs with the public key that was installed.

_Note: Only disable password authentication if you installed a public key to your user as recommended in the previous section, step four. Otherwise, you will lock yourself out of your server!_

open the SSH daemon configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Find the line that specifies PasswordAuthentication, uncomment it by deleting the preceding #, then change its value to “no”. It should look like this after you have made the change:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Type this to reload the SSH daemon:

```bash
sudo systemctl reload sshd
```

### Step 8 : Generate SSH key pair for deployer user

in order to deploy from Gitlab and read repositories we need to setup first deploy keys,
so let's generate an SSH key pair as the deployer user. This time, you can accept the default filename of the SSH keys:

```bash
cat ~/.ssh/id_rsa.pub
```

then copy the following command’s output which contains deployer user public key:

```bash
cat ~/.ssh/id_rsa.pub
```

then add it to your git server

- [Add SSH keys to GitHub](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)
- [Add SSH keys to GitLab](https://docs.gitlab.com/ee/gitlab-basics/create-your-ssh-keys.html)
- [Add SSH keys to Bitbucket](https://confluence.atlassian.com/bitbucket/set-up-an-ssh-key-728138079.html)

Now you will be able to connect to your Git server with your local machine. Test the connection with the following command:

```bash
ssh -T git@mygitserver.com
```
