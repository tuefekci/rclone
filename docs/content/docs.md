---
title: "Documentation"
description: "Rclone Usage"
date: "2015-06-06"
---

Configure
---------

First you'll need to configure rclone.  As the object storage systems
have quite complicated authentication these are kept in a config file
`.rclone.conf` in your home directory by default.  (You can use the
`--config` option to choose a different config file.)

The easiest way to make the config is to run rclone with the config
option:

    rclone config

See the following for detailed instructions for

  * [Google drive](/drive/)
  * [Amazon S3](/s3/)
  * [Swift / Rackspace Cloudfiles / Memset Memstore](/swift/)
  * [Dropbox](/dropbox/)
  * [Google Cloud Storage](/googlecloudstorage/)
  * [Local filesystem](/local/)
  * [Amazon Cloud Drive](/amazonclouddrive/)
  * [Backblaze B2](/b2/)
  * [Hubic](/hubic/)
  * [Microsoft One Drive](/onedrive/)
  * [Yandex Disk](/yandex/)

Usage
-----

Rclone syncs a directory tree from one storage system to another.

Its syntax is like this

    Syntax: [options] subcommand <parameters> <parameters...>

Source and destination paths are specified by the name you gave the
storage system in the config file then the sub path, eg
"drive:myfolder" to look at "myfolder" in Google drive.

You can define as many storage paths as you like in the config file.

Subcommands
-----------

### rclone copy source:path dest:path ###

Copy the source to the destination.  Doesn't transfer
unchanged files, testing by size and modification time or
MD5SUM.  Doesn't delete files from the destination.

Note that it is always the contents of the directory that is synced,
not the directory so when source:path is a directory, it's the
contents of source:path that are copied, not the directory name and
contents.

If dest:path doesn't exist, it is created and the source:path contents
go there.

For example

    rclone copy source:sourcepath dest:destpath

Let's say there are two files in sourcepath

    sourcepath/one.txt
    sourcepath/two.txt

This copies them to

    destpath/one.txt
    destpath/two.txt

Not to

    destpath/sourcepath/one.txt
    destpath/sourcepath/two.txt

If you are familiar with `rsync`, rclone always works as if you had
written a trailing / - meaning "copy the contents of this directory".
This applies to all commands and whether you are talking about the
source or destination.

### rclone sync source:path dest:path ###

Sync the source to the destination, changing the destination
only.  Doesn't transfer unchanged files, testing by size and
modification time or MD5SUM.  Destination is updated to match
source, including deleting files if necessary.

**Important**: Since this can cause data loss, test first with the
`--dry-run` flag to see exactly what would be copied and deleted.

Note that files in the destination won't be deleted if there were any
errors at any point.

It is always the contents of the directory that is synced, not the
directory so when source:path is a directory, it's the contents of
source:path that are copied, not the directory name and contents.  See
extended explanation in the `copy` command above if unsure.

If dest:path doesn't exist, it is created and the source:path contents
go there.

### move source:path dest:path ###

Moves the source to the destination.

If there are no filters in use this is equivalent to a copy followed
by a purge, but may using server side operations to speed it up if
possible.

If filters are in use then it is equivalent to a copy followed by
delete, followed by an rmdir (which only removes the directory if
empty).  The individual file moves will be moved with srver side
operations if possible.

**Important**: Since this can cause data loss, test first with the
--dry-run flag.

### rclone ls remote:path ###

List all the objects in the the path with size and path.

### rclone lsd remote:path ###

List all directories/containers/buckets in the the path.

### rclone lsl remote:path ###

List all the objects in the the path with modification time,
size and path.

### rclone md5sum remote:path ###

Produces an md5sum file for all the objects in the path.  This
is in the same format as the standard md5sum tool produces.

### rclone sha1sum remote:path ###

Produces an sha1sum file for all the objects in the path.  This
is in the same format as the standard sha1sum tool produces.

### rclone size remote:path ###

Prints the total size of objects in remote:path and the number of
objects.

### rclone mkdir remote:path ###

Make the path if it doesn't already exist

### rclone rmdir remote:path ###

Remove the path.  Note that you can't remove a path with
objects in it, use purge for that.

### rclone purge remote:path ###

Remove the path and all of its contents.  Note that this does not obey
include/exclude filters - everything will be removed.  Use `delete` if
you want to selectively delete files.

