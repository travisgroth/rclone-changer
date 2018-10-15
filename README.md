# rclone-changer

## Intro

### What is rclone-changer

This script wraps rclone (http://rclone.org/) with a bacula (http://www.bacula.org) friendly
interface.  Specifically, it implements the autochanger script interface that bacula uses
to interact with tape changers.  See documentation at http://www.bacula.org/9.2.x-manuals/en/main/Autochanger_Resource.html

### What?

Bacula has a basic understanding of tape libraries.  Because they allow a script to abstract interaction with libraries, we 
can pretend to be a tape changer but actually copy files (virtual tapes) back and forth to various cloud storage locations.

In theory, rclone-changer supports anything that rclone does, but it has only been tested with Amazon Cloud Drive, thus far.

### How do I make this work?


1. Have a working bacula install.  That's beyond scope here.
1. Follow the bacula documentation on setting up an autochanger based storage
    device, pool, etc.
1. Install rclone and initialize it to be able to talk to your cloud storage
    of choice (make sure you can copy files back and forth).  rclone-changer
    expects a working rclone config at /etc/bacula/rclone.conf
1. Create a directory to store the virtual tape files 
1. Point your autochanger command at the rclone-changer script with the correct
    arguments

## Details

### Configuring bacula

It is recommended that you use many smallish volumes rather than large ones.  Try to stay at a size that your average file
fits into without getting too tiny.  The reasoning here is in a recovery situation if you're working on random files you 
want to minimize the amount of unrelated data you need to copy localy in order to recover.  If you have a 10G virtual tape
and only need 500M of it, you still need to wait for the full 10G to download before you can beging recovering data.  Hence,
if your typical file size is averaging 700M or so, 1G volumes are probably a good idea.  There's a soft-coded limit of 8192
tape "slots" in rclone-changer but this can be manually changed by manipulating the state file if required.

#### General recommendations

* Use spooling
* Set a very high Maximum Changer Wait on your device so you have time to actually copy your vtape back and forth
* Pre-label your "tapes" in bacula
* TEST YOUR SETUP

### Configuring rclone-chanager

Not a whole lot goes into configuration, however there are some environmental assumptions you should check on.


1. rclone-changer logs to /var/log/bacula.  Make sure the bacula user has permission to do so (rclone-changer runs as the bacula user)
1. rclone-changer keeps state data in /var/lib/bacula.  Again, check permissions.
1. rclone-changer assumes rclone is in /usr/bin/

#### Examples

Storage Daemon config:

* Virtual tapes are stored in an rclone target called 'ACD', under /Backups/bacula
* The volumes are copied to /stor/nobackup/vtape/acd when retrieved
* Spool is 500M
* Changer wait time is longer than it should ever take to copy a volume, based on size


```
    # bacula-sd.conf
    Autochanger {
      Name = "ACD Autochanger"      # The name of Autochanger
      Device = ACD                  # Must match with bacula-sd Device
      Changer Device = 'ACD:/Backups/bacula'
      Changer Command = "/usr/sbin/rclone-changer %c %o %S %a"
    }

    Device {
      Name = ACD
      Media Type = File
      Maximum Changer Wait = 18000
      Archive Device = /stor/nobackup/vtape/acd
      Autochanger = yes
      LabelMedia = yes                    # lets Bacula label unlabeled media
      Random Access = yes
      AutomaticMount = no                 # when device opened, read it
      RemovableMedia = no
      AlwaysOpen = no
      Spool Directory = /mnt/bacula-spool
      Maximum Spool Size = 524288000
    }
```

Director config:

* 1 month volume retention (my backups run fulls every first monday)
* Volume size is limited to 1G
* Allow recycling and pruning automatically
    
```
    # bacula-dir.conf
    Autochanger {
      Name = ACD                    # A name of Storage
      Address = 192.168.1.1
      SDPort = 9103
      Password = "6Nv2Tt2CJAf1TZA6t1cHtktA5aBUAARmJb/4BSckBLRm"
      Device = ACD                  # Must match with bacula-sd.conf Device
      Media Type = File             # Must match with bacula-sd.conf Media Type
      Maximum Concurrent Jobs = 10
    }   
    
    Pool {
      Name = DisktoDisk-Offsite
      Pool Type = Backup
      Recycle = yes                       # Bacula can automatically recycle Volumes
      AutoPrune = yes                     # Prune expired volumes
      Storage = ACD                       # Must match with bacula-dir.conf autochanger name
      Maximum Volume Bytes = 1073741824
      AutoPrune = yes
      Volume Retention = 4 weeks
    }
```
