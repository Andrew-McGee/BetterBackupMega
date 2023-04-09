# BetterBackupMega

BetterBackupMega is a BASH shell script to enhance functionality of MEGA utilities to back up files to the MEGA cloud. Particularly it provides the ability to track updated or new files and only backup those as an incremental backup rather than the existing limitation of MEGAcmd that performs a full backup every time.

Note: This is not file sync to or from MEGA. This is sending and storing regular file backups to MEGA cloud storage.

Important prerequisites:
- You need to have MEGAcmd installed and in your path. https://github.com/meganz/MEGAcmd
- Run mega-cmd at least once to create login token.
- Tested on linux (Fedora 37) bash shell. Should work on other bash shells with the following utils:
- find command to build file list
- diff to compare file lists and build incremental back up list (new files and modified files)
- You need permissions to the backup directories and files - will create file list files in the root directory being backed up

## What it does today

Two executable scripts, **bkup-full** and **bkup-incr** run either a full backup or incremental backup respectively. You must run a full backup at least once before running and incremental, as this creates a file listing of your root backup directory and all sub directories. This plain text file is called 'filelist-now' and is saved in the root of the directory you are backing up (hence you need to run the script with write permissions). For a full backup it uses the find command to recursively list every file and directory with the full pathname and last modified date.

After creating the file list the full backup will run the mega-put command for each file. Note that mega-put queues all these transfers as jobs, so the backup script may end but the transfers could still be queued and running. The last thing the script does after completing the mega-puts is to execute the mega-transfers command to give you an indication if jobs are still queued. Note by default it shows you 10 items. Use the mega-transfers command again to continue to monitor the queue.

When you run **bkup-incr** command it will first look for 'filelist-now' in the root directory you are backing up. It will copy this to 'filelist-last' and generate a new 'filelist-now' using the find command. Next it runs a diff on the two files to determine new files in the list and files with a different last modified date. These differences are saved in 'filelist-incr' in the backup root directory. This new incremental list is then used to backup files using mega-put the same way as the **bkup-full** described above.

By default all backups will be stored in a remote directory called 'backups' in the root of your mega cloud storage. You can change this by editing the dest variable in both the **bkup-full** and **bkup-incr** scripts.

Full backups will be saved in a subdirectory with the prefix 'bkup-full' and date and time. This way you can have multiple full backups that will exist in different subdirectories.

Incremental backups will be saved in a subdirectory with the prefix 'bkup-incr' and date and time. Incremental directories only store the files selected to backup by the **bkup-incr** script. This way you can have multiple versions of the same file in different time stamped subdirectories.

## Usage

You must pass the root backup directory (without trailing slash) as an argument to both **bkup-full** and **bkup-incr**. Example:

``./bkup-full /home/username/docs``

Will backup all regular files and subdirectories recursively from /home/username/docs.

## Limitations

- Does not follow links as find only looks for regular files
- Very basic bare bones - don't expect robust error checking or logging
- No compression, dedupe or encryption (will look at in future)
- No exclude lists (future)
- No management of existing backups (merging, pruning, synthetic full etc)
- No restore ability (use existing mega tools)
- No liability or warranty - USE WITH CAUTION - UNDERSTAND WHAT YOU ARE DOING!
