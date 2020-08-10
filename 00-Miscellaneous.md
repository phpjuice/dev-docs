- [DEV Cheat Sheet](#dev-cheat-sheet)
  - [DB](#db)
  - [SSH File transfer](#ssh-file-transfer)
  - [Oh My zsh Installation](#oh-my-zsh-installation)
    - [2. _Optionally_, backup your existing `~/.zshrc` file:](#2-optionally-backup-your-existing-zshrc-file)
    - [3. Create a new zsh configuration file](#3-create-a-new-zsh-configuration-file)
    - [4. Change your default shell](#4-change-your-default-shell)
  - [Install Plugins](#install-plugins)
    - [Zsh Syntax Highlighting](#zsh-syntax-highlighting)
    - [Zsh Auto Suggestions](#zsh-auto-suggestions)

# DEV Cheat Sheet

## DB

- Create a new database:

  ```bash
  CREATE DATABASE dbname DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
  ```

- Create a new user :

  ```bash
  GRANT ALL PRIVILEGES ON dbname.* TO 'username'@'%' IDENTIFIED BY 'password';
  ```

- Change Password :

  ```bash
  mysqladmin -u user password 'newpassord'
  ```

## SSH File transfer

- local > server :

  ```bash
  scp file root@vps188221.vps.ovh.ca:.
  ```

- server > local :

  ```bash
  scp root@vps188221.vps.ovh.ca:file /local
  ```

## Oh My zsh Installation

```shell
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```

### 2. _Optionally_, backup your existing `~/.zshrc` file:

```bash
cp ~/.zshrc ~/.zshrc.orig
```

### 3. Create a new zsh configuration file

You can create a new zsh config file by copying the template that we have included for you.

```bash
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

### 4. Change your default shell

```bash
chsh -s /bin/zsh
```

## Install Plugins

### Zsh Syntax Highlighting

Clone this repository in oh-my-zsh's plugins directory:

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Activate the plugin in `~/.zshrc`:

```
plugins=( [plugins...] zsh-syntax-highlighting)
```

### Zsh Auto Suggestions

Clone this repository in oh-my-zsh's plugins directory:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

Activate the plugin in `~/.zshrc`:

```
plugins=( [plugins...] zsh-autosuggestions)
```

```bash
apt-cache search php7.3 | grep "php7.3" | awk '{print $1}'
```
