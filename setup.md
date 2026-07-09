# Installations

## install virt-manager
- head over to [virt-manager](https://virt-manager.org/)
- copy the vm manager app installation command for your distro
    ```bash
    # Debian example
    sudo apt-get install virt-manager
    ```
- run the command to install virt-manager

## install RHEL DVD ISO
- head over to [developers.redhat.com](https://developers.redhat.com/products/rhel/download#downloadsbyrelease)
- install latest Red Hat Enterprise Linux (DVD iso, around 10.3 GB)

### create VM and install RHEL on it using virt-manager
- open virt-manager app
- on the top left side click on the monitor icon (Create a new virtual machine)-> choose **Local install media (ISO image or CDROM)**
- browse and open the installed Red Hat Enterprise Linux ...dvd.iso
- click forward or continue
- give memory: 2048 (2GB), CPU: 2
- give storage 20 (default), do not go lower than 15GB.
- on the last screen give it name like: `rhel10-server1` and click finish (it might ask you that virtual network is not created and it wants to create one so click yes and continue and start the install)

### setup user, password and ssh
- choose language
- on the next screen (installation smmary), below system options click on Installation Destination (automatic partitioning) -> click done
- back on installation summary screen, under software options click on software selection and make sure **Server with GUI** is selected if you want GUI (do for the master server)
- back on installation summary screen, under user settings click on root account -> enable root password -> enter root password -> check **allow root SSH login with password**
- back on installation summary screen, under user settings click on user creation -> enter username -> check both user admin and require password options -> create password for user -> done
- back on installation summary screen, under system options click on Network & host name -> on the bottom left side, enter host name like: master-server.ibrahimmoalim.com -> apply -> done
- back on installation summary screen, on the bottom right, click on Begin Installation (this will take sometime)
- once it's done, click on reboot system on the bottom right
- register the server with RHEL developer free account or paid organization account, go to settings -> system -> registration -> enter username and password for RHEL account -> leave organization field blank if in no organization -> register
- set the timezone if it's wrong
    ```bash
    sudo timedatectl set-timezone Africa/Mogadishu
    ```
    Verify:
    ```bash
    timedatectl
    ```

### create a snapshot
- before you install things, once you login, create a snapshot of clean server so we can go back to it if things go sideways
- on the top right side click on **Manage VM Snapshots** (2 monitors icon) -> on the bottom left side, click on ** Create new snapshot** (plus icon) -> give it name like: `first-snapshot-clean-install` -> finish -> click on the far top left side monitor icon (show the graphical console) to go back.
- shutdown the VM by click on power icon in one of the options on the top bar, then close window

### create 2nd VM with RHEL (Minimal Server)
- go to virt-manager and click on create new VM
- choose the same ...dvd.iso and follow the same steps as above (you can make it 15GB storage this time, name it `...-server2`)
- on installation summary screen, under software options -> software selection -> this time choose **minimal install** instead of server with GUI for server 2
- give it it's own root password and allow ssh login through that password
- create user
- - back on installation summary screen, under system options click on Network & host name -> on the bottom left side, enter host name like: server2.ibrahimmoalim.com -> apply -> done
- begin installation and reboot when it's done
- this time instead of graphical login, you'll be thrown into a terminal
- enter username (first input), password on second input
- register the system:
    ```bash
    sudo subscription-manager register --username 'USERNAME' --password 'PASSWORD'
    ```
- attach a subscription

    Usually, the registration command will automatically try to attach a subscription for you. To make absolutely sure your system has found your Developer subscription, run:
    ```bash
    sudo subscription-manager attach --auto
    ```
- verify registration status
    ```bash
    sudo subscription-manager status
    ```
- set the timezone if it's wrong
    ```bash
    sudo timedatectl set-timezone Africa/Mogadishu
    ```
    Verify:
    ```bash
    timedatectl
    ```
- set the fontsize to a normal size if the current one is too small
    - see available fonts
    ```bash
    ls /usr/lib/kbd/consolefonts
    ```
    - apply font (this one is normal size)
    ```bash
    setfont sun12x22
    ```
    - set it permanently
    ```bash
    sudo vim /etc/vconsole.conf
    ```
    ```bash
    FONT="sun12x22"
    ```
- get auto completion for commands with `tab`
```bash
sudo dnf install bash-completion -y && exec bash
```
- ctrl+d or exit to logout
- create a snpashot

### start VMs after shutting them down
- click on the VM on virt-manager
- click on the play button on top to start up the server

### settings and tweaks
- get rid of the system sounds (like aleart sound when you make a mistake in the terminal) in settings -> sounds -> alert sounds -> none. Or turn off system sounds if that option is there
- set screen blank in power settings to **never**
