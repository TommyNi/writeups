# What do we know?

- Task 1 provides us with information about a login form and a rate limiter. This suggests that we cannot use a standard tool like Hydra to brute force the login form.

- We are also given a zip file to download, which contains two text files:
  - `usernames.txt`
  - `passwords.txt`

  This indicates that the username and password needed to solve this challenge can be found in these files. However, with 800+ usernames and 1500+ passwords it's easy to understand that we cannot do this manually, it's way over 1 million username/password combinations. 
  
- Additionally, we know that the login form is accessible at `http://MACHINE_IP`.

# Examine the login form
Start the challenge machine and also set up your AttackBox by: 
- Start a Web browser (like Firefox) 
- Start Burp Suite (Community Edition is good enough)
- Set the proxy tool in the browser (e.g. FoxyProxy) to go through Burp

With Burp Suite we will more easily be able to view the traffic between the web browser and the web server. 