### rclone delete remote:path ###

Remove the contents of path.  Unlike `purge` it obeys include/exclude
filters so can be used to selectively delete files.

Eg delete all files bigger than 100MBytes

Check what would be deleted first (use either)

    rclone --min-size 100M lsl remote:path
    rclone --dry-run --min-size 100M delete remote:path

Then delete

    rclone --min-size 100M delete remote:path

That reads "delete everything with a minimum size of 100 MB", hence
delete all files bigger than 100MBytes.

### rclone check source:path dest:path ###

Checks the files in the source and destination match.  It
compares sizes and MD5SUMs and prints a report of files which
don't match.  It doesn't alter the source or destination.

### rclone dedupe remote:path ###

Interactively find duplicate files and offer to delete all but one or
rename them to be different. Only useful with Google Drive which can
have duplicate file names.

```
$ rclone dedupe drive:dupes
2016/01/31 14:13:11 Google drive root 'dupes': Looking for duplicates
two.txt: Found 3 duplicates
  1:       564374 bytes, 2016-01-31 14:07:22.159000000, md5sum 7594e7dc9fc28f727c42ee3e0749de81
  2:      1744073 bytes, 2016-01-31 14:07:12.490000000, md5sum 851957f7fb6f0bc4ce76be966d336802
  3:      6048320 bytes, 2016-01-31 14:07:02.111000000, md5sum 1eedaa9fe86fd4b8632e2ac549403b36
s) Skip and do nothing
k) Keep just one (choose which in next step)
r) Rename all to be different (by changing file.jpg to file-1.jpg)
s/k/r> r
two-1.txt: renamed from: two.txt
two-2.txt: renamed from: two.txt
two-3.txt: renamed from: two.txt
one.txt: Found 2 duplicates
  1:         6579 bytes, 2016-01-31 14:05:01.235000000, md5sum 2b76c776249409d925ae7ccd49aea59b
  2:         6579 bytes, 2016-01-31 12:50:30.318000000, md5sum 2b76c776249409d925ae7ccd49aea59b
s) Skip and do nothing
k) Keep just one (choose which in next step)
r) Rename all to be different (by changing file.jpg to file-1.jpg)
s/k/r> k
Enter the number of the file to keep> 2
one.txt: Deleted 1 extra copies
```

The result being

```
$ rclone lsl drive:dupes
   564374 2016-01-31 14:07:22.159000000 two-1.txt
  1744073 2016-01-31 14:07:12.490000000 two-2.txt
  6048320 2016-01-31 14:07:02.111000000 two-3.txt
     6579 2016-01-31 12:50:30.318000000 one.txt
```

### rclone config ###

Enter an interactive configuration session.

### rclone help ###

Prints help on rclone commands and options.

Server Side Copy
----------------

Drive, S3, Dropbox, Swift and Google Cloud Storage support server side
copy.

This means if you want to copy one folder to another then rclone won't
download all the files and re-upload them; it will instruct the server
to copy them in place.

Eg

    rclone copy s3:oldbucket s3:newbucket

Will copy the contents of `oldbucket` to `newbucket` without
downloading and re-uploading.

Remotes which don't support server side copy (eg local) **will**
download and re-upload in this case.

Server side copies are used with `sync` and `copy` and will be
identified in the log when using the `-v` flag.

Server side copies will only be attempted if the remote names are the
same.

This can be used when scripting to make aged backups efficiently, eg

    rclone sync remote:current-backup remote:previous-backup
    rclone sync /path/to/files remote:current-backup

Options
-------

Rclone has a number of options to control its behaviour.

Options which use TIME use the go time parser.  A duration string is a
possibly signed sequence of decimal numbers, each with optional
fraction and a unit suffix, such as "300ms", "-1.5h" or "2h45m". Valid
time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

Options which use SIZE use kByte by default.  However a suffix of `k`
for kBytes, `M` for MBytes and `G` for GBytes may be used.  These are
the binary units, eg 2\*\*10, 2\*\*20, 2\*\*30 respectively.

### --bwlimit=SIZE ###

Bandwidth limit in kBytes/s, or use suffix k|M|G.  The default is `0`
which means to not limit bandwidth.

For example to limit bandwidth usage to 10 MBytes/s use `--bwlimit 10M`

