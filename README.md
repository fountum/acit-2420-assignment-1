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

This tutorial will teach you how to create Arch Linux based virtual machines on DigitalOcean with the `doctl` command line interface. No knowledge of DigitalOcean, `doctl`, cloud-init, or Linux systems is needed to understand this guide. Most of the steps in this guide will be done using a terminal. 

# Installing and Configuring doctl

`doctl` is a command line interface that allows you to manage your DigitalOcean resources using a terminal. It's capabale of most of the fucntionality available on the DigitalOcean web UI. 

## Downloading and Installing `doctl`

Depending on your operating system (OS), there are several ways to download `doctl`. This guide will use commands that work in Arch Linux. If you are using a different OS, check [DigitalOcean's installation documentation](https://docs.digitalocean.com/reference/doctl/how-to/install/#step-1-install-doctl).

`doctl` is available on the Arch Linux package repository, so the `pacman` utility can be used to install it:
```sudo pacman -S doctl```

>[!NOTE]
>Before installing packages it is recommended to use the command `pacman -Syu`. This command will synchronizes (install) packages, synchronize the package database (which contains metadata about packages), and update all system packages. 

Command explanation:
- `sudo` temporarily gives a non-root user root privleges, or administrative permissions. Because we are installing packages, we must use `sudo` before `pacman`.  
  - However, this command only works if you have suffcient privleges.
  - `sudo` is used instead of staying logged in as the root user to prevent damaging the system. Logging in as the root user increases the chances of typos or bugs that can harm the system.
- `pacman` is the package manager on Arch Linux. A package manager is a program that can install, update, and remove software packages.  
  - `-S` tells `pacman` to syncronize packages on the machine with the package repository. In other words, this flag will install and update software.

After using this command, the console will show
- the package and dependencies to be installed
- download and installed size
- prompt to confirm download

![](./assets/pacman-doctl.png)

Confirm that you are installing the correct packages and confirm the download (type `y` and press enter). A successful installation gives the following output:
![](./assets/installed-doctl.png)

## Generating a DigitalOcean API token

To allow `doctl` to access your DigitalOcean account, an API token must be generated. API tokens are string that authenticates programs to access the API of another program.

1. Log in to your DigitalOcean control panel and navigate to the [Application and API page](https://cloud.digitalocean.com/account/api/tokens)
![](/assets/digitalocean-api.png)
2. Click *Generate New Token*
3. Type in a name for the token. Then click *Full Access* under Scopes.
![](/assets//digitalocean-token-creation.png)

>[!NOTE] 
> *Token scopes cannot be modified after creation*
> Enabling *Full Access* with this key will allow programs with the API token to perform create, read, update, and delete (CRUD) opertions on your DigitalOcean. To restrict what is actions are possible, select *Custom Scope* instead of *Full Access*. 
> As a security measure, the token will be deleted after 90 days by default. This can be changed to less or more time, but it's recommended to have tokens expire for security purposes.

4. Click *Generate Token* at the bottom of the page. 

5. Copy the token and save it in a text file. The token will only be shown to you once, so don't skip this step.

>[!WARNING]
> Do not share this token with anyone. Posessing and API token will give them access to your DigitalOcean droplets. 

![](/assets/digitalocean-api-token.png)

Once you've copied and saved your API token, you've completed this step.

## Adding Your API Token to doctl

Adding the API Token to `doctl` will allow you to use `doctl` to make changes to your droplets.

1. Enter this command into the terminal: 
```doctl auth init```
- This command used to give doctl permission access your DigitalOcean account within the scope of the API token 

After using the command, you will be prompeted to enter in your API token. 
![](/assets/doctl-auth-init.png)
2. Paste in your API token and press enter. `doctl` will begin validating your token.
![](/assets/doctl-validate-token.png)
3. Verify that you correctly added the token, get your account details using the following command:
```doctl account get```
- This command will try to get your account details.
![](/assets/doctl-account-get.png)

If the command successfully prints your account information, you've succesfully added you API token to `doctl`.

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

## Generating the Keys
To generate an SSH key pair, the utility `ssh-keygen` can be using in the terminal on Windows,MacOs, or Linux systems.

```ssh-keygen -t <ENCRYPTION> -f <PATH> -C <COMMENT>```
Command Explanation:
- `-t` Type of encryption used to generate the key. We will use `ed25519` in this example.
- `-f` Path where the key is created and name of key. This should be the `.ssh` directory in your home directory.
  - `~/.ssh` on Linux
  - `C:\Users\<USERNAME>\.ssh` on Windows
- `-C` An optional comment that's appended on to the end of the public key. Usually contact information of the key holder.

Example Arch Linux usage:
```ssh-keygen -t ed25519 -f ~/.ssh/do-key -C 'demo-key'```


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

DigitalOcean hosts a number of pre-configures images for Debain, Ubuntu, and CentOS but we will have to import Arch Linux manually.

1. Find the latest version of Arch Linux cloud image in the [Arch Linux Package Registry](https://gitlab.archlinux.org/archlinux/arch-boxes/-/packages/)
- Cloud images are pre-configured versions of operating systems intended for cloud infrastructure. They come with settings and software like Cloud-Init to make creating cloud instances more effecient.
- The file name will contain `cloudimg` and end with `.qcow2`
[screen shot]

2. Right click the file name and choose *copy link* in the context menu
3. In the terminal, type in the command: `doctl compute region list`. This will list the servers and where they are located.
(screenshot)
4. Choose a region that is closest geographically and remember its slug. In this guide we will use `SFO3` 
5. To import your image to DigitalOcean, use the following command:
`doctl compute image create <IMAGE-NAME> --image-url <URL> --region <REGION>`
- `<IMAGE-NAME>` The name you want to give this image on DigitalOcean. This is how you'll refer to this image in commands
- `<URL>` URL of the Arch Linux cloud image
- `<REGION>` What servers the image is saved to 
(example?)
(output?)
6. To verify that the custom image was added, type in the command `doctl compute image list`. This commands show all of images available
(screenshot)

# Creating a cloud-init YAML configuration

Cloud-init is a package create to quickly set up and configure systems, VMs, or cloud-based servers. Using a given configuration file, cloud-init will apply those settings without additional manual input. It comes pre-packaged with most cloud images of operating systems, such as the cloud image of Arch Linux we are using. In this step we will create a YAML file to pass to cloud-init when we create our Droplet.

There are many configuration options available using cloud-init. In this guide we will use cloud-init to:
- Add users
- Install packages
- Disable logging in as root via SSH

(explain YAML?)
(citation)

1. Create a file titled `config.yml`
2. Open the file in a text editor
3. Copy the following text and paste it into the file:

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

- `#cloud-config` *Header required at the start of file*. Cloud-init will not recongize the file as cloud config data if omitted
    - citation : https://cloudinit.readthedocs.io/en/latest/explanation/format.html#headers-and-content-types
- `users` defines users and their properties
  - `default` creates the `default_user` defined in `system_info`
- `name`: user's name
- `lock_passwd`: prevents logging in as this user use a password. Can only be log in using SSH key.
- `gecos`: optional comment for user
- `group`: groups users is member of. `wheel` is an admin group given to users to perform administrative actions
- `sudo`: user's sudo privleges. 
  - First `ALL` specifies what hosts commands can be ran on
  - Second `(ALL)` specifies what users commands can be run as
  - `NOPASSWD:` enables the use of sudo without a password
  - Third `ALL` allows all commands to be used
  - citaton?
- `disable_root`: If set to true, disables logging as the root user using SSH
  - Every Linux distribution has a root user named `root`, which has unlimited privleges
  - Attackers can much better odds of bruteforcing a password for `root` than bruteforcing  both a username and password
  - Disabling root log in prevents this vulernability
  - NOTE: it's still possible to log in as the root user after connection is established

  https://wiki.archlinux.org/title/Sudo#Disable_root_login
- ref: https://wiki.archlinux.org/title/Users_and_groups#User_groups



https://wiki.archlinux.org/title/Users_and_groups
# Creating a Droplet

With an SSH key pair generated, an Arch Linux cloud image imported, and a cloud-init YAML file created a DigitalOcean Droplet can be created.

1. Choose a Droplet configuration (Droplet size) from the command `doctl compute size list`
This commands list all the Droplet sizes available on DigitalOcean. In this guide, we will s-1vcpu-512mb-10gb or <slug>, the cheapest size available.
(screenshot of output)

2. To create a droplet, use the command
`doctl compute droplet create <droplet-name> --image <image> --size <size> --region <region> --user-data-file <path> --ssh-keys <key>`
- `--image <IMAGE-NAME>` Image to be installed on the droplet, specified in [Adding an Arch Linux Image to DigitalOcean](#adding-an-arch-linux-image-to-digitalocean)
- `--size <SIZE>` Droplet size
- `--region <REGION>` What server Droplet is created on. In this guide we will use `SFO3` 
- `--user-data-file <FILEPATH>` configured cloud-init YAML file from [Createing a Cloud-Init YAML Configuration](#creating-a-cloud-init-yaml-configuration) to be exectued on the Droplet
- `--ssh-keys <PUB-KEY>` name of public key given to DigitalOcean in [Adding a Public Key to your DigitalOcean Account](#adding-a-public-key-to-your-digitalocean-account)

(output)

https://docs.digitalocean.com/reference/doctl/reference/compute/droplet/
# Connecting to Droplet

Once your Droplet is created you can remotely connect to it using `SSH` in your terminal.

1. Copy the public IP of your droplet from the command `doctl compute droplet get <DROPLET-NAME> --format PublicIPv4`
- This command gets information about the droplet `<DROPLET-NAME>`
- The flag `--format` changes the output to show only the public IPv4 address

2. To connect to the Droplet, type in the following command:
`ssh -i <PRIVATE-KEY> arch@<IP>`
- `<PRIVATE-KEY>` Filepath of the corrisponding private key. If you've followed this guide, it should be located in the `~/.ssh` directory.
- `<IP>` The public IP of the droplet from step 1

(Screenshot, first time connecting etc..)

If the connection is sucessful, your command line will change. (be more specific)

# Verifying Configuration

Formatting errors in the YAML file can result in your cloud-init configuration not executing properly. Unfortuneatly, cloud-init won't directly throw errors so we will have to manually check if the configuration was executed. This section will only cover changes made in the YAML script provided. For more information about debugging cloud-init, see [How to debug cloud-init](https://cloudinit.readthedocs.io/en/latest/howto/debugging.html).

## Checking Packages
`pacman -Q` returns the local package database on your system. In other words it returns a list of every package installed on your system. Combining it with string will find packages that pattern matches that string. If you use `pacman -Q <PACKAGE>` with packages listed in the YAML file, you should see the packages listed. If not, cloud-init did not execute the configuration properly.

## Checking User Creation
The `gecos` entries when defining a user section in the YAML file are stored in `/etc/passwd`. By viewing the contents of this file using `cat`, you should be able to see if cloud-init created users: `cat /etc/passwd`

(use a better command here)

https://cloudinit.readthedocs.io/en/latest/howto/debug_user_data.html

# External Resources
(doctl Reference)[https://docs.digitalocean.com/reference/doctl/reference/]

# References
Rudareanu, V., & Baturin, D. (2023). Linux for System Administrators. Packt Publishing. https://learning.oreilly.com/library/view/linux-for-system/9781803247946/
(n.d.). Pacman. Arch Linux Wiki. https://wiki.archlinux.org/title/Pacman#Upgrading_packages
(n.d.). Sudo. Arch Linux Wiki. https://wiki.archlinux.org/title/Sudo
(n.d.). What is SSH? | Secure Shell (SSH) protocol. Cloudflare. https://www.cloudflare.com/learning/access-management/what-is-ssh/
(n.d.). What is asymmetric encryption? Cloudflare. https://www.cloudflare.com/learning/ssl/what-is-asymmetric-encryption/
(n.d.). How does public key cryptography work? Cloudflare. https://www.cloudflare.com/learning/ssl/how-does-public-key-encryption-work/