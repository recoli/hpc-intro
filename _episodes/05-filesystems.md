---
layout: episode
title: "File systems"
teaching: 10
exercises: 15
questions:
  - "What is the difference between the AFS and Lustre file systems?"
  - "How can I change access control lists to share files with colleagues?"
objectives:
  - "Learn to navigate the file systems at PDC."
  - "Learn to work with access control lists."
keypoints:
  - "AFS and Lustre are two different file systems with different properties."
  - "Access control lists enable you to share files with colleagues."
---

# File systems at PDC

When working on a PDC cluster, you will be interacting with two different file systems. 
Their properties are important to understand in order to make the most out 
of PDC's resources.

## Andrew File System - AFS

- AFS is a (globally) distributed file system accessible to any running AFS client.
- Your main home directory (`$HOME`) is on AFS: `/afs/pdc.kth.se/home/<initial>/>username>`
- Access to your AFS home directory is enabled by your Kerberos ticket, and the *AFS token* generated by it.

## Lustre file system (klemming)

- Nicknamed *klemming* at PDC.
- Lustre is an open-source massively parallel distributed file system.
- Very high performance (5PB storage - 130GB/s bandwidth).
- PDC users have two klemming directories:
  - `/cfs/klemming/nobackup/<initial>/<username>`: used for running jobs, input, output...
  - `/cfs/klemming/scratch/<initial>/<username>`: used for temporary files (old files get deleted).
- High file access speed, but performance deteriorates for many (~10<sup>5</sup>) small files.

## Differences between AFS and Lustre

| AFS         | Lustre         |
| ----------- | -------------- |
| Home directory backed up every day, lost files can be retrieved | Not backed up |
| *Not accessible* to compute nodes on Beskow | Accessible to all compute nodes |
| Limited disk quota, typically 5 GB per user | No strictly enforced quota |
| Rather slow    | High-performance |
| Own implementation of access control lists | Standard access control lists |

> ## Exploring your AFS directory
>
> If you haven't already done so, log in to Tegner:
> ```
> $ ssh <username>@tegner.pdc.kth.se
> ```
> {: .bash}
> After logging in, check your current location:
> ```
> [tegner]$ pwd
> ```
> {: .bash}
> This is your AFS home directory, which is also stored in the `HOME` 
> environment variable:
> ```
> [tegner]$ echo $HOME
> /afs/pdc.kth.se/home/<initial>/<username>
> ```
> {: .bash}
> You can check your disk quota and how much of it you're using
> with the following command:
> ```
> [tegner]$ fs listquota
> # or the shortcut: 
> [tegner]$ fs lq
> ```
> {: .bash}
> This is an example of a `fs` subcommand. To see all 
> available subcommands, type
> ```
> [tegner]$ fs help
> ```
> {: .bash}
> **Which command can you use to list the access control lists (ACLs)?**
> 
> You also have two "special" directories already set up in your AFS 
> home directory: `Public` is a publicly readable directory and is useful 
> for sharing files with collaborators, and `Private` is a non-readable, 
> non-listable directory for your secrets.   
> **Use the command you found for listing access control lists to compare
> the ACLs in `$HOME`, `$HOME/Public` and `$HOME/Private`.**
{: .challenge}


> ## Your AFS backups
> 
> If you accidentally remove a file from your AFS home, you can 
> retrieve it from yesterday's backup in:
> `$HOME/.OldFiles`   
> If it's been longer than a day since you accidentally removed the file, 
> it might still be retrievable, just contact PDC support!
{: .callout}

