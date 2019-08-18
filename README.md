# gammalevel
iPhone Development Environment
_______________________________

Often, for some of the trickier packages to build (like Emacs or most Python-based programs), it is easier to have a development environment on the device than to deal with a cross-compiler. The method I outline here is one way to get one. It takes up room, and it’s outdated, but it works for almost anything you’d want to do, if you put up with it.

If you want to install any of the ports on this website, the best way to do that is through my Cydia repo. You do not need this development environment for those packages.

I’m not going to pretend that this is the best way to do this. I developed this method by changing the one I used for OS 1.0 a little at a time when a new OS came out. It’s a bit of a hack right now, but it works. I point out where I think an improvement could be made.

I’m going to assume you have a freshly jailbroken iPhone or iPad. Feel free to skip steps if you think you can.

I’ve tried to write this as accurately as possible, but errors do happen. Feel free to contact me to work things out. Worst case scenario? You have to do a full restore of your device, which is annoying but not hard. Be sure to have backups, but it shouldn’t come to that!

(Also, standard disclaimer: I claim no responsibility if somehow you make your device blow up, or otherwise fail, while using this guide.)

For this guide, I’m going to assume at least passing familiarity with the unix shell. After all, you’re here for a development environment! Above that, I’ll try to explain what I feel is needed to continue.

Finally, this guide was written for iOS 3.2 on the iPad. It shouldn’t be significantly different for similar versions on both the iPhone and iPad, though. Particularly, it seems to still work on a jailbroken iPad running iOS 4.2.1.

Special thanks must go to Børre Ludvigsen, who patiently worked through this guide on his own and helped me iron out most of the bigger issues.

Let’s get started!
Getting Shell Access

More than likely, since you’re here at all, you’ve already done this. It’s included here for completeness. Feel free to skip ahead!

If you’ve just jailbroken, install the OpenSSH package. Also, installing the OpenSSH reconnect helper and Insomnia is a good idea. The helper will reconnect you automatically if you lose your shell connection, and Insomnia keeps that from happening at all. Nothing is more annoying that having to fidget with your device just to keep the Wifi up!

You will also need to install “APT 0.7 Strict” and “Core Utilities” from Cydia. The first will let us easily install Cydia packages from the command line, which is a lot more convenient, and the second is handy for working on the command line.

SSH in to your device with the username root and the password alpine. We’re going to change this now, for security, and then we’ll give the user mobile a password so we can log in without being root.

#iphone:~ root# passwd
#Changing password for root.
#New password: [type a password here]
#Retype new password: [repeat it here]
    #iphone:~ root# passwd mobile
    #Changing password for mobile.
    #New password: [type a different or same password here]
    #Retype new password: [again...]
    #iphone:~ root# 

Getting an Editor

Right now we have no way of editing files. For now, I recommend installing nano. It’s lightweight and easy to wrap your head around. If you’ve never used it before, look around the web for a GNU nano tutorial. If nano‘s not your thing, you could also install vim or even emacs (though for emacs, you need my Cydia repository).

iphone:~ root# apt-get install nano # or 'vim', or 'emacs', or...

(Note: I usually run nano as nano -w, which keeps nano from automatically wrapping lines, which can ruin most configuration files.)
Setting Up sudo

Working as root is bad, very bad. I’ve accidentaly deleted my /usr once, bricking my iPad. However, it gets annoying to have to su into root every time you want to install something. So let’s install sudo!

iphone:~ root# apt-get install sudo

Now we need to edit the sudoers file to make sudo useful. We set the EDITOR environment variable to keep visudo from complaining that vi isn’t there. If you’le using vim, you can leave it out.

iphone:~ root# EDITOR=nano visudo

Right below the line that says “root ALL=(ALL) ALL“, write the similar line “mobile ALL=(ALL) ALL“, then save and exit.

(Note: when you run sudo as mobile, it will ask you for a password sometimes. This is mobile‘s password, not root‘s.)
Logging In as mobile

Before we continue, we’re going to drop our superuser privelege. Log out and log back in as mobile. As a test, make sure sudo is working fine. It’ll give you a nice scary administration notice when you run it the first time, as a bonus!

