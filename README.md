git@github.com:fountum/acit-2420-assignment-1.git# acit-2420-assignment-1

Assignment 1 for ACIT 2420

  
- [Introduction](#Introduction)
- [Prerequisites](#Prerequisites)
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


# Installing and Configuring doctl

https://docs.digitalocean.com/reference/doctl/how-to/install/
`sudo pacman -S doctl`
api token
- **only shown once when created** copy and save it in a file
- has experation date
- enable CRUD operations

`doctl auth init --context <NAME>`
enter API token
`doctl auth switch --context <NAME>`

verify it works:
`doctl account get`


# Creating an SSH Key Pair
`ssh-keygen` utility should be installed on your Windows/MacOs/Linux system.

`ssh-keygen -t ed25519 -f ~/.ssh/do-key -C 'arch-linux-09-21'`

`-t` Type of encryption used to generate the key.
`-f` Path where the key is created. 
`-C` an optional comment. this is appended on to the end of the public key

Check folder for keys, add screenshots

# Adding a Public Key to your DigitalOcean Account 
https://docs.digitalocean.com/reference/doctl/reference/compute/ssh-key/
copy public key
`doctl compute ssh-key create <key-name> --public-key <pub-key>`
`key-name` key's name on DigitalOcean
`pub-key` contents of public key folder

verify it's added: `doctl compute ssh-key list`

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