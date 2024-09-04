---
layout: post
title: "A Dive Into Password"
date: 2014-10-13 21:08:55 -0500
comments: true
categories: CodeReading
tags: [Password, CodeReading]
---

## Passwd

The passwd is program on Unix systems to manage users' passwords. The user and password information on most Unix systems is stored in two separate files: /etc/passwd for user information, and /etc/shadow for password information, including encrypted password value, expiration data, UID, GID, and etc.. The
rationale behind storing information in separate files is discussed in [Why shadow your passwd file?](http://www.tldp.org/HOWTO/Shadow-Password-HOWTO-2.html)
<!--more-->

The passwd program is a part of the shadow-utils, which includes a series of programs to manage user accounts, group accounts, and converting plain passwords to shadow password format, such as: groupadd, useradd, usermod, login, passwd, su, and etc..

The source file of <code>passwd</code> is available in [Debian Alioth Page](http://pkg-shadow.alioth.debian.org/) and [Debian Package Information Page](https://packages.debian.org/source/wheezy/shadow). I downloaded the 4.1.5 original source for my study. See opensource isn't just a campaign slogan, it's something real!

Basically, what passwd does is to manage passwords, like updating passwords,  setting minimum and maximum password expiration date, and all these information is saved to the /etc/passwd and /etc/shadow files, and password information in particular, needs an encryption library for protection, instead of being saved as plain text.

Below is a __NON__-comprehensive list of code files required by the passwd program in the shadow-utils code base:
<pre>
src/passwd.c
lib/defines.h
lib/getdef.(h/c)
lib/shadow.(h/c)
lib/shadowio.(h/c)
lib/commonio.(h/c)
lib/sgetspent.c
</pre>


## How we change our password

### Data structure definitions first?

* __Shadow passwd struct__:
Defined in shadow.h in linux include directory. Defines the structure of the shadow file. The pointer in the main function is defined as:

```c
const struct spwd *sp;  /* Shadow file entry for user   */
...
/* Structure of the password file.  */
struct spwd
{
    char *sp_namp;              /* Login name.  */
    char *sp_pwdp;              /* Encrypted password.  */
    long int sp_lstchg;         /* Date of last change.  */
    long int sp_min;            /* Minimum number of days between changes.  */
    long int sp_max;            /* Maximum number of days between changes.  */
    long int sp_warn;           /* Number of days to warn user to change
    the password.  */
    long int sp_inact;          /* Number of days the account may be
    inactive.  */
    long int sp_expire;         /* Number of days since 1970-01-01 until
    account expires.  */
    unsigned long int sp_flag;  /* Reserved.  */
};
```

* __Passwd Structure__:
Defined in pwd.h in linux include directory. Defines the <code>/etc/passwd</code> file structure. The pointer in the main function is defined:
```c
const struct passwd *pw;  /* Password file entry for user      */
...
/* The passwd structure.  */
struct passwd
{
    char *pw_name;                /* Username.  */
    char *pw_passwd;              /* Password.  */
    __uid_t pw_uid;               /* User ID.  */
    __gid_t pw_gid;               /* Group ID.  */
    char *pw_gecos;               /* Real name.  */
    char *pw_dir;                 /* Home directory.  */
    char *pw_shell;               /* Shell program.  */
};
```

* __Shadow\_db structure__: Defined in <code>lib/shadowio.c</code>. It's a doubly linked list, storing information for all the shadow file entries. It's declared with a type called <code>struct commonio_db</code>, defined in <code>commonio.h</code>.

    Shadow library has <code>commonio.c</code> for all the common io data structures and operations, and <code>shadowio.c</code>, which could be seen as a wrapper around common io for all shadow file data structure and operations.

    The <code>shadow_db</code> defined as below. The <code>SHADOW_FILE</code> as you might have already guessed, is a macro defined as <code>"/etc/shadow"</code>.
    
```c
static struct commonio_db shadow_db = {
    SHADOW_FILE,            /* filename */
    &shadow_ops,            /* ops */
    NULL,                   /* fp */
    #ifdef WITH_SELINUX
    NULL,                   /* scontext */
    #endif
    NULL,                   /* head */
    NULL,                   /* tail */
    NULL,                   /* cursor */
    false,                  /* changed */
    false,                  /* isopen */
    false,                  /* locked */
    false                   /* readonly */
};
```

* Some important global variables
```c
static char *name;   /* The name of user whose password is being changed */
static char *myname; /* The current user's name */
static bool amroot;  /* The caller's real UID was 0 */
static char crypt_passwd[256];

```


### Main function

Though the whole password update procedure could be simply described as "reading and updating the <code>/etc/passwd</code> and <code>/etc/shadow</code> file", the shadow library uses piles of code to check identity, permission, and several layers of function calls for encryption, and finally updating files. It needs to consider every aspect of the problem, which makes the code size larger than you  might expect.

Also, passwd libray took [PAM](http://www.wikiwand.com/en/Linux_PAM), [TCB](http://www.wikiwand.com/en/Trusted_computing_base) and [SELinux](http://www.wikiwand.com/en/Security-Enhanced_Linux) into considerations. I would skip these here for I don't yet have time to study all.

A good place to start reading is the <code>main()</code> entry of the <code>passwd.c</code>. The procedures could be summarized as follows:

* __Initialization__:
Init data structures (<code>const struct passwd *pw</code>, <code>const struct spwd *sp</code>, etc.), sanitize environment, check if the user is root, ...;
* __Parse parameters__:
A large switch case for all parameters. As I'm now only interested in updating my password, I would follow the execution path where no parameters are given;
* __Get username, check permissions__:
```c
pw = get_my_pwent();
...
if (!amroot && (pw->pw_uid!=getuid())){...}
...
sp = getspnam(name);
...
check_password(pw, sp);
```
Get username, init pw and sp data structures, check if the user is root or if the user is trying to change his own password. Then it checks the validity of the user's account: is it expired, is its min password change time reached? These are in <code>check_password()</code> function.

* __Get new password__:

```c
if (new_password(pw) != 0){...}
```
Here's where there's most fun. It' when the <code>passwd</code> program prompts you for your old password, and tell you to input your new password. If you fail in trying too many times, the program would get upset and refuses to update password for you.
Under the hood, it also does the following things:
* Encrypt your input with <code>pw_encrypt()</code>(defined in <code>lib/encrypt.c</code>), then compare it with the old encrypted string. There must be a lot of fun to dig into the encryption method, but it's not in the scope of this blog;
* Warns you of weak password;
* Encrypt the password then immediately wipe the cleartext password, saves the encrypted password to the global variable <code>crypt_passwd</code>, which would then copied to other data structures, and then saves to the shadow file.

* __Update shadow file__:

```c
update_shadow();
```
The program warns you the username you are changing password, then it calls the <code>update_shadow()</code> if you have shadow file. Otherwise, it calls <code>update_noshadow()</code>.

The <code>update_shadow()</code> is going to the core of the program, and it's what I will observe closely.

### update_shadow() function

Function <code>update_shadow()</code> is defined in <code>src/passwd.c</code>, and the summary of the procedures is:

* __Set a global lock__:

```c
if (spw_lock() == 0){...}
```
Lock the shadow password file access. Spit an error if it's already locked.

* __Open the shadow file__:

```c
if (spw_open(O_RDWR)==0){...}
```
Taking a deeper look inside the <code>spw_open</code> in <code>lib/shadowio.c</code>, you could find that here is when it opens up the shadow file, reads it, and stores all the entries to the <code>shadow_db</code> doubly linked list.

* __Locate the entry by name__:

```c
sp = spw_locate (name);
```
Also a function call in <code>lib/shadowio.c</code>. The <code>name</code> param is the current username.

* __Create nsp Data Structure__:

```c
nsp = __spw_dup (sp);
```
It copies the content in sp to a new pointer nsp;

*  __Update the encrypted passwd__:

```c
update_crypt_pw(nsp->sp_pwdp);
```

Finally! The <code>crypted_passwd</code> is copied to the data structure, with:
```c
cp=xstrdup(crypt_passwd)
```
inside of <code>update_crypt_pw()</code> function. This process is hidden so deep.
The nsp data structure would then carry this encrypted password to the shadow file. The program would also update metadata, such as the expiration date and so on;

* __Update the shadow\_db Data Structure__:
```c
if (spw_update(nsp) == 0){
    return commonio_update(&shadow_db, (const void *)sp);
}
```
The <code>spw_update()</code>, again is a wrapper for the relating <code>commonio_update()</code>. Inside it would try to find the entry of the <code>shadow_db</code> data structure, or create new entry when not found. Then it saves all the information in the <code>sp</code> to the <code>shadow_db</code>.

* __Close the shadow file, unlock the global lock__:
*```c
spw_close();
...
spw_unlock();
```
Close and saves the shadow file to its place. Unlock the global lock, concludes the whole process. Still, there's much interesting things to look at inside the <code>spw_close()</code> and <code>commonio_close()</code>, but I think I've written long enough.


## Afterthoughts

Reading code is fun, recording the whole process is even more so. It's a rewarding process, especially for some high-quality code as shadow library. It kinda teaches you how top-notch programmers tackles system-level problems. It's also tiring though, when you dig into all the function calls, variables (especially global variables) while tracking its execution path. At some point I really wish the code could be a little bit more commented.

I might have the energy to blog all the code I will read, but I think I will definitely read more code before I start writing something similar. To conclude, it's actually fun experience that quenches your curiosity of "How it actually works".


> Written with [StackEdit](https://stackedit.io/).
