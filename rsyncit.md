# rsyncit

`rsyncit` is a shell script that I use to synchronize directories on my computer with directories on the many web sites I administer. It allows me to place a file in a directory I want to sync that contains all the necessary information to `ssh` to the remote host. I use keypairs instead of passwords. Always.

`rsyncit` requires `rsync`, which is included in the standard MacOS install, but which you need to install yourself in most Linux distros. It's always one of the first things I add, along with `emacs`, `github`, `gcc`, and [Clozure Common Lisp](http://clozure.com/).

Here's the script:

```
#!/bin/bash
if [[ -z $* ]];then rsync -av ./ `./.sshdir`;else rsync $* `./.sshdir`;fi
```

The magic executable in each directory is named `.sshdir`. Here's the one in my [steemit source directory]('https://steemit.com/kakuro/@billstclair/organizing-steemit-source):

```
echo $DO:/var/www/billstclair.com/steemit
```

`DO` is shell variable set in one of my login scripts. It's short for "Digital Ocean", my web hosting provider. Nowadays, I add symbolic names to `~/.ssh/config` instead, but that's an old one.

There are two ways to use the `rsyncit` script.

The following synchronizes the whole directory, and all its sub-directories.

```
rsyncit
```

The following synchronizes just the listed files:

```
rsyncit -av file1 file2 ...
```