This only limits the bandwidth of the data transfer, it doesn't limit
the bandwith of the directory listings etc.

### --checkers=N ###

The number of checkers to run in parallel.  Checkers do the equality
checking of files during a sync.  For some storage systems (eg s3,
swift, dropbox) this can take a significant amount of time so they are
run in parallel.

The default is to run 8 checkers in parallel.

### -c, --checksum ###

Normally rclone will look at modification time and size of files to
see if they are equal.  If you set this flag then rclone will check
the file hash and size to determine if files are equal.

This is useful when the remote doesn't support setting modified time
and a more accurate sync is desired than just checking the file size.

This is very useful when transferring between remotes which store the
same hash type on the object, eg Drive and Swift. For details of which
remotes support which hash type see the table in the [overview
section](/overview/).

Eg `rclone --checksum sync s3:/bucket swift:/bucket` would run much
quicker than without the `--checksum` flag.

When using this flag, rclone won't update mtimes of remote files if
they are incorrect as it would normally.

### --config=CONFIG_FILE ###

Specify the location of the rclone config file.  Normally this is in
your home directory as a file called `.rclone.conf`.  If you run
`rclone -h` and look at the help for the `--config` option you will
see where the default location is for you.  Use this flag to override
the config location, eg `rclone --config=".myconfig" .config`.

### --contimeout=TIME ###

Set the connection timeout. This should be in go time format which
looks like `5s` for 5 seconds, `10m` for 10 minutes, or `3h30m`.

The connection timeout is the amount of time rclone will wait for a
connection to go through to a remote object storage system.  It is
`1m` by default.

### -n, --dry-run ###

Do a trial run with no permanent changes.  Use this to see what rclone
would do without actually doing it.  Useful when setting up the `sync`
command which deletes files in the destination.

### --ignore-existing ###

Using this option will make rclone unconditionally skip all files
that exist on the destination, no matter the content of these files.

While this isn't a generally recommended option, it can be useful
in cases where your files change due to encryption. However, it cannot
correct partial transfers in case a transfer was interrupted.

### --log-file=FILE ###

Log all of rclone's output to FILE.  This is not active by default.
This can be useful for tracking down problems with syncs in
combination with the `-v` flag.

### --low-level-retries NUMBER ###

This controls the number of low level retries rclone does.

A low level retry is used to retry a failing operation - typically one
HTTP request.  This might be uploading a chunk of a big file for
example.  You will see low level retries in the log with the `-v`
flag.

This shouldn't need to be changed from the default in normal
operations, however if you get a lot of low level retries you may wish
to reduce the value so rclone moves on to a high level retry (see the
`--retries` flag) quicker.

Disable low level retries with `--low-level-retries 1`.

### --modify-window=TIME ###

When checking whether a file has been modified, this is the maximum
allowed time difference that a file can have and still be considered
equivalent.

The default is `1ns` unless this is overridden by a remote.  For
example OS X only stores modification times to the nearest second so
if you are reading and writing to an OS X filing system this will be
`1s` by default.

This command line flag allows you to override that computed default.

### -q, --quiet ###

Normally rclone outputs stats and a completion message.  If you set
this flag it will make as little output as possible.

### --retries int ###

Retry the entire sync if it fails this many times it fails (default 3).

Some remotes can be unreliable and a few retries helps pick up the
files which didn't get transferred because of errors.

Disable retries with `--retries 1`.

### --size-only ###

Normally rclone will look at modification time and size of files to
see if they are equal.  If you set this flag then rclone will check
only the size.

This can be useful transferring files from dropbox which have been
modified by the desktop sync client which doesn't set checksums of
modification times in the same way as rclone.

When using this flag, rclone won't update mtimes of remote files if
they are incorrect as it would normally.

### --stats=TIME ###

Rclone will print stats at regular intervals to show its progress.

This sets the interval.

The default is `1m`. Use 0 to disable.

### --delete-(before,during,after) ###

This option allows you to specify when files on your destination are
deleted when you sync folders.

Specifying the value `--delete-before` will delete all files present on the
destination, but not on the source *before* starting the transfer
of any new or updated files.

Specifying `--delete-during` (default value) will delete files while checking
and uploading files. This is usually the fastest option.

Specifying `--delete-after` will delay deletion of files until all new/updated
files have been successfully transfered.

