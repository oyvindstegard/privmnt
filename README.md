# privmnt - easy ad-hoc management of personal encrypted file systems

Tool for easy ad-hoc management of personal encrypted file systems on
Linux using dm-crypt and LUKS.

Create, open/mount and unmount/close encrypted file systems on demand
through very simple commands. It is meant to be invoked as regular
user, but due to using Linux kernel block level encryption facilities,
the *user must have administrative privileges on the system*, and *sudo
will be invoked by privmnt where required*. Privmnt attempts to ensure
that any failed operations are properly cleaned up, checks for many
error conditions and hides the tedious sequence of commands which are
necessary to mount, unmount and properly close dm-crypt/LUKS encrypted
file systems.

It is not meant to be used as a system tool to setup general purpose
shared encrypted disk partitions, although it is able to create
standard encrypted file systems on block devices, as well as in
regular files.

If you're already using properly set up full disk encryption on your
computer, you probably don't need this tool. It's more useful when
handling smaller encrypted "vaults" inside your computer, or when
using encrypted thumb drives, which can be opened and closed on
demand.

Privmnt can be invoked without a terminal and prompt for encryption
passphrase using a graphical dialog box. It supports a simple
graphical informational user interface through
[Zenity](https://github.com/GNOME/zenity) for all operations exception
the `create` command, which does require an interactive terminal.

## Requirements and installation

This script requires a Linux distribution of some sort, as the
technology used is Linux specific. The
[cryptsetup](https://gitlab.com/cryptsetup/cryptsetup) command must be
installed.

Copy the script to some place in your PATH:

    sudo cp privmnt /usr/local/bin/
    sudo chmod 755 /usr/local/bin/privmnt
    
.. or similar.

Give it a test:

    privmnt -h
    
If it complains about missing requirements, check the command paths at
the top of the script, and make sure they are present on the system.
If the paths are wrong, adjust to suit your Linux distro by editing
the privmnt script directly.

## Usage

    Use: privmnt COMMAND [OPTIONS]

    Manage and create personal encrypted file systems

    COMMAND may be one of:
    m, mount   Open encrypted file system and mount it.
    u, umount  Unmount encrypted file system and close it.
    t, toggle  Mount if not already mounted, umount otherwise.
    s, status  Test if mounted and exit with code 0 if so,
               or a non-zero code otherwise. No output is produced.
    c, create  Make a new encrypted file system (terminal interactive wizard)

    Options:
    -d <dir>, set the mount directory, default: /home/$USER/Private
    -i <img>, set path to file/device containing LUKS encrypted file system image
              By default, a storage file path is derived from mount directory.
    -m <opts>, set mount options for file system
    -s Silent errors, do not fail even if already unmounted or mounted
    -h Show this help.

    Commands that require root privileges will be invoked using /usr/bin/sudo.
    This includes /sbin/cryptsetup, /bin/lsblk, /bin/mount and /bin/umount.

    Version 1.0


### Create a private file system, with storage in a file

Issue command, and assuming your username is `theuser`:

    privmnt create -d ~/Private
    
This will start an interactive wizard in the terminal.

    # Create a new encrypted file system
    Use <CTRL+C> to abort wizard.

    ## Enter directory path where it should be mounted:
    Path> /home/theuser/Private

Hit enter to accept the default mount point as specified with option
`-d`.

    ## Enter a file or block device path to use as storage:
    Path> /home/theuser/.Privatefs

The suggested place to use as storage for the new encrypted file
system is a regular file that sits alongside the mount point
directory, and is named similarly. You can also specify a block device
(partition or disk) instead, like `/dev/sdb1`. In that case, the whole
partition is used as storage.

    Using new file /home/theuser/.Privatefs as storage
    ## Enter desired size of storage file in in mega og gigabytes:
    Size(xM or xG)> 1G

The size needs to be preallocated, and you must specify the size of
the file used as storage. Here we say one gigabyte of storage. This
will give somewhere close to one gigabyte of free space in the
encrypted file system to be created.

    ## Select file system type (choose ext4 if unsure):
    1) ext4
    2) ext2
    3) vfat
    #? 1

Select option 1 to create a standard ext4 file system.

    ## Summary of parameters:
    Mount at:            /home/theuser/Private
    Using storage file:  /home/theuser/.Privatefs
    File system:         ext4
    Size:                1024 M

    Proceed ? [y/N]> y

Finally, we are presented with a summary of the options. Type `y
<RETURN>` to proceed, or just hit enter to abort.

    ## Creating directory /home/theuser/Private ..
    ## Creating empty file at /home/theuser/.Privatefs ..
    ## Filling storage with pseudo-random data, this can take a while ..

In this step, the storage file is filled with pseudo-random data. This
operation can take a long time, depending on how large the file system
is. For large block devices/partitions, it can take very long. This
examples does not use a block device directly, but only a small 1GiB
file, so it will be quick.

    1024+0 records in
    1024+0 records out
    1073741824 bytes (1,1 GB, 1,0 GiB) copied, 9,29179 s, 116 MB/s

    ## Setting up LUKS on /home/theuser/.Privatefs ..

    WARNING!
    ========
    This will overwrite data on /home/theuser/.Privatefs irrevocably.

    Are you sure? (Type uppercase yes): YES
    
The next step is cryptsetup formatting the storage file into an
encrypted LUKS-container.

    Enter passphrase for /home/theuser/.Privatefs:
    Verify passphrase: 

    ## Opening encrypted device ..
    Enter passphrase for /home/theuser/.Privatefs:

    
Type and verify your desired passphrase, then type it a third time to
open the LUKS container. The setup process continues.

    ## Creating file system ..
    [sudo] password for theuser: 
    
