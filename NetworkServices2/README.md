## NETWORK SERVICES 2

**Task 1 Get Connected**

**Task 2 Understanding NFS**

**What is NFS?**

NFS stands for \"Network File System\" and allows a system to share
directories and files with others over a network.

By using NFS, users and programs can access files on remote systems
almost as if they were local files.

**How does NFS work?**

**We don\'t need to understand the technical exchange in too much detail
to be able to exploit NFS effectively- however if this is something that
interests you, I would recommend this resource:**

First, the client will request to mount a directory from a remote host
on a local directory just the same way it can mount a physical device.

The mount service will then act to connect to the relevant mount daemon
using RPC.

The server checks if the user has permission to mount whatever directory
has been requested.

It will then return a file handle which uniquely identifies each file
and directory that is on the server

If someone wants to access a file using NFS, an RPC call is placed to
NFSD (the NFS daemon) on the server. This call takes parameters such as:

> • The file handle
>
> • The name of the file to be accessed
>
> • The user\'s, user ID
>
> • The user\'s group ID

These are used in determining access rights to the specified file. This
is what controls user permissions, I.E read and write of files.

**What runs NFS?**

Using the NFS protocol, you can transfer files between computers running
Windows and other non-Windows operating systems, such as Linux, MacOS or
UNIX.

A computer running Windows Server can act as an NFS file server for
other non-Windows client computers. Likewise, NFS allows a Windows-based
computer running Windows Server to access files stored on a non-Windows
NFS server.

**More Information:**\
Here are some resources that explain the technical implementation, and
working of, NFS in more detail than I have covered here.



What does NFS stand for?\
***Network File System\***

What process allows an NFS client to interact with a remote directory as
though it was a physical device?

Mounting\
What does NFS use to represent files and directories on the server?

file handle\
What protocol does NFS use to communicate between the server and client?

RPC\
What two pieces of user data does the NFS server take as parameters for
controlling user permissions? Format: parameter 1 / parameter 2\
user id / group id\
Can a Windows NFS server share files with a Linux client? (Y/N)\
Y\
Can a Linux NFS server share files with a MacOS client? (Y/N)\
Y\
What is the latest version of NFS? \[released in 2016, but is still up
to date as of 2020\] This will require external research.

4.2

**Task 3 Enumerating NFS**\
**What is Enumeration?**

Enumeration is defined as \"a process which establishes an active
connection to the target hosts to discover potential attack vectors in
the system, and the same can be used for further exploitation of the
system.\" - It is a critical phase when considering how to enumerate and
exploit a remote machine - as the information you will use to inform
your attacks will come from this stage\
**Requirements**\
In order to do a more advanced enumeration of the NFS server, and
shares- we\'re going to need a few tools. The first of which is key to
interacting with any NFS share from your local machine: **nfs-common**.

**NFS-Common**\
It is important to have this package installed on any machine that uses
NFS, either as client or server. It includes programs such\
as: **lockd, statd**, **showmount**, **nfsstat, gssd**, **idmapd** and
**mount.nfs**. Primarily, we are concerned with \"showmount\" and
\"mount.nfs\" as these are going to be most useful to us when it comes
to extracting information from the NFS share. If you\'d like\
more information about this package, feel free to\
read:

You can install **nfs-common** using \"*sudo apt install nfs-common*\",
it is part of the default repositories for most Linux distributions such
as the Kali Remote Machine or AttackBox that is provided to TryHackMe.

**Port Scanning**\
Port scanning has been covered many times before, so I\'ll only cover
the basics that you need for this room here. If you\'d like to learn
more about **nmap** in more detail please have a look at the room.

The first step of enumeration is to conduct a port scan, to find out as
much information as you can about the services, open ports and operating
system of the target machine.

You can go as in-depth as you like on this, however, I suggest using
**nmap** with the **-A** and **-p-** tags.

**Mounting NFS shares**\
Your client's system needs a directory where all the content shared by
the host server in the export folder can be accessed. You can create\
this folder anywhere on your system. Once you\'ve created this mount
point, you can use

the \"mount\" command to connect the NFS share to the mount point on
your machine like so:\
**sudo mount -t nfs IP:share /tmp/mount/ -nolock**\
Let\'s break this down

> **Tag** **Function**
>
> sudo Run as root
>
> mount Execute the mount command
>
> -t nfs Type of device to mount, then specifying that it\'s NFS

IP:share The IP Address of the NFS server, and the name of the share we
wish to mount

> -nolock Specifies not to use NLM locking

Now we understand our tools, let\'s get started!

Answer the questions below\
Conduct a thorough port scan scan of your choosing, how many ports are
open?
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/1.png)

7\
Which port contains the service we\'re looking to enumerate?

2049\
Now, use /usr/sbin/showmount -e \[IP\] to list the NFS shares, what is
the name of the visible share?
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/2.png)