> ## Exploring your Lustre directories
>
> You can go to your directories on the Lustre file system by
> ```
> [tegner]$ cd /cfs/klemming/nobackup/<initial>/<username>
> ```
> {: .bash}
> and
> ```
> [tegner]$ cd /cfs/klemming/scratch/<initial>/<username>
> ```
> {: .bash}
> One way to avoid having to type out the full path of your Lustre `nobackup` and 
> `scratch` directories over and over again is to define *aliases* in your 
> bash startup file `$HOME/.bashrc`. Everything in this file gets executed 
> when you log in to PDC. So by adding the following two lines to your `.bashrc`, 
> you only need to type `scr` or `nobak` to go to your Lustre directories:
> ```
> alias scr='cd /cfs/klemming/scratch/<initial>/<username>/'
> alias nobak='cd /cfs/klemming/nobackup/<initial>/<username>/'
> ```
> {: .bash}
> Finally, to see how many files you have and how much disk space they use, type 
> ```
> [tegner]$ lfs quota -u $USER /cfs/klemming
> ```
> {: .bash}
> (note the use of the environment variable `$USER`, which is your username)  
{: .challenge}

## Access control lists

- AFS has extended implementation of ACLs, where users can define new groups.
  - Seven different permissions set on *directory level*.
- Lustre supports standard ACLs using mode bits.
  - Three different permissions (read, write, execute) set on *file level*.

**For AFS**, the possible permissions are listed in the table below.

|Notation|File action| Description                      |
| ------ | --------- | -------------------------------- |
|r	 | read      | read the files in the directory  |
|l	 | list      | list the files in the directory  |
|i	 | insert    | create new files                 |
|d	 | delete    | delete files                     |
|w	 | write     | modify files                     |
|k	 | lock	     | lock files in the directory      |
|a	 | administer|change the ACL for the directory  |

**For Lustre**, the POSIX (Portable Operating System Interface)
file system permission model defines
three classes of users called `owner`, `group`, and `other`,
and the possible permissions are read (`r`), write (`w`) and execute (`x`).  
- The command `ls -l` displays the `owner`, `group`, and `other` class permissions 
in the first column of its output. 
- For a regular file with read+write access for `owner`, read access for `group` and no access for `other`, the first column would show `-rw- r-- ---`.

The three base permissions have an equivalent representation as ACLs, 
which can be displayed by the `getfacl` command.
```bash
[tegner]$ getfacl -a dir
# file: dir
# owner: <my-username>
# group: users
user::rwx
group::r-x
other::---
```

To change ACLs of files and directories, the `setfacl` command can be used, which is 
more general than the `chmod` command which only changes file permissions.

> ## Modifying ACLs on AFS
>
> To alter the ACLs of a directory in your AFS home, use the command 
> (where `sa` is short for `setacl`)
> ```
> [tegner]$ fs sa <directory> <user> <permissions> 
> ```
> {: .bash}
> See the table above for possible AFS permissions.  
> **For this exercise, pair up with someone sitting next to you, and try the following:**
> 1. Create a new directory in your AFS home folder (`$HOME`). Change the ACL of that 
> directory so your friend gets `read` and `write` access.
> 2. Try writing a file in each other's shared directories (most simply, just do `$ touch foo`)
> 3. Afterwards, remove the directory that you shared in your `$HOME`.
{: .challenge}

> ## Modifying ACLs on Lustre
>
> To alter the ACLs of a directory in `/cfs/klemming` directories, use the command 
> ```
> [tegner]$ setfacl -m u:<username>:<permissions> -R /cfs/klemming/nobackup/<initial>/<username>/<some-directory>
> ```
> {: .bash}
> The possible file permissions are `r`, `w` and `x`. `-m` stands for *modify* and
> `-R` is for recursive.
> To allow your colleague to read a file in your klemming directory, you need to 
> use `r-x` (`x` to allow traversal through directories).  
> **For this exercise, pair up again with someone sitting next to you.**
> **Then do the following:**
> 1. Create a new directory in your `/cfs/klemming/nobackup` directory. 
> Change the ACL of that directory so your friend gets `read` access.
> 2. Write a file in your new directory, and see if your friend can copy it.
> 3. Remove the `read` permissions for your friend again, and retry copying.
{: .challenge}


