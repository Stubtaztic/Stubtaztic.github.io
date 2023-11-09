# A detailed step-by-step guide to installing, configuring, and customizing Arch Linux 

## Step 1 - Downloads

Download a virtual machine that is appropriate for your OS (Windows/Mac OS/Linux).

Download the current Arch Linux ISO torrent file [here](https://archlinux.org/download/).

Next download and install qBitTorrent (for the appropriate OS of course) [here](https://www.qbittorrent.org/download).

## Step 2 - Configure ISO file

Open and go through the setup for the qBitTorrent.

Then put the downloaded ISO torrent file into qBitTorrent and wait for it to be unpacked.

Once unpacked, go to your devices download folder and find the raw ISO file.

## Step 3 - Configure Virtual Machine


### Difference between Mac and Windows

**Mac:**

Open the Virtual Machine appliction.

Drag and drop the downloaded ISO file into the dropbox of the setup wizard.

Click continue and seclet Linux OS. 

Scroll through the configuration options until you find **Linux 5.x kernel 64-bit**

On the last setup wizard step, select the UEFI boot firmare and click finish.

Once finished, go to the Virtual Machine option in the top taskbar and click settings.

Go to Processor & Memory and change the processor cores to 2 and memory to 2048MB (2GB).

Then go to Hark Disk (SCSI) and change disk size to 20GB. Check the advanced options and make sure the bus type is SCSI

**Windows:**

Open the Virtual Machine appliction.

Select Installer disk image file (ISO) and select the downloaded ISO file.

Choose Linux as the Guest Operating System and select **Linux 5.x kernel 64-bit** as the version.

Name the virtual machine _Arch Linux_

set disk size to 20GB

On the last part of the set up go to Customize Hardware and go to Memory.

Set the memory to 2GB.

Finish the setup wizard the go to the application folder and find the Arch Linux file with the extention *.vmx

**Be very careful with this part!!**

Open into notepad or VScode and return the first line so that you have a blank line 2.

It should look like this:
```
Line 1: .encoding = "windows-1252"
Line 2:
Line 3: config.version = "8"
```
in the 2nd line, write:
```
firmware = "efi"
```

It should now look like this:
```
Line 1: .encoding = "windows-1252"
Line 2: firmware = "efi"
Line 3: config.version = "8"
```
Save the file return to the Virtual Machine application.

## Step 4 - Boot (_setup cont._)

Start the virtual machine and wait for the boot to stop at the terminal. 

You should have a terminal line that says: 
```
root@archiso ~ #
```

Verify the boot is in fact UEFI by listing the efivars directory by using the following command:

```
# ls /sys/firmware/efi/efivars
```

Verify the network interface is up and running:

```
# ip link
```

Verify the network is connected (you can terminate the process with ^C):

```
# ping archlinux.org
```

Use `timedatectl` to verify the time/date is accurate:
```
# timedatectl status
```

## Step 5 - Partitioning Disks (_setup cont._)

The disks do not auto partition themselves unlike other operating systems. We have to do this part maunally.

To see which disks we have to work with use:

```
# fdisk -l
```

In our case we will be working with the /dev/sda disk

Access that disk with:

```
# fdisk /dev/sda
```

Now we want to add new partitions. We can see what commands we need to use to do this by typing m for help. 

To create the first new partition type `n` then press enter.

Type `p` for primary then press enter.

Type `1` for default then press enter.

We want the starting point to be at the beginning of the disk space so leave this part blank and press enter.

We want for it to have 500MB of space so type `+500M` then press enter.

**Your new partition should be finished.**

Now we repeat the steps with only small changes.

Create the first new partition type `n` then press enter.

Type `p` for primary then press enter.

Type `2` for default then press enter.

We want the starting point to be at the beginning of the last partition point on the disk space so leave this part blank and press enter.

We want to give it the remaining space so leave this part blank then press enter.

**Your second partition should be finished.**

## Step 6 - Formatting Partitions (_setup cont._)

We want to make the dev/sda2 or root partition to Ext4 file system. Type this command in to do so:

```
# mkfs.ext4 /dev/sda2
```

For the dev/sda1 or EFI partition, we want to format it to FAT32. This this command in to do so:

```
# mkfs.fat -F 32 /dev/sda1
```
## Step 7 - Mount the Partitions (_setup cont._)

Now we are going to mount the partitions.

For the root partition, we want to mount it to `/mnt`. To do so we type:

```
mount /dev/sda2 /mnt
```

For the EFI partition, we want to mount it to `/mnt/boot`. To do so we type:

```
mount --mkdir /dev/sda1 /mnt/boot
```

## Step 8 - Installation

We will now download the essential packages for the Linux system. The `pacstrap` command will do this for us:

```
# pacstrap -K /mnt base linux linux-firmware
```

Once this finishes downloading you may move on to the next step. 

## Step 9 - Configuration

### 9.1
We will generate an Fstab file:

```
# genfstab -U /mnt >> /mnt/etc/fstab
```
### 9.2
Change root into the new system:

```
# arch-chroot /mnt
```
### 9.3

Make sure the timezone data is correct by directly installing `tzdata`

```
# pacman -S tzdata
```
Create a symbolic link for the time zone:

```
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```
Set the time zone:

```
# systemctl set-timezone /America/Chicago
```

Run `hwclock` to generate `/etc/adjtime`

```
# hwclock --systohc
```
### 9.4

Install nano:

```
# pacman -S nano
```
### 9.5
Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and the other needed UTF-8 locales. Generate these locales by running:

```
# locale-gen
```

We can now access and edit `/etc/locale.gen` with the nano command:

```
# nano /etc/locale.gen
```
Scroll through the file until you find `en_US.UTF-8 UTF-8` and delete the `#`.

Then use `^x` to exit and save the file 
##

Create the `local.conf` file using nano

```
# nano locale.conf
```

Once inside the file, write:

```
LANG=en_US.UTF-8
```
Again, use `^x` to exit and save the file

Now you have to move this file in the /etc directory so we will type:

```
# mv locale.conf /etc
```

Create new initramfs with:

```
# mkinitcpio -P
```

### 9.6

Now we want need to configure the network environment. Create the hostname file with nano. Once inside name it whatever you wish to be the hostname; I did arch_vm.

Then move this file to /etc as well:

```
# mv hostname /etc
```

Also, we need to install network manager. 

```
# pacman -S networkmanager grub
```

### 9.7 

Set the root password:

```
# passwd
```

### 9.8 

Now we need to create and set up the boot loader. We start off with making the boot EFI directory:

```
# mkdir /boot/EFI
```
Install efibootmgr:

```
# pacman -S efibootmgr
```
Next, we need to install the boot loader:

```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --modules="tpm" --disable-shim-lock
```

Then make a grub config file with:

```
# grub-mkconfig -o /boot/grub/grub.cfg
```
Install lxde, the desktop environment as well:

```
# pacman -S lxde
```

Install xorg-xinit:

```
# pacman -S xorg-xinit
```

Nano create `~/.xinitrc` and write the following script. 

```
# nano ~/.xinitrc
```
```
#!bin/zsh
#
# ~/.xinitrc
#
# Executed with startx

if [ -d /etc/X11/xinit/xinitrc.d ]; then
    for f in /etc/X11/xinit/xinitrc.d/*; do
        [ -x "$f" ] && . "$f"
    done
    unset f
fi

exec startlxde
```

Then, give it executable permissions.

```
chmod +x ~/.xinitrc
```


Finally install the remaining packages:

```
# pacman -S man-db man-pages texinfo zsh sudo openssh
```
## Step 10 - Finishing the VM

We are almost done with the arch linux set up. Now we need to exit out of the chroot environment by typing:

```
# exit
```
Once exiting we want to shutdown the virtual machine and remove the installation medium. Go to the settings of the virtual machine. Open CD/DISK, click advanced options, remove .iso disk.

## Step 11 - Configuring the DE (Desktop Environment)

### 11.1

Once you have restarted the system, let to boot into the terminal and login with root. 

**IMPORTANT!!!**

We want the network manager to work when we use the VM/Desktop enviornment so we need to enable the network manager. This will allow us to continue using pacman and any other command or action that requires internet. This command will enable it (Yes, the capital N and M are required because this is how linux recognizes and names the NetworkManager.service file):

```
# systemctl enable NetworkManager
```

Double check that it is running with:

```
# systemctl status NetworkManager
```

### 11.2
Now type `startx`. This will boot you into the GUI Desktop Environment.

After entering the environment you will go to program tools and open the terminal inside the DE. From here, we will do the remaining set up. 

First we will create two new users; codi and ethan (myself). The requirement of the assignment requests to set codi's password to `GraceHopper1906` and to force a change after first login. The password for the self user is important for yourself to remember but for the sake of documentation I will put something random:

```
# useradd -p asdf1234 ethan
```
```
# useradd -p GraceHopper1906
# passwd --expire codi
```

We can double check that codi's password will be forced to change on next login with:
```
# chage -l codi
```
The assignmnet also asks to give both users sudo permissions. First, we need to edit the `visudo` file so that it can recognize the group sudo. Make sure vi is installed as this is a unique editor unlike nano that does a syntax check on the file before saving and exiting. This is important with files like visudo where an incorrect syntax can corrupt the file/system.

```
# pacman -S vi
```

Now we open the visudo file with:

```
# sudo visudo
```

Scroll down until you find the line that says, `"## Uncomment to allow members of the group sudo to execute any command"`. Remember how I said that the vi editor is unique unlike nano? Well, it has special controls. To delete the comment in front of the line, move the white box (typeline) over the # symbol in front of the line `# %sudo ALL=(ALL:ALL) ALL`. Then press the `x` key on your keyboard. To save an exit type `:wq` (The semi-colon is required to type)

Now that the sudo group is recognized, we can create the sudo group:

```
# groupadd sudo
```

We want to give the created users codi and ethan sudo privilages. We can do this with the `usermod` command:

```
# sudo usermod -aG sudo codi
# sudo usermod -aG sudo ethan
```

Also, I switched the shell of codi and ethan to zsh. I did this with the following commands:

```
# usermod --shell /bin/zsh codi
# usermod --shell /bin/zsh ethan
```
To make the root user boot into zsh when opening the terminal edit the `.bashrc` file with nano and write `zsh`:
```
# nano .bashrc

zsh
```
Save and exit.

Lastly, we will give each user a home directory.

```
# mkdir /home/codi
# cp -rT /etc/skel /home/codi
# chown -R codi:codi /home/codi
```
```
# mkdir /home/ethan
# cp -rT /etc/skel /home/ethan
# chown -R ethan:ethan /home/ethan
```
### 11.3

Since we are using a different shell, we need to go through the configuration of a new zsh user.

Reopen the terminal which will open as root and will start the zsh (we will repeat these steps for the other users).
```
# su <user>
```

A pop up for the configuation will be displayed in the terminal along the lines of "This is the z shell configuration function for new users". Press (1) to enter the main menu for configuration. Now you will see multiple options to setup. Press (1) for the history and set to default; then save and exit. Press (2) for the completion and press (1) for default. Finally, press (3) for keymap and press (1) to set to Emac binding. Press (e) then (0). To complete the configuration, press (0).

Close and reopen the terminal to make sure that root and all other users boot into zsh.

### 11.4

We want to customize our DE to our liking so we will install a broswer and change the color of the terminal.

To install the brower (I chose firefox), type:
```
# pacman -S firefox
```

Go through the installation with the default setup. Leave the first and second prompt empty and simply hit enter. When prompted Y/n, type y and press enter.

To change the color of the terminal, we are going to create enable the `promptinit` program. Start with being logged in as root. nano into the `.zshrc` file. Move to the bottom of the file (Below the "# End of lines added my compinstall") and write the following: 

```
# This bit is to autoboot into the DE on login
startx

autoload -Uz promptinit && promptinit

prompt_mytheme_setup() {
  PS1='%F{green}[%n%m]%f%F{blue}%B%~%b%f:$ '
  RPS1='%F{white}%t%f'
}

prompt_themes+=( mytheme )

prompt mytheme
```

Save and exit the file, then type the following commands into the shell (as root in the root home directory):

```
# mkdir ~/.zprompts 
# fpath=("$HOME/.zprompts" "$fpath[@]")
# ln -s mytheme.zsh ~/.zprompts/prompt_mytheme_setup
```

Finally, copy the `.zshrc` file to the other users:

```
# cp /root/.zshrc /home/ethan
```

switch user to the other users and nano edit the `.zshrc` file by changing this line:

```
zstyle :compinstall filename '/root/.zshrc'
```

To this (put the correct user's name in place of the one written):

```
zstyle :compinstall filename '/home/ethan/.zshrc'
```

### 11.5 

Last but not least, we will create some alias for the `.zshrc` file. These are basically handmade shortcuts that can save time in the terminal instead of typing everything out. Nano into the file (you can set different aliases for each user or edit the `/etc/zshrc` file for sytemwide aliases), go to the very bottom and add the following:

I will add a comment over those that aren't self explanitory but you don't need to add the comment to the `.zshrc` file.
```
alias ls='ls --color=auto'
alias ll='ls -la'
alias l.='ls -d .* --color=auto'
alias su='su -l'
alias c='clear'
alias h='history'
alias reboot='sudo /sbin/reboot'
alias poweroff='sudo /sbin/poweroff'
alias shutdown='sudo /sbin/shutdown'
alias install='pacman -S'
alias update='pacman -u'
```

## Step 12 - End

That's all! 

The Arch Linux virtual machine is up, running, and personalized with style.
