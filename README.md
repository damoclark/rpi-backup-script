# rpi-backup-script

**Linux-based backup script for backing up Raspberry Pi operating systems (such
as Raspbian) over a network.**

If you are looking to backup your Raspberry Pi to locally attached SD card,
[rpi-clone](https://github.com/billw2/rpi-clone) may be of interest to you.

## Overview

Do you have one or more Raspberry Pi Computers in your home, or workplace? Are
you backing them up? rpi-backup-script provides some reasonably simple shells
scripts to perform backups of your favourite low-powered devices over the
network so that if ever the worst should happen to your pi's memory card (or the
pi itself), you have a means of restoring it back to operation again.

It uses standard Linux-based tools to achieve this including, SSH and Rsync.

You make an initial copy of your SD card using the dd utility (on Linux or OSX)
or win32 Disk Imager (on Windows), and then rpi-backup-script takes over from
there.

![rpi-backup-script overview](https://docs.google.com/drawings/d/1SW6KZjolJTIymInTCZaPm63akU4DX0viFarNtF54nxw/pub?w=960&amp;h=720)

## Software Requirements/Dependencies

* SSH key authentication
* SSH
* Rsync
* blkid
* kpartx

## Optional Dependencies

* zerofree
* fallocate (util-linux 2.25.2 w/dig holes option)

## Dependency Installation Instructions

This section outlines instructions for using package managers of either Redhat
or Debian derived systems to install the pre-requisite software for
rpi-backup-script.

### Redhat derived distributions (e.g. RHEL, Fedora, CentOS)

```bash
$ sudo yum install openssh-clients rsync util-linux kpartx zerofree
```

### Debian derived distributions (e.g. Debian, Ubuntu)

```bash
$ sudo aptitude install openssh-client rsync util-linux kpartx zerofree
```


## Script Installation Instructions

At the command line, download either using:

### git

```bash
$ cd /usr/local
$ sudo git clone https://github.com/damoclark/rpi-backup-script
$ echo 'export PATH=$PATH:/usr/local/rpi-backup-script/bin' >/tmp/rpi-backup-script.sh
$ chmod 755 /tmp/rpi-backup-script.sh
$ sudo mv /tmp/rpi-backup-script.sh /etc/profile.d/
$ . /etc/profile.d/rpi-backup-script.sh
```
or using:

### zip

```bash
$ cd /tmp
$ wget 'https://github.com/damoclark/rpi-backup-script/archive/master.zip'
$ cd /usr/local
$ sudo unzip /tmp/master.zip
```
Now, you should be able to run the script with

```bash
$ backup-rpi -h
```
and receive this output:

```
Usage: backup-rpi.sh [OPTION]... CONFIG_FILE

Backup running Raspberry Pi over network to local image file

  -c    Compare each file for backup rather than simply compare file modify/size
  -n    Dry run (only show what actions would be taken)
  -s    Sparsify the image file
  -z    Run zerofree over the image file after backup complete
  -h    This usage information

You must specify a CONFIG_FILE. Others are optional
```

## Configuration & Usage

This section explains how to use the backup tools and the options available.

###SSH Key Authentication
On the rpi backup server, rpi-backup-script running locally as the root user
needs to be able to login to your Raspberry Pi computers also as root via SSH.
It simply needs this level of access to be able to access and backup all the
files. To do this, you will need to follow the steps below

1. [Enable your SSH Server on all your Raspberry Pi computers](https://www.raspberrypi.org/documentation/remote-access/ssh/%20Enable%20SSH%20on%20Raspbian)
on Boot
1. Give your root user accounts on all your Raspberry Pi computers a password.
To do this, while logged into each Raspberry Pi as the pi user, type:
```sudo passwd```.  You will be asked to provide a password, and then repeat it.
1. [Enable login as root via SSH](https://mike632t.wordpress.com/2015/05/26/enable-root-logins-using-ssh-jessie/)
on each Raspberry Pi to be backed up
1.  [Generate an SSH keypair](https://www.howtoforge.com/linux-basics-how-to-install-ssh-keys-on-the-shell#step-onecreation-of-the-rsa-key-pair)
**on your backup server** as the root user on your backup server
1. [Copy the public key](https://www.howtoforge.com/linux-basics-how-to-install-ssh-keys-on-the-shell#step-threecopying-the-public-key)
generated to the root user of every Raspberry Pi
1. Test that it works by attempting to login to your Raspberry Pi with
```ssh <ip address|hostname of raspberry pi>``` without asking for a password.

### Initial creation of your backup image file
Before you can commence using rpi-backup-script with a Raspberry Pi, you need to
generate an image (or clone) of your SD Card as a file.

You can do this on your rpi backup server, or another computer and copy the
resultant image file onto your rpi backup server.

Firstly, shutdown your Raspberry Pi, and remove the SD Card. Insert the SD Card
into the computer you have chosen to perform the image backup on.

Then follow the
[instructions](https://thepihut.com/blogs/raspberry-pi-tutorials/17789160-backing-up-and-restoring-your-raspberry-pis-sd-card)
provided by The Pi Hut according to your chosen operating system.

There are plenty of other resources that explain the process of creating a
backup image of an SD card.

* [Mac OSX](http://computers.tutsplus.com/articles/how-to-clone-raspberry-pi-sd-cards-using-the-command-line-in-os-x--mac-59911).
* [Windows](http://lifehacker.com/how-to-clone-your-raspberry-pi-sd-card-for-super-easy-r-1261113524)
* [Linux](https://raspberrypi.stackexchange.com/questions/311/how-do-i-backup-my-raspberry-pi)


### Configuration File Format
rpi-backup-script has a configuration file that can be used to define the backup
parameters for your Raspberry Pi/s. For most common usage, all you will need to
change is the name or IP address of the Raspberry Pi, and the path to the backup
image file that you just created, and you are set to go. The format of the
configuration file is nothing more than a shell script, that loads variables
into memory. This means you can use it to create all sorts of customisations if
you so wish.

Here is the contents of the sample configuration file ```etc/raspberrypi.sample```:

Pay attention to the options, ```srchost``` and ```destfile```.  At a minimum,
you may only need to change these options and you are all set.

```bash
#Sample Configuration file for backup-rpi

#Copy this file to a new name, and change the options below to suit

#Specify the hostname or ip address of Raspbery Pi Computer
srchost='raspberrypi'

#Specify the image file to mount
destfile="/backups/raspberrypi.img"

#Specify the local directory on which to mount the image file
destroot='/mnt/raspberrypi'

#Specify the partition number and mount point under destroot for each partition
#in the image
destpart[1]='/boot' #/dev/mmcblk0p1
destpart[2]='/' #/dev/mmcblk0p2

#Rsync default options (dont change unless you know what you are doing)
#Default rsync options for typical UNIX-like filesystems (e.g. ext[234],xfs, etc)
rsyncfs[default]='-av -H -A -X --numeric-ids'
#Default rsync options for MSDOS-based filesystems like vfat
rsyncfs[vfat]='-rvt --modify-window=1'
rsyncfs[msdos]="${rsyncfs[vfat]}"
#Standard rsync options for all filesystems
rsync='--delete --one-file-system'
```
You can save your configuration file to the ```etc``` directory of your
rpi-backup-script installation (either ```/usr/local/etc``` or
```/usr/local/rpi-backup-script/etc``` depending on which instructions you
followed for installation). 

### backup-rpi

Here is the command line usage for the backup-rpi script:
```
Usage: backup-rpi.sh [OPTION]... CONFIG_FILE

Backup running Raspberry Pi over network to local image file

  -c    Compare each file for backup rather than simply compare file modify/size
  -n    Dry run (only show what actions would be taken)
  -s    Sparsify the image file
  -z    Run zerofree over the image file after backup complete
  -h    This usage information

You must specify a CONFIG_FILE. Others are optional
```
As explained, you must specify a config file name for the script to use for the
backup process.  If the config filename isn't an absolute path, or does not
exist in the current working directory, then backup-rpi will attempt to load a
like-named config file from the etc directory of rpi-backup-script itself.
Based on the installation instructions above, this will be either
```/usr/local/etc``` or ```/usr/local/rpi-backup-script/etc```.

For example,
config file ```/usr/local/etc/rpi-server``` could be loaded with:

```bash
# backup-rpi rpi-server
```

While a config file located in ```/home/user/rpi-server``` as:

```bash
# backup-rpi /home/user/rpi-server
```

An explanation of the options to backup-rpi follow:

#### -c : Compare using rsync -c checksum option
By default, rsync will simply compare the size and modify timestamp of each
file.  If the size and timestamp match, then its deemed to be identical. 

The -c option tells rsync to read and compare every file during the backup.
This ensures a thorough copy that is known to be exactly the same. This option
can be useful on the first run, but also if the clock on the Raspberry Pi (or
the rpi backup server) at any
stage is incorrect. 

#### -n : Dry run of the script
By providing the -n option, backup-rpi will go through the steps required to
perform the backup, but will use the rsync --dry-run option preventing it from
actually updating the backup image file.

This option can be useful to determine what is out of date in your backup image,
as compared to your Raspberry Pi. 

#### -s : Sparsify image
The backup image file can be made [sparse](https://en.wikipedia.org/wiki/Sparse_file)
so that it only consumes as much storage on your backup host as is used by your
Raspberry Pi. This option should be used in combination with the -z option
(explained below).

If you use -s option, then backup-rpi will either use ```fallocate -d``` to
sparsify your image file inline (in other words, deallocate areas of the back
file that aren't being used in place), or will make a copy of your image file to
the same directory using ```cp --sparse=always```, and then mv this file back
over your original image if the -d option to fallocate isn't available on your
operating system (CentOS 7 for instance). The former approach is quicker and
uses less disk space. 

#### -z : Zerofree
[Zerofree](http://frippery.org/uml/) is a nifty utility that analyses an
unmounted ext2/3/4 file system locating unused disk space, and then writes zeros
to those locations. This works well in combination with the sparsify option
above, which can then deallocate these contiguous sequences of zeros, thus
saving disk space.

Zerofree will execute before sparsify option, minimising storage usage.

Keep in mind that this software is provided as-is as expressed in the licence
information, and no guarantees are made with respect to functionality. You use
this software at your own risk. 

### mount-rpi
The mount-rpi script is used by backup-rpi, but can be called independently to
mount a Raspberry Pi backup image file to the local filesystem. This can be
useful for accessing the backup files themselves, performing maintenance on the
backup file (such as a filesystem check), retrieving files from your backup
and other such tasks. 

Here is the command line usage for the backup-rpi script:
```
Usage: mount-rpi.sh [OPTION]... CONFIG_FILE

Mount Raspbian image file to directory

  -a    Attach image to device file only (don't mount image)
  -n    Dry run (only show what actions would be taken)
  -r    Read only attachment/mounting
  -f    Fsck (file system check) partitions before mounting
  -h    This usage information

You must specify a CONFIG_FILE. Others are optional
```
Similarly to backup-rpi, mount-rpi requires that you specify a config file name
for the script to use for the mounting process. The config file can be exactly
the same one used for the backup. mount-rpi will only use the configuration
options relevant to mounting the backup image.

#### -a : Attach image to loopback device only
With the -a option, mount-rpi will attach the backup image file to a loopback
device (starting at /dev/loop0 or whatever device is available), but not mount
the separate filesystems/partitions therein to the host's virtual filesystem. 

This can be useful when you wish to operate on the backup image file, without it
being mounted. You may wish to manually perform a fsck (filesystem check) on the
image, run zerofree manually, or other such tasks.

#### -n : Dry run
Like the -n option to backup-rpi, with mount-rpi, -n option will go through the
steps of mounting the image according to the options given, but not actual
perform the attachment or mounting. This gives you an idea of what the script
does, without actually doing it.

#### -r : Read only mount
With -r option, mount-rpi will attach the backup image file to a loopback
device, and mount the partitions/filesystem, but only mount them read only. This
can be useful to prevent accidental writing to the backup image file when you
wish to access its content. 

#### -f : Fsck partitions/filesystems
With the -f option, mount-rpi will attach the backup image file to a loopback
device, but before mounting any partitions/filesystems, will perform a 'forced'
fsck of each partition listed in the config file by running ```fsck -f```. This
can be useful to ensure the integrity of your backup image file prior to any
operations on it.

### umount-rpi
The umount-rpi script is also used by backup-rpi on completion of a backup, but
can be called independently to unmount a Raspberry Pi backup image file after
mount-rpi has been used to mount to the local filesystem. Use this command when
you have finished with your backup image. It will umount any
partitions/filesystems that are listed in the config file, and then detach the
image file from the loopback device. 

Here is the command line usage for the backup-rpi script:
```
Usage: umount-rpi.sh [OPTION]... CONFIG_FILE

Unmount Raspbian image file from directory

  -u    Umount filesystems, but leave image 'attached' to loopback device
  -h    This usage information

You must specify a CONFIG_FILE. Others are optional
```
Similarly to backup-rpi, and mount-rpi, umount-rpi requires that you specify a
config file name for the script to use for the mounting process. The config file
can be exactly the same one used for the backup. umount-rpi will only use the
configuration options relevant to unmounting the backup image.

#### -u : Umount filesystems, but leave image 'attached' to loopback device
So with the -u option, umount-rpi will unmount all the partitions/filesystems
from the host virtual file system, but will leave the loopback device attached
to the file.

This can be useful if you have finished accessing/manipulating the contents of
the backup image, but wish to do further maintenance at a raw level manually
afterwards, such as the use of fsck, zerofree or other tools.

## Caveats
This script does not support LVM. So while it could be used for other
Linux-based operating systems other than Raspberry Pi distros (such as
Raspbian), it will only work if LVM is not in use.

The script must run as the root user on both the backup server, and also have
password-less SSH access (using key-based authentication without a passphrase)
to the root user of every Raspberry Pi that is to be backed up. The upshot is
that if your backup server's root user account is compromised, then the attacker
will have direct access to the root user of all your Raspberry Pi computers.

I'm sure there are plenty other limitations. See how you go.

## TODO
Look at the [TODO.md](TODO.md) file for details.

## Licence 
Copyright (c) 2016 Damien Clark<br/>

Licenced under the terms of the [GPLv3](https://www.gnu.org/licenses/gpl.txt)<br/>
![GPLv3](https://www.gnu.org/graphics/gplv3-127x51.png "GPLv3")

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL DAMIEN CLARK BE LIABLE FOR ANY DIRECT, INDIRECT,
INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. 




