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

Press the "Log in"-button about ten times, and you will se that "Too many bad login attempts!" and a Captcha appears. For every login we need to solve a mathematical puzzle. This is the rate limiter that was mentioned in Task 1. We cannot brute force this with Hydra. 

![image](https://user-images.githubusercontent.com/6702854/236621346-f88dfee3-1ba9-428c-8dde-6dcd52157071.png)


Make sure that you try to login with a correct CHAPTA, so we can intercept all kinds of requests from the login form.

# Getting 
