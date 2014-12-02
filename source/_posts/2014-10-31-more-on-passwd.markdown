---
layout: post
title: "More On Passwd"
date: 2014-10-31 21:30:45 -0500
comments: true
categories: CodeReading
---

A little bit more interesting discoveries while digging into the passwd code file.

## The 's' flag in file permission

First, the file permission of passwd executable is <code>-rwsr-xr-x</code>. There's an 's' flag which don't usually appear in common Unix files. The usage of the 's' field is explained here:

http://en.wikipedia.org/wiki/Setuid

<!--more-->
Which means when a user runs the passwd program, his effective uid will become the owner of the executable file, which is root in this case. While inside the passwd program, it uses <code>getuid</code>, which returns the user's real id instead of effective id.

## On updating the shadow file

I also do notice that the whole passwd program would only require one Unix system capability: the <code>CAP_FCHOWN</code> capability, which is required when you're changing the owner of one file. Here's why the program needs it.

As a matter of fact, the passwd program never actually directly writes into the <code>/etc/shadow</code> file. For some reason (perhaps security concerns), it writes into a temp file first, set the uid and gid of the temp file, and then rewrite the shadow file with the temp file.

The code is defined in <code>commonio.c</code> file, <code>commonio_update</code> function. As described in the code bellow:

```c
// set the temp filename
snprintf (buf, sizeof buf, "%s+", db->filename);

...

// open the file with name defined in buf, permissions defined in sb, returns the file pointer
db->fp = fopen_set_perms (buf, "w", &sb);

...

// write all data to db->fp, the temp fp
if (write_all (db) != 0){...}

...

// Rename the temp filename in buf to db->filename, which is /etc/shadow
if (lrename (buf, db->filename) != 0){...}
```


> Written with [StackEdit](https://stackedit.io/).