At this point, sudo is invoked to access the mapped block device which
represents the opened LUKS container. The file system will now be
created.

    mke2fs 1.44.1 (24-Mar-2018)
    Creating filesystem with 261632 4k blocks and 65408 inodes
    Filesystem UUID: ac188730-8fa3-43d8-9165-974c74fe1bac
    Superblock backups stored on blocks: 
    	32768, 98304, 163840, 229376

    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (4096 blocks): done
    Writing superblocks and filesystem accounting information: done

    ## Mounting and adjusting permissions ..
    ## Unmounting ..
    ## Closing encrypted device ..
    ## Completed the process.

After creating file system, permissions are adjusted for maximum
privacy, then it is unmounted and the LUKS container is closed. It is
now ready to use.

    ## To mount the newly created encrypted file system, use command:

        privmnt mount -d '/home/theuser/Private'

    ## To unmount and close the encrypted file system, use command:

        privmnt umount -d '/home/theuser/Private'

The process ends with a description of the commands required to mount
and access the encrypted file system, and unmount/close it.

When using the default directory `~/Private` as mount point, and a
default storage file `~/.Privatefs`, then you do not need to specify
any options to mount or unmount. Just say `privmnt m` to mount and
`privmnt u` to unmount. You can also call these commands without a
terminal, e.g. run from window manager, and you will get a graphical
prompt for passphrase and progress status.

### Mount a private file system

Mount at `MOUNTPOINT` with storage in a regular `FILE`:

    privmnt m -d MOUNTPOINT -i FILE
    
Specifying `-i FILE` is not necessary if the file exists in the same
directory as the mount point and is named like the basename of the
mount point directory with a leading dot and a suffix of "fs".

### Unmount and close a private file system

Unmount encrypted file system mounted at `MOUNTPOINT`:

    privmnt u -d MOUNTPOINT
    
This unmounts the file system and closes the LUKS container.

### Setting up sudo to avoid password prompts

This tool invokes sudo for system commands that require root
privileges. To avoid getting prompted about sudo-password in addition
to your encrypted file system passphrase, the following can be added
to `/etc/sudoers`:

    Cmnd_Alias PRIVMNT = /sbin/cryptsetup *, /sbin/lsblk *, /bin/mount *, /bin/umount *
    theuser ALL=NOPASSWD: PRIVMNT
    
Assuming `theuser` is your username.

## Setting up an encrypted file system on a thumb drive

Find out which device is your thumbdrive. A helpful command may be
`blkid` or `lsblk` to show block devices attached to the system.

For the following example, we assume that `/dev/sdh` is the device of
the inserted USB thumb drive. You must be absolutely certain of this,
as all data on the drive will be erased.

Create an encrypted file system on the drive:

    privmnt c -i /dev/sdh -d ~/securethumb

(The mount directory does not matter, you can mount it anywhere you want. It is 
only asked as a convenience.)
    
Follow the wizard. The entire thumbdrive will be used as storage.

Mount thumb drive:

    privmnt m -i /dev/sdh -d ~/securethumb
    
Unmount and secure thumb drive:

    privmnt u -d ~/securethumb

The thumb drive will also be mountable using standard operating system 
procedures for LUKS-encrypted volumes, and likely you will be asked of 
passphrase by your desktop environment automatically when inserting the thumb 
drive. Privmnt can be useful when such streamlined integrations are not in 
place, or you prefer to control the mounting manually.

## Security considerations

There is no replacement for a proper understanding of important
security aspects when dealing with encrypted file systems. Here are
some things to think about:

1. Use a long passphrase that you can easily remember. Most real
   security is lost if a weak password is chosen to secure your data.
   
2. Be aware of surroundings. When an encrypted file system is open and
   mounted, the files are accessible to the operating system in
   general. This means that tools like mlocate/slocate, backup
   services and indexing services might access the files without you
   being aware of it. Contents of files can be cached by programs you
   use to work with private data, and store the cache on unencrypted
   media.
   
   This point can be significantly mitigated by using full disk
   encryption in addition, but then you likely will not find privmnt
   useful.

3. Data integrity. With LUKS-formatted encryption containers, the real
   master decryption key is stored as part of the header. Be aware
   that if the first 2 MiB of the storage device gets corrupted or
   becomes unreadable, generally all data in the LUKS container is
   lost. Therefore, LUKS header backups is recommended, but how to do
   that is beyond the scope of this document (see cryptsetup/LUkS 
   docs).
   
   Also, when using a regular file as storage, the file cannot safely
   be copied while the encrypted file system is mounted (risk of
   corrupting the encrypted file system). You must unmount before
   taking a backup of it, for instance.
   
4. File system permissions. Privmnt tries to ensure that the file
   systems it creates limit permissions to the current user. This
   applies both to the encrypted storage file (file system image) and
   when the encrypted file system is mounted. Still, you should double
   check that permissions look ok when using it on a shared system.
   
5. Specific to this tool, when using the graphical interface for
   obtaining the passphrase, the secret will be stored in memory
   temporarily in the zenity and privmnt bash process, before being
   passed on to cryptsetup. You can avoid this and directly input to
   cryptsetup by using the terminal interface (e.g. invoking privmnt
   from a terminal instead of running directly from window system.). I
   personally do not deem this a great risk, but it is something to
   keep in my when using a shared system.

## Testing

Privmnt has been tested on Ubuntu 16.04 and Ubuntu 18.04.

If you've tested it on other distributions, I'd like to hear about it,
so I can update this section with the results.

## Author

Øyvind Stegard <oyvind@stegard.net>

## License

Distributed under the [GPL v3
license](https://opensource.org/licenses/GPL-3.0), this is free and
open source software.