### --timeout=TIME ###

This sets the IO idle timeout.  If a transfer has started but then
becomes idle for this long it is considered broken and disconnected.

The default is `5m`.  Set to 0 to disable.

### --transfers=N ###

The number of file transfers to run in parallel.  It can sometimes be
useful to set this to a smaller number if the remote is giving a lot
of timeouts or bigger if you have lots of bandwidth and a fast remote.

The default is to run 4 file transfers in parallel.

### -v, --verbose ###

If you set this flag, rclone will become very verbose telling you
about every file it considers and transfers.

Very useful for debugging.

### -V, --version ###

Prints the version number

Configuration Encryption
------------------------
Your configuration file contains information for logging in to 
your cloud services. This means that you should keep your 
`.rclone.conf` file in a secure location.

If you are in an environment where that isn't possible, you can
add a password to your configuration. This means that you will
have to enter the password every time you start rclone.

To add a password to your rclone configuration, execute `rclone config`.

```
>rclone config
Current remotes:

e) Edit existing remote
n) New remote
d) Delete remote
s) Set configuration password
q) Quit config
e/n/d/s/q>
```

Go into `s`, Set configuration password:
```
e/n/d/s/q> s
Your configuration is not encrypted.
If you add a password, you will protect your login information to cloud services.
a) Add Password
q) Quit to main menu
a/q> a
Enter NEW configuration password:
password>
Confirm NEW password:
password>
Password set
Your configuration is encrypted.
c) Change Password
u) Unencrypt configuration
q) Quit to main menu
c/u/q>
```

Your configuration is now encrypted, and every time you start rclone
you will now be asked for the password. In the same menu you can 
change the password or completely remove encryption from your
configuration.

There is no way to recover the configuration if you lose your password.

rclone uses [nacl secretbox](https://godoc.org/golang.org/x/crypto/nacl/secretbox) 
which in term uses XSalsa20 and Poly1305 to encrypt and authenticate 
your configuration with secret-key cryptography.
The password is SHA-256 hashed, which produces the key for secretbox.
The hashed password is not stored.

While this provides very good security, we do not recommend storing
your encrypted rclone configuration in public, if it contains sensitive
information, maybe except if you use a very strong password.

If it is safe in your environment, you can set the `RCLONE_CONFIG_PASS`
environment variable to contain your password, in which case it will be
used for decrypting the configuration.

If you are running rclone inside a script, you might want to disable 
password prompts. To do that, pass the parameter 
`--ask-password=false` to rclone. This will make rclone fail instead
of asking for a password, if if `RCLONE_CONFIG_PASS` doesn't contain
a valid password.


Developer options
-----------------

These options are useful when developing or debugging rclone.  There
are also some more remote specific options which aren't documented
here which are used for testing.  These start with remote name eg
`--drive-test-option` - see the docs for the remote in question.

### --cpuprofile=FILE ###

Write CPU profile to file.  This can be analysed with `go tool pprof`.

### --dump-bodies ###

Dump HTTP headers and bodies - may contain sensitive info.  Can be
very verbose.  Useful for debugging only.

### --dump-filters ###

Dump the filters to the output.  Useful to see exactly what include
and exclude options are filtering on.

### --dump-headers ###

Dump HTTP headers - may contain sensitive info.  Can be very verbose.
Useful for debugging only.

### --memprofile=FILE ###

Write memory profile to file. This can be analysed with `go tool pprof`.

### --no-check-certificate=true/false ###

`--no-check-certificate` controls whether a client verifies the
server's certificate chain and host name.
If `--no-check-certificate` is true, TLS accepts any certificate
presented by the server and any host name in that certificate.
In this mode, TLS is susceptible to man-in-the-middle attacks.

This option defaults to `false`.

**This should be used only for testing.**

Filtering
---------

For the filtering options

  * `--delete-excluded`
  * `--filter`
  * `--filter-from`
  * `--exclude`
  * `--exclude-from`
  * `--include`
  * `--include-from`
  * `--files-from`
  * `--min-size`
  * `--max-size`
  * `--min-age`
  * `--max-age`
  * `--dump-filters`

See the [filtering section](/filtering/).

Exit Code
---------

If any errors occurred during the command, rclone will set a non zero
exit code.  This allows scripts to detect when rclone operations have
failed.

