---
description: My write up for the Hack the Box machine Valentine
---

# Valentine

## Box Overview

![](.gitbook/assets/vmplayer_m7pbvysfib.png)

Valentine is a easy rated difficulty linux box with a difficulty rating of 4.1/10 at the time of writing.

In our attempt to root the box we will use heartbleed to get access to the password for a SSH key that we find during our initial enumeration of the box. From there we will use an interesting tmux privilege escalation technique to become root.

## Recon

### Nmap

As usual we will run nmap to get an intial scan of the box:

```text
nmap -sC -sV 10.10.10.79
```

![Nmap scan output](.gitbook/assets/vmplayer_bog38zhd5i.png)

Looking through the results of the scan we see that there are two websites and an ssh port open. We also notice that the ssl-cert is referencing a name `valentine.htb`. We will want to add this to our /etc/hosts file for later.

![added valentine.htb to /etc/hosts](.gitbook/assets/vmplayer_fihaaordqy.png)

Navigating to both the http and the https version of the site gives us just an image

![](.gitbook/assets/vmplayer_onesrjt8rr.png)

With a bit of googling and reverse image searching we discover that the image is associated with a vulnerability called heartbleed.

### What is Heartbleed?

In essence Hearthbleed is a error in logic that allows an attacker to get chucks of random memory that they should not have access to.

Here is a link to the explanation of it by [xkcd](https://xkcd.com/1354/). I think it explains what heartbleed is in a concise and easy to understand way.

## Inital Exploitation

Using the hint provided on the websites, we will use heartbleed on the machine. To do this we will first use a tool called searchsploit which will search the exploit database for any exploits related to heartbleed with this command

```text
searchsploit heartbleed
```

![searchsploit results](.gitbook/assets/vmplayer_lsgdgyjpmm.png)

We get multiple results, and the one we want to use is the bottom one. To download the exploit to the current directory we will use the command:

```text
searchsploit -m exploits/multiple/remote/32745.py
```

![](.gitbook/assets/vmplayer_8ftczejolu.png)

### Running the Exploit

Now to run the exploit we enter the command:

```text
python 32764.py 10.10.10.79
```

![heartbleed output](.gitbook/assets/vmplayer_qvg055qo5y.png)

The output produced is way too large for us to handle easily. An easy way to make the output more manageable woudl be to remove any output that is just 00. We can do this by entering the command:

```text
python 32745.py 10.10.10.79 | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00"
```

![](.gitbook/assets/vmplayer_kkfjaqppce.png)

Much better!

To get the same or a similar output to the screenshot above you may need to run the exploit multiple times. But what we can tell from the output is that there are pages called `/encode.php` and `/decode.php`.And a encoded string `$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==`.

The == at the end of the encoded string tells us that it is most likely a base 64 encoded string. So I went to [cyberchef](https://gchq.github.io/CyberChef/), a website that is great for quickly encoding and decoding basic strings. And our encoding string is decoded to 

![](.gitbook/assets/vmplayer_m5wfp60u3x.png)

At this point we have alot of data but nothing to use it on. So with no clear way to proceed I decided to move on to enumerating the websites.

### Website Enumeration

As usual we will use gobuster to directory bust the website on the machine.

```text
gobuster dir -u http://10.10.10.79 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![gobuster output](.gitbook/assets/vmplayer_cxs49twteg.png)

Looking through the list of directories found the one that seems most interesting to me in `/dev`.

Inside of `/dev` we find two files, hype\_key and notes.txt.

![Notes.txt](.gitbook/assets/vmplayer_gdquofxr52.png)

![hype\_key](.gitbook/assets/vmplayer_ighihkn11t.png)

Notes.txt seems to be a todo list left by the owner of this machine. Nothing too interesting there apart from a hint that there might be something worth looking at in the decoder. This would seem like a hint that the string we found earlier using the heartbleed exploit might be important here.

Whereas hype\_key looks to be encoded so we go over to cyberchef again and use the Hex decoder to get a RSA private key.

```text
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

As it says on the second line, the key is currently encrypted. So we need to decrypt it before we can continue.

To do this we can use the command:

```text
openssl rsa -in hype_key_encrypted -out key_decrypted
```

| Command | Explanation |
| :--- | :--- |
| openssl | uses openssl tool |
| rsa | sets the encryption method |
| -in | the input file |
| -out | the output file |

Then when prompted we need to input the passphrase that we decoded earlier: `heartbleedbelievethehype`.

![key being decrypted using openssl](.gitbook/assets/vmplayer_ktb0rddurd.png)

#### DON'T FORGET

To be able to use the key the permissions to the key file must be set to 600 using the command:

```text
chmod key_decrypted 600
```

Otherwise the key will not work.

### Connecting with SSH

From the key namign scheme it is safe to assume that the user's name for which we have decrypted their ssh key is hype. So now that we have their ssh key and their password we are now able to connect to the machine using ssh. Using the command below:

```text
ssh -i key_decrypted hype@10.10.10.79
```

![](.gitbook/assets/vmplayer_vimqdj4vm6.png)

We can get the file user.txt by moving to the user hype's desktop

![user.txt](.gitbook/assets/vmplayer_ckipmx29y1.png)

## Root PrivEsc

Going through the list of usual privillege escallation commands, we find that `history` returns some interesting commands.

![](.gitbook/assets/vmplayer_wkwunuhc5h.png)

From command 13 onwards was us so we can ignore them. But looking at commands 9 through to 12 we can clearly see that the user has been using tmux. This immediately caught my eye and interest. This is because tmux has some cool ways to priv esc.

By running `tmux ls` we see that there are currently no tmux sessions running. But looking a bit closer we see that the owner of the box has been using come flags with his tmux usage.

| Command | Explanation |
| :--- | :--- |
| -L | sets the socket name |
| -S | sets the socket path |

So with this new found knowledge we can now understand that user went to the .devs folder, started a tmux session with the socket `dev_sess`. Then the user tried to attach to that session with `-a t dev_sess` on line 10. But that is not the correct way to do that. The user then runs the help command to find out how to perform the action he wants to perform, and then on line 12 manages to connect to his tmux session that he crated.

So, by taking a closer look at the socket

![](.gitbook/assets/vmplayer_jgebdcudx7.png)

We can see that the permissions for the socket are that its owned by root, but its group is hype as well as the file being readable by group! This means we can access it.

All this means is that we can recreate what the other user did to get root access.

use the command:

```text
tmux -S /.devs/dev_sess
```

![](.gitbook/assets/vmplayer_qu8n4brj3v.png)

We are now root and can get the root.txt flag.

