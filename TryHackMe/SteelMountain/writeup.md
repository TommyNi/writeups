# Steel Mountain
This is a write up for the TryHackMe room [Steel Mountain](https://tryhackme.com/room/steelmountain).

## Table of Contents
- [Task 1 Introduction](##Task-1-Introduction)
- [Task 2 Initial Access](##Task-2-Initial-Access)
- [Task 3 Privilege Escalation](##Task-3-Privilege-Escalation)
- [Task 4 Access and Escalation Without Metasploit](##-Task-4-Access-and-Escalation-Without-Metasploit)


## Task 1 Introduction


Press "Start Machine" and wait for the machine to boot up. Since it's a Windows machine it may take more than a minute to boot up fully.

Open a Terminal window. For convenience, store the IP address in a local variable, like this:
```
$ TARGET_IP=10.10.57.65
```

### Question: Who is employee of the month?
- Hint: Reverse image search

1. Copy the IP address and paste it into your web browser on your attack box or your own VPN-connected machine. You shold see the page shown below:
   <br/><img src="startpage.png" width="300px" >

2. Right click and "View Page Source". 

3. Find the source of the img tag to find the answer.
    ```
    // snip //
    <h3>Employee of the month</h3>
    <img src="/img/<redacted>.png" style="width:200px;height:200px;"/>
    </center>
    // snip //
    ```

## Task 2 Initial Access

### Question: Scan the machine with nmap. What is the other port running a web server on?

1. In the previously opened Terminal Windows, type
   ```
    $ nmap -sV $TARGET_IP
   ```
   The -sV option will give us versions of found services.

   Which will give the following reponse
   ```
    Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-05 18:32 EDT
    Nmap scan report for 10.10.57.65
    Host is up (0.048s latency).
    Not shown: 989 closed tcp ports (conn-refused)
    PORT      STATE SERVICE            VERSION
    80/tcp    open  http               Microsoft IIS httpd 8.5
    135/tcp   open  msrpc              Microsoft Windows RPC
    139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
    3389/tcp  open  ssl/ms-wbt-server?
    8080/tcp  open  http               HttpFileServer httpd 2.3
    49152/tcp open  msrpc              Microsoft Windows RPC
    49153/tcp open  msrpc              Microsoft Windows RPC
    49154/tcp open  msrpc              Microsoft Windows RPC
    49155/tcp open  msrpc              Microsoft Windows RPC
    49156/tcp open  msrpc              Microsoft Windows RPC
    Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 72.84 seconds
    ```
    2. As we can see there is also another port 8080 using the http service, under the SERVICE column. This is the answer.

### QUESTION: Take a look at the other web server. What file server is running?
    
1. As we can see from the nmap output above port 8080 uses "HttpFileServer httpd 2.3". 

2. Go to http://TARGET_IP:8080
    <br/><img src="otherhttpPort.png" width="400px" > 
    <br>
    to view the page. There is also a confirmation of the "HttpFileServer 2.3" service being used.

3. Search for "HttpFileServer httpd 2.3" in your favorite search tool. You should find the full name "Rejetto HTTP File Server". 

### QUESTION: What is the CVE number to exploit this file server?

1. One of the highest search hits is probably https://www.exploit-db.com/exploits/39161. This specific version of Rejetto File Server has a Remote Code Execution (RCE) vulnerability!
     <br/> 
    <img src="cve.png" width="700px" > 

### QUESTION: Use Metasploit to get an initial shell. What is the user flag?


## Task 3 Privilege Escalation

## Task 4 Access and Escalation Without Metasploit


