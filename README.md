# Bontmia

## What is Bontmia?

**Bontmia** stands for **Backup Over Network To Multiple Incremental Archives** and is a network based backup with similar rotation scheme as with tapes.
It saves a configurable number of last month, week, day, hour, and minute backups.
Each backup is a complete snapshot of the original directories.
Only new or changed files consume space when a new snapshot is generated.
It relies heavily on rsync and remote access is handled securely over SSH.

## Bontmia Functionality

* Each backup is a complete "snapshot" of the backed-up directories.
* Files unchanged since the last backup don't take up extra space and aren't transferred over the network.
* It preserves a configurable selection of backups, similar to tape rotation schemes.
* Remote backup is performed encrypted over SSH for security.
* Each backup is stored with its absolute path.
* The bandwidth can be limited to avoid affecting production systems.
* It supports copying files, directories, device files, named pipes, symlinks, and hard links.
* It preserves user ID, group ID, and permissions for all file types.

## Archive Structure

Archives are placed in a directory structure like this:

```
2003/05/06/04:00/
2003/05/07/04:00/
2003/05/08/04:00/
2003/05/08/05:00/
2003/05/08/06:44/
```

This corresponds to `YYYY/MM/DD/HH:MM/`, indicating when the backup was issued.
Since the granularity of the backup archives is one minute, it is not possible to run two different backups within the same minute.

Under these directories, the archived files and directories are stored within a directory named after the hostname backed up and its absolute path, like this:

```
2003/05/06/04:00/foo:/home/jev
                /bar:/home/jev
                /baz:/home/jev
2003/05/07/04:00/foo:/home/jev
                /bar:/home/jev
                /baz:/home/jev
```

## Operation

The following example shows how to perform a daily backup and keep the last 7 days, 4 weeks, 12 months, and 2 years of backups.

```bash
$ bontmia --dest ./backup  --rotation \
                  0minutes0hours7days4weeks12month2years \
                  foo.bar.com:/home/jev \
                  foo.bar.com:/etc \
                  foo.bar.com:/usr/local \
                  foo.bar.com:/var
```

When this command runs, it outputs the following on my system (with the hostname changed).
Since the computer hasn't been online continuously, not all backups have been run, but the last 'x' backups are saved for each filter.
The active filter for each backup is shown. Any backup no longer matching a filter is removed.

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
  Saving /backup/2003/06/29/19:07 by filters:  month
  Saving /backup/2003/07/20/10:00 by filters:  month
  Saving /backup/2003/08/26/22:30 by filters:  weeks month
  Saving /backup/2003/09/07/00:00 by filters:  weeks
  Removing /backup/2003/09/13/00:00
  Saving /backup/2003/09/14/00:00 by filters:  days weeks
  Saving /backup/2003/09/15/00:00 by filters:  days
  Saving /backup/2003/09/16/00:00 by filters:  days
  Saving /backup/2003/09/17/00:00 by filters:  days
  Saving /backup/2003/09/18/00:00 by filters:  days
  Saving /backup/2003/09/19/00:00 by filters:  days
  Saving /backup/2003/09/20/00:00 by filters:  days weeks month years
```

As seen, the last 7 days are saved, the last 4 weeks are saved, and the last 3 months are saved.
The backup has only been running for the last 3 months, so there are no older monthly backups.
The same applies to the yearly backup.

### Example 1

In the following example, filter A has a unit size indicated by the separation between the '|'s. Filter A saves the last 4 backups.

```
backups     b b b b   b b       b                 b            b   b  b
filter A   |         |  s      |s        |        s|         |        s|
```

### Example 2

This example shows how different filters work together. Filter A saves the last 4 backups, filter B saves the last 3, and filter C saves the last 2.

```
backups   b b b b   b   b       b         b     b b   b     b b b b   b
filter A   | | | | | | | | | | | | | | | | | | | | | | | | | |s|s|s| |s|
filter B   |         |         |s        |        s|        s|        s|
filter C   |                    s        |                            s|
------------------------------------------------------------------------
To save                         s                 s         s s s s   s
```

## Fork

This repository holds a fork of the original bontmia, which was available at [http://folk.uio.no/johnen/bontmia/](https://web.archive.org/web/20200715185949/http://folk.uio.no/johnen/bontmia/).

Bontmia was originally written by John Enok Vollestad and later modified by several contributors.
This fork is my attempt to clean up the code, debug it, [adhere to modern bash practices](https://github.com/koalaman/shellcheck/wiki), and merge various improvements shared in different forks.

## Copyright

Bontmia may be used, modified, and redistributed only under the terms
of the GNU General Public License, found in the `COPYING` file in this
distribution, or at
[http://www.fsf.org/licenses/gpl.html](http://www.fsf.org/licenses/gpl.html).
