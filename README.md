# Initialize Ubuntu distro in WSL2 without Microsoft Store

The following guide might help you to initialize Ubuntu distributions due to different purposes (e.g. Go or .NET Core development, running Docker containers, etc.) in WSL2 without using Microsoft Store where Ubuntu and other distributions are available. I originally made it to help me remember the settings I use. Later maybe I will automatize with a PowerShell script.

The guide assumes, that you are running Windows 10 which is updated to [version 2004](https://docs.microsoft.com/en-us/windows/whats-new/whats-new-windows-10-version-2004) (build 19041 or higher) and you have [WSL2 already installed](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

## Download the latest Ubuntu version

Before everything you have to download the optimised and certified Ubuntu server images from the [Ubuntu Cloud](https://ubuntu.com/download/cloud) portal's [image repository](http://cloud-images.ubuntu.com). Please find the lastest versions below:

* Ubuntu Server 20.04 LTS (amd64):\
  <http://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64-wsl.rootfs.tar.gz>
* Ubuntu Server 19.10 (amd64):\
  <http://cloud-images.ubuntu.com/eoan/current/eoan-server-cloudimg-amd64-wsl.rootfs.tar.gz>
* Ubuntu Server 18.04 LTS (amd64):\
  <http://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64-wsl.rootfs.tar.gz>

## Deploy a disto in WSL2

According to the [WSL Documentation](https://docs.microsoft.com/en-us/windows/wsl/reference), the best way to interact with the Windows Subsystem for Linux is to use the `wsl.exe` command.

To list the existing distributions, issue the `wsl -l -v` command. This will help to avoid to try deploying a new distribution with the name of an existing one.

```console
  NAME                 STATE           VERSION
* distribution1        Running         2
  distribution2        Stopped         2
```

Deploying a distribution can be done with the `wsl --import <distro> <deployment_folder> <image>` command. Let's assume, that we would like to use the new distribution for developing .NET Core application with Ubuntu 20.04 LTS, therefore select `ubuntu-20.04-dotnetcore` as the name of the instance.

```powershell
wsl --import ubuntu-20.04-dotnetcore E:\WSL\ubuntu-20.04-dotnetcore E:\Downloads\ubuntu-20.04-server-cloudimg-amd64-wsl.rootfs.tar.gz
```

After the successful deployment, we can see the new distribution in the list.

```console
  NAME                           STATE           VERSION
* distribution1                  Running         2
  distribution2                  Stopped         2
  ubuntu-20.04-dotnetcore        Stopped         2
```

Since the new distribution is not the default one, it can be used with the `wsl -d ubuntu-20.04-dotnetcore` command. If you would like to have it as default, it can be easily done by issuing the `wsl --set-default ubuntu-20.04-dotnetcore` command.

## Adding a user with sudoers persmission (optional)

By default there is no other users in the case of the new distribution just the `root`. Due to security reasons I recommend to create a normal user which has sudoers permission. Run the new distribution on the Windows host:

```powershell
wsl -d ubuntu-20.04-dotnetcore
```

First, you have to create a user. In the guide let's call it as `normaluser`, and please issue the following command.

```bash
adduser normaluser
```

Please follow the instructions.

```console
Adding user `normaluser' ...
Adding new group `normaluser' (1000) ...
Adding new user `normaluser' (1000) with group `normaluser' ...
Creating home directory `/home/normaluser' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for normaluser
Enter the new value, or press ENTER for the default
        Full Name []: Normal User
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
```

Give the sudoers privileges to the `normaluser`:

```bash
adduser normaluser sudo
```

Set `normaluser` as the default one via the `/etc/wsl.conf`:

```bash
tee /etc/wsl.conf <<BLOCK
[user]
default=normaluser
BLOCK
```

The change will be effective only in a new session, so let's exit from the current one with the `logout` command, and shut down the distribution in the WSL.

```powershell
wsl --shutdown ubuntu-20.04-dotnetcore
```

When you run the new distribution again with the `wsl -d ubuntu-20.04-dotnetcore` command the `normaluser` will be used by default.

## Install Powerline-Go as a prompt (optional)

[Powerline](https://github.com/powerline/powerline) is a statusline plugin for [vim](https://www.vim.org/). Based on its idea [Powerline-Go](https://github.com/justjanne/powerline-go) has been created to have a beautiful [Bash](https://www.gnu.org/software/bash/) prompt.

To install it you have to issue the following commands within the new distribution (after issuing `wsl -d ubuntu-20.04-dotnetcore`):

```bash
sudo su
cd /usr/local/bin
wget https://github.com/justjanne/powerline-go/releases/download/v1.17.0/powerline-go-linux-amd64
chmod +x powerline-go-linux-amd64
tee -a /etc/bash.bashrc > /dev/null <<BLOCK
# Enable Powerline-Go
function _update_ps1() {
        PS1="$(/usr/local/bin/powerline-go-linux-amd64 -error $?)"
}
if [ "$TERM" != "linux" ] && [ -f "/usr/local/bin/powerline-go-linux-amd64" ]; then
        PROMPT_COMMAND="_update_ps1; $PROMPT_COMMAND"
fi
BLOCK
```

Remark: Powerline-Go requires [fonts](https://github.com/powerline/fonts) which contain some special characters which are related for example to Git. In case of [Windows Terminal](https://github.com/microsoft/terminal) I recommend to use [Cascadia Code](https://github.com/microsoft/cascadia-code/releases), which can be set in `settings.json` file with adding `"fontFace": "Cascadia Code PL"` to the actual terminal profile.

## Basic Git configuration (optional)

In case of using the new distribution with Git, then some basic configuration should be used within the new distribution (after issuing `wsl -d ubuntu-20.04-dotnetcore`):

```bash
git config --global user.name "Your Name"
git config --global user.email "yourname@email.com"
git config --global alias.hist "log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"
git config credential.helper store
```
