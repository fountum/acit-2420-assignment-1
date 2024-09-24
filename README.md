git@github.com:fountum/acit-2420-assignment-1.git# acit-2420-assignment-1

Assignment 1 for ACIT 2420

  
- [Introduction](#Introduction)
- [Installing and Configuring doctl](#installing-and-configuring-doctl)
- [Creating SSH Key Pair](#Creating-SSH-Key-Pair)
- [Adding a Public Key to your DigitalOcean Account](#adding-a-public-key-to-your-digitalocean-account)
- [Adding an Arch Linux Image to DigitalOcean](#adding-an-arch-linux-image-to-digitalocean)
- [Creating a cloud-init YAML configuration](#creating-a-cloud-int-yaml-configuration)
- [Creating a Droplet](#Creating-a-Droplet)
- [Connecting to Droplet](#connecting-to-droplet)
- [Verifying Configuration](#verifying-configuration)
- [External Resources](#External-Resources)
- [Citations](#Citations)


# Introduction

This tutorial will teach you how to create Arch Linux based virtual machines using DigitalOcean with the `doctl` command line interface. No knowledge of DigitalOcean, `doctl`, cloud-init, or Linux systems is needed to understand this guide. Most of the steps in this guide will be done using a terminal. 

# Installing and Configuring doctl

`doctl` is a command line interface that allows you to manage your DigitalOcean resources using a terminal. It's capabale of most of the fucntionality available on the DigitalOcean web UI. (Citation?)

## Downloading `doctl`

Depending on your operating system (OS), there are several ways to download `doctl`. This guide will use commands that work Arch Linux. If you are using a different OS, check [DigitalOcean's installation documentation](https://docs.digitalocean.com/reference/doctl/how-to/install/#step-1-install-doctl).

`doctl` is available on the Arch Linux package repository, so the `pacman` utility can be used to install it:
`sudo pacman -S doctl`

Command explanation:
- `sudo` temporary gives a non-root user root privleges, or administrative permissions. Because we are installing packages, we must use `sudo` before `pacman`.  
  - `sudo` is used instead of staying logged in as the root user to prevent damaging the system. Logging in as the root user increases the chances of a misused command modifying critical files and damage the system (citation?)
- `pacman` is the package manager on Arch Linux. A package manager is a program that can install, update, and remove software packages. (https://learning.oreilly.com/library/view/linux-for-system/9781803247946/B18575_08.xhtml#_idParaDest-110) 
  - The flag `-S` syncronizes the packages installed on the machine with the package repository. In other words, it installs and updates software.

After using this command, the console will show
- the package and dependencies to be installed
- ????? Figure out what other input goes here

Ensure that you are installing the correct packages and confirm the download (type and enter `y`)
INSERT SCREENSHOT OF COMMAND + OUTPUT

To confirm `doctl` had been properly installed, use (`man doctl`)

## Generating a DigitalOcean API token

To allow `doctl` to access your DigitalOcean account, an API token must be generated. API tokens are string that authenticates programs to access the API of another program. (citation?)

1. Log in DigitalOcean and navigate to the [Application and API page](https://cloud.digitalocean.com/account/api/tokens)
2. Click *Generate New Token*
3. (SCREEN SHOT OF PAGE) Type in a name for the key. Then click *Full Access* under Scopes

NOTE: 
- Enabling Full Access with this key will allow programs with the APK token to perform create, read, update, and delete (CRUD) opertions. To restrict what is actions are possible, select a custom scope below. *Token scopes cannot be modifed after creation*(picture here, maybe explanation?)
- As a security measure, the token will be deleted after 90 days by default. This can be changed to less or more time, but it's recommended to have tokens expire for security purposes (citation?)

4. Scroll to the bottom and click *Generate Token*
(I assume it's done here but maybe more?)

5. Copy and save the token in a file.
*CAUTION: the token will only be shown to you once.*

## Adding your API token to doctl

In the terminal, enter the following commands to attach an API token to doctl:
`doctl auth init --context <CONTEXT-NAME>`
- `--context` means "authentication context", or essential a different DigitalOcean account.  (https://docs.digitalocean.com/reference/doctl/reference/auth/)
You will be prompeted to enter in your API token. After that, a list of all contexts will be shown. (maybe wrong)

To switch contexts, use the command:
`doctl auth switch --context <CONTEXT-NAME>`
(What does this show + screenshots)

To verify that you correctly added the token, get your account details using the command:
`doctl account get`
Your terminal output should looks similar to this:
(screen shot)

### External References
https://docs.digitalocean.com/reference/doctl/how-to/install/


# Creating an SSH Key Pair

## What is Secure Shell (SSH)?
Secure Shell (SSH) is a protocol that allows for data to sent securely over unsecure networks. SSH is used to remotely connect to servers and issue commands. It uses public key cryptography to encrypt and decrypt data, as well as authenitcate users. At a high level, the protocol works like this in the context of remote servers:
- An SSH key pair is generated, resulting in one *public key* and *private key*
  - Public keys are used to encrypt data; this is given to the server
  - Private keys are used to decrypt data; this is kept by the user/client
  - The keys themselves are put in a cryptography algorithm with data to produces what looks like random data but is actually a very specific output from the key + data  
  - Only the corrisponding key pairs work with each other
- When establishing a connection to the serve, the client provides the private key to authenticate themself
- When sending data, the server will use the public key to encrypt the data, which can only be decrypted using the private key
  - This prevents 'man-in-the-middle' attacks as attackers will not beable to view the contents of the data if they get a hold of it

(citations please)
https://www.cloudflare.com/learning/access-management/what-is-ssh/
https://www.cloudflare.com/learning/ssl/how-does-public-key-encryption-work/
https://www.cloudflare.com/learning/ssl/what-is-asymmetric-encryption/

To generate an SSH key pair, the utility `ssh-keygen` can be using in the terminal on a majority of Windows/MacOs/Linux systems.

Usage:
`ssh-keygen -t <ENCRYPTION> -f <PATH> -C <COMMENT>`
Command Explanation:
- `-t` Type of encryption used to generate the key. We will use `ed25519` in this example.
- `-f` Path where the key is created and name of key. This should be the `.ssh` directory in your home directory.
- `-C` An optional comment that's appended on to the end of the public key

Example Arch Linux usage:
`ssh-keygen -t ed25519 -f ~/.ssh/do-key -C 'ACIT-2420-Assignement1'`


Check folder for keys, add screenshots

# Adding a Public Key to your DigitalOcean Account 

1. Copy the contents of the public key.
2. Type the command `doctl compute ssh-key create <KEY-NAME> --public-key <PUB-KEY>` and hit enter.
- <KEY-NAME> The name you want to give the key, such as 'ACIT-2420'
- <PUB-KEY> The contents of the public key file.
3. Type the command `doctl compute ssh-key list` to list all of the SSH keys on this account. 
4. Verify that your key was added correctly



https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/

# Adding an Arch Linux Image to DigitalOcean
`doctl compute image create <image-name> --image-url <url> --region <region>`
`<image-name>` custom image's name on DigitalOcean. You'll use this name to issue commands using this image
`<url>` distribution's download url
`<region>` what server the image is stored on. pick a region which is geographically the closest to you
- To see a list of regions, use `doctl compute region list`

insert example use here

# Creating a cloud-init YAML configuration

```
#cloud-config
users:
  - default

system_info:
  default_user:
    name: arch
    lock_passwd: true
    gecos: default arch user if you see this it works
    groups: wheel
    sudo: ["ALL=(ALL) NOPASSWD: ALL"]
    shell: /bin/bash
    ssh-authorized-keys:
      - <public-key>

packages:
  - neovim
  - less
  - bash-completion
  - tmux
  - git

disable_root: true
```

`#cloud-config` required at the start of file for cloud-init to recongize the file as cloud config data
    - citation : https://cloudinit.readthedocs.io/en/latest/explanation/format.html#headers-and-content-types
`users` defines users and their properties
- `default` creates the `default_user` defined in `system_info`
name: user's name
lock_passwd: prevents logging in as this user use a password. Can only be log in using SSH key.
gecos: optional comment for user
group: groups users is member of. `wheel` is an admin group
sudo: user's sudo privleges. 
- First `ALL` specifies what hosts commands can be ran on
- Second `(ALL)` specifies what users commands can be run as
- `NOPASSWD:` enables the use of sudo without a password
- Third `ALL` allows all commands to be used
- citaton?

- ref: https://wiki.archlinux.org/title/Users_and_groups#User_groups



https://wiki.archlinux.org/title/Users_and_groups
# Creating a Droplet
https://docs.digitalocean.com/reference/doctl/reference/compute/droplet/


`--size` specs of droplet 
insert a link for reference or use command `doctl compute size list`
`--region` region VM is hosted 
insert a link forereference or use `doctl compute region list`
`--user-data-file` path to YAML file for cloud-init. 
s-1vcpu-512mb-10gb 
`doctl compute droplet create <droplet-name> --image <image> --size <size> --region <region> --user-data-file <path> --ssh-keys <key>`


# Connecting to Droplet

get ip: `doctl compute droplet get <droplet-name> --format PublicIPv4`
`ssh -i ~/.ssh/<key-name> arch@<ip>`


# Verifying Configuration
`pacman -Q <package-name>`
`cat /etc/passwd`

# External Resources
(doctl Reference)[https://docs.digitalocean.com/reference/doctl/reference/]