/home\
Time to mount the share to our local machine!

First, use \"*mkdir /tmp/mount*\" to create a directory on your machine
to mount the share to. This is in the /tmp directory- so be aware that
it will be removed on restart.
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/3.png)

Then, use the mount command we broke down earlier to mount the NFS share
to your local machine. Change directory to where you mounted the share-
what is the name of the folder inside?

cappucino\
Have a look inside this directory, look at the files. Looks like we\'re
inside a user\'s home directory\...

no Answer needed\
Interesting! Let\'s do a bit of research now, have a look through the
folders. Which of these folde**rs** could cont**a**in keys that would
give us remote access to the server?

.ssh\

Which of these keys is most useful to us?
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/4.png)
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/5.png)

id_rsa\
Copy this file to a different location your local machine, and change
the permissions to \"600\" using \"chmod 600 \[file\]\".

Assuming we were right about what type of directory this is, we can
pretty easily work out the name of the user this key corresponds to.

Can we log into the machine using *ssh -i \<key-file\>
\<username\>@\<ip\>* ? (Y/N) Y

**Task 4 Exploiting NFS**\
**We\'re done, right?**

Not quite, if you have a low privilege shell on any machine and you
found that a machine has an NFS share you might be able to use that to
escalate privileges, depending on how it is configured.

**What is root_squash?**

By default, on NFS shares- Root Squashing is enabled, and prevents
anyone connecting to the NFS share from having root access to the NFS
volume. Remote root users are assigned a user "nfsnobody" when
connected, which has the least local privileges. Not what we want.
However, if this is turned off, it can allow the creation of SUID bit
files, allowing a remote user root access to the connected system.

**SUID**\
So, what are files with the SUID bit set? Essentially, this means that
the file or files can be run with the permissions of the file(s)
owner/group. In this case, as the super-user. We can leverage this to
get a shell with these privileges!

**Method**\
This sounds complicated, but really- provided you\'re familiar with how
SUID files work, it\'s fairly easy to understand. We\'re able to upload
files to the NFS share, and control the

permissions of these files. We can set the permissions of whatever we
upload, in this case a bash shell executable. We can then log in through
SSH, as we did in the previous task- and execute this executable to gain
a root shell!

**The Executable**\
Due to compatibility reasons, we will obtain the bash executable
directly from the target machine.

With the key obtained in the previous task, we can use SCP with the
command scp -i key_name username@10.10.154.141:/bin/bash
\~/Downloads/bash to download it onto our attacking machine.

Another method to overcome compatibility issues is to obtain a standard
Ubuntu Server 18.04 bash executable, the same as the server\'s- as we
know from our nmap scan. You can download it If you want to download it
via the command line, be careful not to download the github page instead
of the raw script. You can\
use wget https://github.com/polo-\
sec/writing/raw/master/Security%20Challenge%20Walkthroughs/Networks%202/bash.

Note that this method requires an internet connection, so you won\'t be
able to download it when using a free AttackBox.

**Mapped Out Pathway:**\
If this is still hard to follow, here\'s a step by step of the actions
we\'re taking, and how they all tie together to allow us to gain a root
shell:

NFS Access -\>\
Gain Low Privilege Shell -\>\
Upload Bash Executable to the NFS share -\>\
Set SUID Permissions Through NFS Due To Misconfigured Root Squash -\>
Login through SSH -\>\
Execute SUID Bit Bash Executable -\>\
ROOT ACCESS\
Lets do this!

Answer the questions below\
First, change directory to the mount point on your machine, where the
NFS share should still be mounted, and then into the user\'s home
directory.
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/6.png)

No answer needed\
Download the bash executable to your Downloads directory. Then use \"cp\
\~/Downloads/bash .\" to copy the bash executable to the NFS share. The
copied bash shell must be owned by a root user, you can set this using
\"sudo chown root bash\" No answer needed\
Now, we\'re going to add the SUID bit permission to the bash executable
we just copied to the share using \"sudo chmod +\[permission\] bash\".
What letter do we use to set the SUID bit set using chmod?

s\
Let\'s do a sanity check, let\'s check the permissions of the \"bash\"
executable using \"ls -la bash\". What does the permission set look
like? Make sure that it ends with -sr-x.

-rwsr-sr-x\
Now, SSH into the machine as the user. List the directory to make sure
the bash executable is there. Now, the moment of truth. Lets run it with
\"*./bash -p*\". The -p persists the permissions, so that it can run as
root with SUID- as otherwise bash will sometimes drop the permissions.
![alt text](https://github.com/K4R7H1K-D/TryHackMe/blob/main/NetworkServices2/7.png)

no Answer need\
Great! If all\'s gone well you should have a shell as root! What\'s the
root flag?

THM{nfs_got_pwned}
