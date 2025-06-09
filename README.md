Bontmia
=======

# What is bontmia

It stands for **Backup Over Network To Multiple Incremental Archives** and
is a network based backup with similar backup scheme as with tapes.
Saves configurable numbers of last month, week, day, hour and minute
backups. Each backup is a complete snapshot of the original
directories. Only new/changed files takes up space when generating a
new snapshot. Remote access is done over ssh.

# Bontmia Functionality

* Each backup is a complete "snapshot" of the backed up directories.
* Files that is not changed since the last backup does not take up extra space and is not transferred over the network.
* Preserves a configurable selection of backups in a similar manner as tape rotation schemes.
* Remote backup is done encrypted over ssh for security reasons.
* Each backup is stored with absolute path.
* It is possible to limit the bandwidth to avoid affecting production systems.
* Support copying files, directories, device files, named pipes, symlinks and hard-links.
* Preserves user id, group id and permissions for all types of files. 

# Archive structure

The archives is placed in a directory structure like this:

```
2003/05/06/04:00/
2003/05/07/04:00/
2003/05/08/04:00/
2003/05/08/05:00/
2003/05/08/06:44/
```

which is `YYYY/MM/DD/HH:MM/` when the backup was issued. Since the granularity of the backup archives is one minute there is not possible to run two different backups within the same minute.

Under these directories the archived files and directories are stored within a directory named the hostname backed up and absolute path. Like this:

```
2003/05/06/04:00/foo:/home/jev
                /bar:/home/jev
                /baz:/home/jev
2003/05/07/04:00/foo:/home/jev
                /bar:/home/jev
                /baz:/home/jev
```

# Operation

The following shows an example of how to do backup once every day and keep the last 7 days, 4 weeks, 12 monthly and 2 years. The hostname is changed.

```
$ bontmia --dest ./backup  --rotation \
                  0minutes0hours7days4weeks12month2years \
                  foo.bar.com:/home/jev \
                  foo.bar.com:/etc \
                  foo.bar.com:/usr/local \
                  foo.bar.com:/var
```

When this is run it outputs the following on my system (the hostname is changed). Since the computer has not been on all the time not all the backups have been run but the last x backups is saved for each filter. Which filter that is active for each backup is shown. The one removed is not longer filtered to be saved.

```
bdirmatch: ^(./foo.bar.com:/home/jev|foo.bar.com:/etc|foo.bar.com:/usr/local|foo.bar.com:/var)
Hardlink to files in last backup when unchanged
  (/backup/2003/09/19/00:00)
  foo.bar.com:/home/jev

  Continuing with the next backup source

  foo.bar.com:/etc

  Continuing with the next backup source

  foo.bar.com:/usr/local

  Continuing with the next backup source

  foo.bar.com:/var

  Continuing with the next backup source


Moving the complete backup into the backup archive
  (/backup/unfinished_backup -> /backup/

Calculates which backups to save
  Saving /backup/2003/06/29/19:07 by filters:  month
  Saving /backup/2003/07/20/10:00 by filters:  month
  Saving /backup/2003/08/26/22:30 by filters:  weeks month
  Saving /backup/2003/09/07/00:00 by filters:  weeks
  Removing /backup/2003/09/13/00:00
  Saving /backup/2003/09/14/00:00 by filters:  days weeks
  Saving /backup/2003/09/15/00:00 by filters:  days
  Saving /backup/2003/09/16/00:00 by filters:  days
  Saving /backup/2003/09/17/00:00 by filters:  days
  Saving /backup/2003/09/18/00:00 by filters:  days
  Saving /backup/2003/09/19/00:00 by filters:  days
  Saving /backup/2003/09/20/00:00 by filters:  days weeks month years
```

As one can see the last 7 days is saved, the last 4 weeks is saved and the last 3 month is saved. The backup have only run for the last 3 month and therefore there is no more month backups. Similar for the year backup.

## Example 1
In the following example the filter A have a unite size indicated by the separation between the |'s. The filter A is saving the last 4 backups.

```
backups     b b b b   b b       b                 b            b   b  b
filter A   |         |  s      |s        |        s|         |        s|
```

## Example 2
This example shows how different filters work together. Filter A saves the last 4 backups, filter B saves the last 3 and filter C saves the last 2.

```
backups   b b b b   b   b       b         b     b b   b     b b b b   b
filter A   | | | | | | | | | | | | | | | | | | | | | | | | | |s|s|s| |s|
filter B   |         |         |s        |        s|        s|        s|
filter C   |                    s        |                            s|
------------------------------------------------------------------------
To save                         s                 s         s s s s   s
```

# Fork

This repository holds a fork of the original bontmia available at
[http://folk.uio.no/johnen/bontmia/](https://web.archive.org/web/20200715185949/http://folk.uio.no/johnen/bontmia/)

Bontmia was originally written by John Enok Vollestad and later modified by several contributors.
This fork is my attempt at cleaning-up the code, debugging it, [respect modern bash practices](https://github.com/koalaman/shellcheck/wiki) and merging the various improvements shared in different forks.

# Copyright

Bontmia may be used, modified and redistributed only under the terms
of the GNU General Public License, found in the file COPYING in this
distribution, or at
[http://www.fsf.org/licenses/gpl.html](http://www.fsf.org/licenses/gpl.html)
