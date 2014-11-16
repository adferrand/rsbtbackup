RsbtBackup is a simple python tool for incremental backups using rsync and taking advantage of the incremental snapshot capability of a btrfs subvolume.

This tool is inspired from the shell script developed by oxplot : https://github.com/oxplot/rsyncbtrfs.

Requirements
============

You will need rsync, btrfs-progs and of course, a backup partition formatted with btrfs.

Install
=======

Just get the rsbtbackup file and put it anywhere on your backup machine (including outside the btrfs backup partition).

Configuration
=============

RsbtBackup can be used as an importable library in another python script. Just import it in your script :

    $ vi /your/path/yourscript
    #!/usr/bin/env python
    from rsbtbackup import RsbtBackup

RsbtBackup can be used as an standalone script. First, make it executable :

    $ chmod +x /your/path/rsbtbackup

Usage
=====

To settle things, let's say that we manage a local server (our backup machine), and a remote server named "remote". We want to backup the local server's /etc directory and the remote server's /home directory. The local server has a btrfs partition mounted on /backup where the backups will lie.

First, we need to create a directory for each backup. For example :

    $ mkdir -p /backup/localhost/etc
    $ mkdir -p /backup/remote/home

Secondly, we initiate the backup directories and do the backups. The initiation must be done before any backup, otherwise the backup will be aborted : it creates a .log directory inside the backup directories. Within it lies a file named rsbtbackup, and a file name activity.log that contains all errors and informations generated during the backups. The activity.log file is automatically rotated into activity.log.1 when its size exceed 1Mo.

Depending of the usage type of RsbtBackup, init and backup will be done by theses ways :

* As an importable library

RsbtBackup library gives access to the RsbtBackup class, refering to bounded methods init and backup. Theses methods accept one parameter variable taking the form of a python dictionnary. For init it must contain a DESTPATH string, for backup it must contain a DESTPATH and a SCRPATH string. An script example is given below :

    $ vi /your/path/rsbtbackup
    #!/usr/bin/env python
    from rsbtbackup import RsbtBackup

    # New instance of RsbtBackup
    mybackup = RsbtBackup()

    # Initiate the backup directories
    args1 = {'DESTPATH':'/backup/localhost/etc'}
    args2 = {'DESTPATH':'/backup/remote/home'}
    mybackup.init(args1)
    mybackup.init(args2)

    # Do the backups
    args1 = {'DESTPATH':'/backup/localhost/etc','SRCPATH':'/etc'}
    args2 = {'DESTPATH':'/backup/remote/home','SRCPATH':'remote:/home'}
    mybackup.backup(args1)
    mybackup.backup(args2)

* As a standalone executable

Initiate the backup directories :

    $ /your/path/rsbtbackup init /backup/localhost/etc
    $ /your/path/rsbtbackup init /backup/remote/home

Do the backups :

    $ /your/path/rsbtbackup backup /etc /backup/localhost/etc
    $ /your/path/rsbtbackup backup remote:/etc /backup/localhost/etc

In shell help can be shown with the commande :

    $ /your/path/rsbtbackup --help

Result
======

After the first backup, the backup directory will look like (for one usage example given before) :

    $ ls /backup/remote/home
    2014-11-16-09:00 cur

The timestamped directory is a btrfs subvolume and contains the relative backup, and cur is a symlink pointing on the last backup (for our example, 2014-11-16-09:00).

If we run the backup again, the backup directory will look like :

    $ ls /backup/remote/home
    2014-11-16-09:00 2014-11-16-10:00 cur

Then the cur symlink points to the new last backup (here, the btrfs subvolume 2014-11-16-10:00).