iphone:~ mobile$ sudo whoami

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

Password: [type in mobile's password]
root
iphone:~ mobile$

The last line saying root is the key: it means sudo is set up right and correctly giving you root access.
Installing GCC and Friends

Finally, we get to the fun part. First off, let’s install some general utilities for our development environment:

iphone:~ mobile$ sudo apt-get install ldid make wget patch gawk

Some explanation: ldid modifies programs to fake iPhone OS into running them like official binaries. make is the standard GNU build tool. We’ll use wget to fetch tarballs from the internet right in the terminal, and then patch to patch them. Autotools, which just about every package uses, needs gawk to run.

Next, we’re going to install gcc itself, but if we just go and do it, we’ll run into a problem with the libgcc offered on Telesphoreo: it refuses to install, because it breaks the system. Instead, we’ll install the dummy package found here, and trick APT into thinking it’s already installed.

iphone:~ mobile$ wget http://gammalevel.com/forever/fake-libgcc_1.0_iphoneos-arm.deb
iphone:~ mobile$ sudo dpkg -i fake-libgcc_1.0_iphoneos-arm.deb

Now we’re prepared to install gcc and some development headers. Note that these headers are compatibility headers, meant to ease the transition from iOS 1.0 to 2.0. That is, they are old. I only trust them for the standard POSIX headers, and even for that they fail in some parts. This is one place that could probably be improved: more on that later.

iphone:~ mobile$ sudo apt-get install iphone-gcc com.bigboss.20toolchain

If you’re anything like me, you’ll be itching to write a little “Hello, world!” program right now and try out your shiny new gcc. Well, you’d be dissappointed. These packages are so old that they need fixing first.
Fixing What’s Broken
Libraries

Unfortunately, you have some downloading to do. Go fetch the iPhone Developer SDK. Once you have it, on a Mac, simply install it. If you’re not on a Mac, you’ll need an archive tool that reads DMGs and Mac packages. I know that 7zip works well on Windows, and probably works fine through Wine on other systems.

We’ll be looking in the iPhoneOS3.2.sdk directory, but by all means change this version number if you need to. On a Mac with the installed official SDK, this is at “/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS3.2.sdk“. If you’re using 7zip, open the Xcode DMG, then the 5.hfs partition, which will take a while. Then, open up “iPhoneSDK/Packages/iPhoneSDKHeadersAndLibs.pkg” for the most recent version, or “.../iPhoneSDKXXX.pkg” for a different version. Inside that package, open Payload, then Payload~, then “.“. The directory we are looking for is then at “Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS3.2.sdk“.

We need to copy libraries from the official SDK on to the device. scp works really well, if you have it: it transfers files over SSH. However, anything that gets files from your computer to your device will work.

(Note for the interested: iPhoneOS is missing libraries to link against, so we’ll be copying them over. Later on, we’ll be editing system headers. Also, none of the frameworks on the device come with headers, so you’ll need to copy those over as you need them. It occurs to me that we might be able to just copy over the entire official SDK, and skip doing this by hand. It might work, but I haven’t tried it. For now, I just copy over whatever’s missing as I run into it. I would like to look into this, though.)

In the directory iPhoneOS3.2.sdk/usr/lib, you will find the following files:

libgcc_s.1.dylib
libSystem.B.dylib
libstdc++.6.0.9.dylib
libiconv.2.dylib

We need to copy these files to mobile‘s home directory, /var/mobile, on the device. Once there, reopen your shell on your device and move them to /usr/lib:

iphone:~ mobile$ sudo mv *.dylib /usr/lib/

We also need to reconstruct a few symlinks:

iphone:~ mobile$ sudo ln -s /usr/lib/libSystem.B.dylib /usr/lib/libSystem.dylib
iphone:~ mobile$ sudo ln -s /usr/lib/libSystem.dylib /usr/lib/libc.dylib
iphone:~ mobile$ sudo ln -s /usr/lib/libSystem.dylib /usr/lib/libm.dylib
iphone:~ mobile$ sudo ln -s /usr/lib/libSystem.dylib /usr/lib/libpthread.dylib

iphone:~ mobile$ sudo ln -s /usr/lib/libstdc++.6.0.9.dylib /usr/lib/libstdc++.6.dylib
iphone:~ mobile$ sudo ln -s /usr/lib/libstdc++.6.dylib /usr/lib/libstdc++.dylib

iphone:~ mobile$ sudo ln -s /usr/lib/libiconv.2.dylib /usr/lib/libiconv.2.4.0.dylib

Headers

We also need to copy over some key C++ headers. In iPhoneOS3.2.sdk/usr/include/c++/4.0.0/arm-apple-darwin9, there is a directory named bits. Copy that directory and all it contains to your device, then install it:

iphone:~ mobile$ sudo mkdir /var/include/c++/4.0.0/arm-apple-darwin9
iphone:~ mobile$ sudo mv bits /var/include/c++/4.0.0/arm-apple-darwin9/
iphone:~ mobile$ sudo ln -s /var/include/c++/4.0.0/arm-apple-darwin9/{,v6}
iphone:~ mobile$ sudo ln -s /var/include/c++/4.0.0/arm-apple-darwin{9,8}
iphone:~ mobile$ sudo ln -s {/var,/usr}/include/c++

Manual Header Modifications

Not only are there critical system libraries missing, but some of the headers are just plain wrong, too. It seems that somewhere along the line, iPhone OS moved from a 32 bit inode to a 64 bit inode, so there are a lot of structures defined in these headers that have the wrong size.

(Note for the interested: Beleive me when I say that hunting down bugs in system headers is a nightmare. Just keep in mind, if you seem to be getting random segfaults for no reason, or some extremely subtle bug that has no reason to exist, it’s probably a system header error I haven’t found yet! Compare the given headers with the ones from Apple, with __DARWIN_64_BIT_INO_T defined.)

First off, we’re going to edit “/private/var/include/sys/stat.h“. Make sure to edit with sudo, so you have write permissions! In struct stat, remove the line that says:

ino_t           st_ino;         /* [XSI] File serial number */

Between the entries for st_nlink and st_uid, add this line:

__uint64_t      st_ino;         /* [XSI] File serial number */

After the entry for st_ctimespec, add this line:

struct timespec st_birthtimespec;       /* time of file creation */

Finally, after the entries for st_ctime and st_ctimensec, add these lines:

time_t          st_birthtime;           /* [XSI] Time of file creation */
long            st_birthtimensec;       /* nsec of file creation */

We’re also going to edit “/private/var/include/sys/dirent.h“. First, we’re going to change the definition of __DARWIN_MAXNAMLEN:

#define __DARWIN_MAXNAMLEN      1023

In the definition of struct dirent, remove the line at the top that says

ino_t d_ino;            /* file number of entry */

In its place, write in:

__uint64_t d_ino;       /* file number of entry */
__uint64_t d_seekoff;

Between the entries for d_reclen and d_type, write in:

__uint16_t d_namlen;    /* length of string in d_name */

Finally, between the entries for d_type and d_name, remove the line that says:

__uint8_t d_namlen;     /* length of string in d_name */

That’s it!
Minor Details

Some configure scripts and Makefiles (like Emacs’s) looks for the C compiler under cc, which is supposed to exist on standard setups. Since we emphatically don’t have a standard setup, we have to make a link.

iphone:~ mobile$ sudo ln -s /usr/bin/gcc /usr/bin/cc

Apparently sometimes the GCC you get won’t search /var/include, which is where all the standard C headers are located. To fix this, add the following lines to ~/.profile:

export C_INCLUDE_PATH=/var/include
export CPLUS_INCLUDE_PATH=/var/include
export OBJC_INCLUDE_PATH=/var/include

Make sure to log out and log back in for these changes to take effect.
Testing your GCC

If you feel so inclined, now would be the time to test out your build environment. GCC works exactly as it does on Mac OS X, that is, exactly like on other systems, but with added options for linking with frameworks. Keep in mind, you may need to sign your programs before they’ll run. I have found that I don’t, but you may need to. If your program crashes for no reason when you start it, you need to run

iphone:~ mobile$ ldid -S program_name

There’s some way to change the system so you don’t ever need to do this, but last I heard there were some long-term side-effects.
You’re Done!

Congratulations!

