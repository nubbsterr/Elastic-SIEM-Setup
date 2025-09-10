> [!CAUTION]
> This guide WILL be getting revamped very soon to run off a simple Docker Desktop deployment method, which will DRASTICALLY downsize this guide and allow us to immediately get into DFIR simulations! See documentation [here](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/local-development-installation-quickstart).

# Elastic-SIEM-Setup
A guide for building your own SIEM using the Elasticsearch, Beats, and Kibana, and how to simulate, analyze and triage attacks for a homelab. Courtesy of the internet and other sources.

> [!IMPORTANT]
> You can use VMware to set up this entire lab. I would actually recommend it if you have access to the software since VirtualBox Shared Clipboard doesn't work even with Guest Additions installed, from my experience at least. Your next best option is using SSH so you can copy commands from your host "onto" the VM.

<strong>Big</strong> thank you to all of the lovely sources on the internet. I am not one to shy away from reading documentation but this kind of project is virutally impossible for me given my limited knowledge of the Elastic services.

This guide is aimed to be very step-by-step oriented; my goal is to guide you through everything as well as I can. If you do have questions, feel free to message me on Discord (nubbieeee) or email me (sherm5344@gmail.com)! Also massive shoutout to [crin](https://www.youtube.com/@basedtutorials/videos) for getting <strong>me</strong> properly started in cybersec. Without him, I would probably be aimlessly doing programming projects or frying my ESP32. 

Big thanks to the following people as well:
- Lavender 💜 for a lot of red team knowledge behind the scenes regarding C2s and maldev.
- Chroma for also assisting me in red team but more so in blue team with understanding SOC; you helped me lots with this project with constructing the incidents.
- Eurofighter for loads of red team knowledge, mainly having to do with maldev. You are the reason why I got into researching red team!!

# Some Tips Before Diving In
Please check out the <strong>[Sources section](#sources)</strong> of this guide for any important documentation as needed. Other bits of documentation are hyperlinked throughout the guide and are optional to read but highly recommended for your own understanding.

Also, **please** consult the Q&A section below if you ever have questions as you go through the guide!

Lastly, I do expect some amount of competence with a Linux terminal for this project. We will be using SSH, lots of `sudo` and command-line text editors; I expect you to be comfortable with common Linux commands like `cd`, `mkdir`, `systemctl`, etc. Naturally, I will guide you as much as I can, but if anything is confusing you, it can be: 
1. Voiced to me. Yes, you can message me on Discord or via email with questions. 
2. Googled and/or understood with AI. I won't shove it down your throat, but ChatGippity is marvelous for these roles of teaching the unknown. Of course, it is advised to do your own research and pay attention to the AI's responses for hallucination(s) so you don't fool yourself.

With that being said, if you would like to contribute to this repo, feel free to message me or make a PR to the repo!

# What You Will Learn!
By the end of this project, <strong>you are going to know a lot of stuff</strong>; you'll have basically done a fair portion of what a SOC analyst does in their like:
- How to use VirtualBox to create VMs and configure networks and port forwarding
- Understand the workings of a SIEM, from ingest to visuallizing the data.
- Understand how Elasticsearch, Beats and Kibana work together to create a SIEM.
- Understand what a [CA](https://en.wikipedia.org/wiki/Certificate_authority) is, the signing process, and how SSL certificates function.
- Establish simulated attacks in a controlled environment and employ root cause analysis.
- Understand how Active Directory works at a very basic level with Domains, DCs, DAs, etc.
- Understanding how Atomic Red Team tests can be used for defensive purposes to enhance security postures.
- Understand [MITRE ATT&CK](https://attack.mitre.org/) and using it to show TTPs of an attack.
- Understand the [Kill Chain framework](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html); understand the attacker POV.
- Understand incident response measures for triaging attacks.

# Q&A
> [!IMPORTANT]
> Below is a list of some questions I have answered preemptively to save time for both me and **you**:

> [!TIP]
> Q: *How can I reach you?* A: DM me on Discord (nubbbieeee)!

> [!TIP] 
> Q: *How do I stop all my services?* A: **They will stop on their own when you shutdown your VM,** there is no need to stop them manually. if anything, it may lead to goofing things up.

> [!TIP] 
> Q: *Why aren't you using an updated Ubuntu Server version?* A: In all honesty, I was following the [LevelEffect](https://www.leveleffect.com/blog/how-to-set-up-your-own-home-lab-with-elk) from the beginning and have been using it as reference. It too used Ubuntu Server 18.04 LTS, so I followed suit. You can probably get away with installing a newer OS install and save a snapshot before doing so!

> [!TIP] 
> Q: *Why use this guide when others exist?* A: The rest of them, if any exist, **suck**. Most 'ELK setup' guides use Elastic Cloud, which is totally fine, but hosting everything and going step-by-step just makes you that much more informed and knowledgable. On top of me providing DOCUMENTATION for you to read AS YOU WISH. I don't know of many guides that do that. And no, those Arduino project guides don't count because they all copy each other line for line, letter for letter.

> [!TIP]
> Q: *Why is this so long!!!!!?????*  A: **I go into a lot of detail + this is what real learning is.** Taking the time to put in the effort to reap the fruits of your labour, as I have throughout writing this guide.

> [!TIP]
> Q: *`xyz` isn't documented in your guide! Why is that?* A: Either I 1) Missed it completely while writing it or 2) Purposefully ignored it because it was either self-explanatory or way too lengthy to explain. In the event that I do miss something that you believe is important to be documented, you can DM me on Discord (nubbieeee).

# The Setup Begins
This is our first step into our SIEM-building journey. 

First things first, we need a VM to host our services. We'll be using VirtualBox for just that. Because I am such a nice kiddo, <strong>I have a bash script made to install VirtualBox 7.1 for Ubuntu linked on this repo :)</strong> [Sources](#sources) are linked below specifically for this setup.

Once VirtualBox is set up, we can install an ISO for our VM; preferably Linux. I am running an [Ubuntu Server VM](https://releases.ubuntu.com/bionic/). 

To set up the VM:
1. Open VirtualBox and Select `Machine` then `New`.
2. Give your VM a cool name and select the previously installed ISO image as your chosen OS.
3. Check the box to skip the unattented install. This is so we can tinker everything ourselves.
4. Adjust virtual hardware settings. I have my VM running 4GB of RAM, 2 CPU cores and ~50GB of SSD storage.
5. Click Finish and complete your VM setup.

> [!IMPORTANT] 
> Before starting our VM, we will connect it to a NAT network in preparation to join other machines to the network so we can deploy Agents to them. In VirtualBox, hit `File` --> `Tools` --> `Network Manager`. Click `NAT Networks` and hit `Create`. Afterwards, go to `Settings` on our newly made server VM. Go to `Network` --> `Attached To` --> `NAT Network`. Our newly made network should be immediately selected! From here, we can continue on!

6. Start up your VM once everything is ready!

To set up the OS on our VM:
1. Select your language accordingly. ![Select a Language](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/select-language.png)
2. If you get an "installer update" notice, you can skip it as you wish.
3. <strong>Record the IP shown next to DHCPv4, we will need this!</strong>
4. We don't need a proxy, so we can skip configuring one. We also don't need an alternative mirror for Ubuntu, so leave everything there as is.
5. Continue setting up the storage config for the OS, there is no need to tinker with anything unless <strong>you</strong> want to :) ![Storage Config](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/storage-config.png)
6. Confirm your user credentials and SSH setup, tick the OpenSSH Server Box. We don't need any featured software on our VM, so continue with the installation. ![Inputting user credentials, you can change yours to your liking!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/profile-setup.png)
7. Wait for the OS install to complete :) ![waiting](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/waiting-for-install.png)

# Installing and Configuring Elasticsearch
Before we even get to starting anything, go into your VirtualBox menu and select the 3 bars next to your VM, then select `Snapshots`. Click `Take` and name the snapshot however you like. Congratulations, you have just implemented a failsafe in case you break your VM somehow. <strong>I strongly recommend you regularly take snapshots of your VM when you reach checkpoints in this guide.</strong>

Once you reboot your system, wait for your log statements to stop, then press Enter. Continue to logging in as your user. When you're in, your screen should look something like this:
![Voila](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/login-complete.png)

The last thing we need to do before getting to real SIEM setup fun is to update our system with the following commands:
```
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt install zip unzip jq -y
```
From here on out, I will be using SSH to access our server. However, it is totally fine to do everything on the VM manually.

# Intermission: "SSH no worko!"
If you're like me and SOMEHOW don't have ssh enabled by default, you can and should do the following:
1. Run `sudo systemctl status ssh` to see if SSH is even live on your system. 
2. If you see an error entailing `ssh.service` couldn't be found. Don't panic. We will simply reinstall OpenSSH so we can continue.
3. Run the following command: `sudo apt-get remove --purge openssh-server&& sudo apt-get update && sudo apt-get install openssh-server`.
4. If all is well, you can run `sudo systemctl status ssh` and see that SSH is currently a dead process. Run `sudo systemctl start ssh` to start it up! You can also run `sudo systemctl enable ssh` to just have it be a startup process.

Because I am such a nice guy, I have left a script on this repo that can do all of this for you provided you are on an Ubuntu/Debian system! 

# Intermission: More SSH Setup
This step is optional for setting up SSH. If we want to SSH onto our VM, we'll require a port forwarding rule such that TCP traffic from the host machine is directed to the VM at port 22 (SSH port).

Go to `File` --> `Tools` --> `Network Manager` --> `NAT Networks` --> `Port Forwarding`. Click the small green button and add the following properties to the rule:
- Name: [Whatever name you want]
- Protocol: TCP
- Host IP: 127.0.01
- Host Port: [any port to specify with SSH]
- Guest IP: [Your server's IP, check with `ip addr`]
- Guest Port: 22

With this rule, we can SSH from our host machine onto the VM using `ssh -p [IP set for host port] [server username]@[server hostname]`. 

# Continuing to setup Elasticsearch
<strong>The horrors persist but so do we, let's continue!</strong> If your VM is not running right now, go and start it up. We'll SSH in just like before and get straight to business.

Firstly, we will be installing `apt-transport-https`, which on its own enables APT transport to be done w/ HTTPS and not HTTP; slightly more secure stuff for our VM which could prove useful for production environments.

Run `sudo apt install apt-transport-https -y` and continue. Next we add the GPG key for Elasticsearch and its repo to our Ubuntu sources.

Run `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -` to add the GPG key and `echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list` to see the repo in our ubuntu sources. 

![GPG and repo added!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/gpg-repo-success.png)

Run `sudo apt update` to update our package lists and now we are ready to install the Elasticsearch service. Run `sudo apt install elasticsearch`.

Now that Elasticsearch is installed, we need to reload all running daemons and services on our VM. Since we are adding a new service (Elasticsearch), this is necessary. It wasn't needed for SSH since we changed nothing with configuration files, and configuration is already handled by apt (to my knowledge) but for Elasticsearch, we will be modifying our configuration files. Run `sudo systemctl daemon-reload` to do so. 

We will want to confirm that Elasticsearch is actually on our system, so run `sudo systemctl status elasticsearch.service`; we want to see a 'dead' process.

![Yes I did a typo.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elasticsearch-success.png)

With that all said and done, we are officially finished the installation phase for Elasticsearch and will now get to configuring it.

# Configuring Elasticsearch and Testing It Out
This step is entirely for configuring Elasticsearch; telling it how to run, what ports it needs, communication, etc.

Open `/etc/elasticsearch/elasticsearch.yml` in vim using `sudo vim /etc/elasticsearch/elasticsearch.yml`. You may use any editor of your choice. All I will say about Vim keybinds is that `ESC+Colon+q!` will forcefully exit and not save the file,`i` will enter insert mode where you can edit text, and `ESC` will take you out of any mode and put you into normal mode where you can traverse the file as you wish.  Firstly, we will edit the `cluster.name` property to be the name of your choice for your homelab. 
![Delete those pesky pound symbols.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/edit-elasticsearch-config.png)

Continue editing the file. The `network.host` property will be our VM's IP address. The `http.port` property will be left alone, **but make sure to uncomment it**, and lastly, add the `discovery.type: single-node` property (this property tells Elasticsearch how it should form a cluster). **Do not forget the colon next to the property name!** 

Save and exit then run `sudo systemctl start elasticsearch` to start the service. Wait a lil for the command to complete. We can also run `sudo systemctl enable elasticsearch` to automatically start the service on boot of our VM, **which is recommended.** Once the command completes, run `curl --get http://NETWORK_HOST_IP:9200`. NETWORK_HOST_IP is the network.host IP we have in our config file!
![We got something!!!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elasticsearch-curl-success.png)

Congratulations, we just set up Elasticsearch and can confirm that a cluster is up and running. Port 9200 is where we will send all of our logged data to; from Beats or Elastic Agents if you go down that route. Our next step is setting up Kibana in the same manner.

If you ever want to edit the `elasticsearch.yml` configuration on your own time, **you will need to run `sudo systemctl restart elasticsearch.service` so that the updated configuarion is read by Elasticsearch.**

# Configuring Kibana
Configuring Kibana starts with installing it; run `sudo apt install kibana`.

Next we will update the YML configuration of Kibana just like we did with Elasticsearch. In my case of running Vim as my editor, run `sudo vim /etc/kibana/kibana.yml`.

Uncomment the `server.port` property as port 5601 is the default option and we have no need to edit it. We will edit the `server.host` and `server.name` properties to be our IP address and an appropriate name respectively. One thing you may notice is that you can adjust the credentials for accessing the Kibana server. You can leave them blank for now, as we will change them later on.
![Our edits.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/kibana-config.png)
![Default credentials are a big nono!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/kibana-nono-security.png)

Save and exit our changes to the YML file and go ahead and start up the Kibana service. If you're like me and have Elasticsearch not enabled in systemd, you will need to start it up in the following order:
1. Start Elasticsearch w/ `sudo systemctl start elasticsearch` if not already started.
2. Start Kibana w/ `sudo systemctl start kibana`. We can also make Kibana run on boot w/ `sudo systemctl enable kibana`.

Once both Elasticsearch and Kibana are running, we will go ahead and check the status of both services by running `sudo systemctl status elasticsearch && sudo systemctl status kibana`.
![Both services running just fine.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-kibana-success.png)

That "legacy OpenSSL" message thing is not an issue. According to ChatGPT (yes you heard that right), this is simply for backwards compatibility measures involving communication with Kibana's Node.js runtime. Basically, we do not care in the slightest. 

Kibana and Elasticsearch are now up and running and all that's left is Beats for ingestion. To quickly go over Beats, we will actually be using **filebeat**, which is a kind of Beat offered by Beats (wowza). [Here is a link for further reading on other beats you can use.](https://www.objectrocket.com/resource/what-are-elasticsearch-beats/) For our purposes, filebeat will be just fine to get going and testing our environment. However, I will personally guide you through installing **Winlogbeat** to create our very own mock incidents inspired by **chroma** on crin's server.

# Configuring Filebeat
Install filebeat by running `sudo apt install filebeat`. As seen withboth Kibana and Elasticsearch, we will need to edit filebeat's YML file; `/etc/filebeat/filebeat.yml`, which I will go ahead and open with vim as I have been doing thus far.

Configure the Elasticsearch portion of Filebeat. Kibana's config is totally fine. All we need to do is change the IP of the Elasticsearch host.

![Elasticsearch-Filebeat configuration.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-filebeat-config.png)

That is all we need to do. Our next step is to literally set up Filebeat in the terminal. This will manage how index management w/ Elasticsearch works, on top of disabling Logstash for log ingest.

Run `sudo filebeat setup --index-management -E output.logstash.enabled=false 'output.elasticsearch.hosts=["ELASTICSEARCH_HOST_IP_ADDRESS:9200"]'`. ELASTICSEARCH_HOST_IP_ADDRESS is the host IP address we chose in our elasticsearch.yml file. Index management manages how Elasticsearch goes about its indexing business, which I will not go over since that'd take 10 krillion years. `-E` overwrites specifc config settings, which we do next by 1) Disabling Logstash and 2) Ensuring we have our Elasticsearch host correctly set because why not.

Once that's over we can actually start everything up in the following order if NOT ALREADY STARTED:
1. Start Elasticsearch: `sudo systemctl start elasticsearch`
2. Start Filebeat: `sudo systemctl start filebeat`
3. Start Kibana: `sudo systemctl start kibana`
![Everything looks good!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/all-services-running.png)

Assuming nothing blew up, we can now run one last `curl` to actually check if Elasticsearch is getting something from Filebeat. Run `curl --get http://ELASTICSEARCH_HOST_IP_ADDRESS:9200/_cat/indices?v`. This command may look insane, but it's relatively simple:
1. `_cat/indices` is the CAT Indices API. Simply put, it is an API that returns high level information about indices in an Elasticsearch cluster. We are making an API call with this URL.
2. `?v` = verbose. Gives column headers to output and makes it more readable. 

Now if you're a complete dumby like me, and forgot to run `sudo filebeat setup`, then we're DOOMED! Not. Just shutdown filebeat, run the setup command, and restart filebeat with the following order:
1. `sudo systemctl stop filebeat.service`, <strong>this took a while so do NOT Ctrl+C and possibly ruin your day.</strong>
2. `sudo filebeat setup --index-management -E output.logstash.enabled=false 'output.elasticsearch.hosts=["ELASTICSEARCH_HOST_IP_ADDRESS:9200"]'`
3. `sudo systemctl restart filebeat.service` && `sudo systemctl status filebeat.service`
![IT WORKS!???????????????!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/filebeat-success.png)

That yellow healthcheck notice is nothing to worry about. To my knowledge, it is indicating that replica shards are unassigned, which basically means we are at risk of data loss, which we don't care about in our current state. No only that, we have no primary shards, since we have no indexed data, so <strong>we do not care.</strong>

# Establishing Elastic Services IPs and Understanding Certificate Authorities
Our first step with making a CA is to understand why we are doing it. We are making a CA so we can implement [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security), thus HTTPS into our Elastic services, and that requires certificates.

Firstly, we need to specify the instances of our services so we can later grant them a certificate. Navigate to `/usr/share/elasticsearch` and create a file called `instances.yml` by running `sudo touch instances.yml` and then opening it in a text editor. This file will hold information regarding what Elastic components we are using and their host IPs.
![Creating the file in the directory.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/creating-instances-yml.png)

Great stuff, let's add the following text that I will not copy because it is too tiresome:
![instances.yml configuration.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/instances-yml-config.png)

Replace the `10.0.2.15` with your appropriate host IP. You may be wondering what that `fleet` service is; it is for managing your fleet of Elastic Agents, which themselves collect all sorts of information past log data. Simply put, they are fundamental if you wish to set up Elastic Agents later down the line.

Save the file and exit our text editor. Our next step is to literally create a PKI (Public Key Infrastructure) for ourselves. To do so, we will need keys and certificates, and of course, a certificate authority, which we will make using Elasticsearch's utilities.

# Creating the CA and Generating Certificates
Navigate to `/usr/share/elasticsearch` and run the following: `sudo bin/elasticsearch-certutil ca --pem`. Elasticsearch-certutil will aid use with creating basic [X.509](https://www.techtarget.com/searchsecurity/definition/X509-certificate) certificates as well as signing them. The `ca --pem` bit will 
1) Create a new CA
2) Create the CA certificate and private key in PEM format, as the command output will say. 
![Running the command and seeing the output!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-certutil-success.png)

Awesome sauce. We now have a zip file of our our CA certificate and private key. Unzip the zip folder with `sudo unzip ./elastic-stack-ca.zip`. You should have a `ca/` directory with a certificate file and private key. Our next step is to generate the certificate to sign for each of our instances.

Generate the certificate with the following command: `sudo bin/elasticsearch-certutil cert --ca-cert ca/ca.crt --ca-key ca/ca.key --pem --in instances.yml --out certs.zip`. Let me make this shrimple to understand:
1. Provide the CA key and certificate to be included in each certificate and key respectively (shared secrets and for signing the certificates).
2. Everything is set to the PEM format, given the `--pem` option.
3. `--in instances.yml` is going to effectively use each instance specified in `instances.yml` and generate its certificate and private key.
4. Output the final compressed zip with the name `certs.zip`.
![Creating our certificates for each instance.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/creating-instances-certificates.png)

The command output on its own does a great job giving a ton of documentation, which is awesome. We'll unzip the zip folder and then make a new directory `certs` solely to organize our goodies. Run `sudo unzip certs.zip` then `sudo mkdir certs`. We'll be organizing:

1. `sudo mv elasticsearch/* certs/`
2. `sudo mv kibana/* certs/`
3. `sudo mv fleet/* certs/`

Next, we will create directories to store the CA certificate and private key for each of our services:
1. `sudo mkdir -p /etc/elasticsearch/certs/ca`
2. `sudo mkdir -p /etc/kibana/certs/ca`
3. `sudo mkdir -p /etc/fleet/certs/ca`

Then we copy the private key and CA certificate to said directories:
1. `sudo cp ca/ca.* /etc/elasticsearch/certs/ca`
2. `sudo cp ca/ca.* /etc/kibana/certs/ca`
3. `sudo cp ca/ca.* /etc/fleet/certs/ca`

And finally, copy the service certificates and keys we generated in the beginning to each of their `certs` directory:
1. `sudo cp certs/elasticsearch.* /etc/elasticsearch/certs`
2. `sudo cp certs/kibana.* /etc/kibana/certs`
3. `sudo cp certs/fleet.* /etc/fleet/certs`

Last thing to do before moving on is to clean up our now empty directories as we have moved their contents to the above new folders: `sudo rm -r elasticsearch/ kibana/ fleet/`. 

# Intermission: Proper Security for SIEM Deployment
This step is all about applying least privilege to our Elasticsearch certificates and keys as a layer of security in our deployment.

We'll begin by navigating to `/usr/share` and run the following commands:
1. `sudo chown -R elasticsearch:elasticsearch elasticsearch/`, which will 1) Recursively set the ownership of the `elasticsearch` directory to Elasticsearch and 2) Change its group to the `elasticsearch` group, which will further allow us to limit the scope of Elasticsearch's permissions.
2. `sudo chown -R elasticsearch:elasticsearch /etc/elasticsearch/certs/ca`. Same stuff as above but now for Elasticsearch's CA copies, so it can access that directory as well.

**Please do  NOT do this `chown` privilege restriction to Kibana's directories FOR ANY REASON. Kibana needs access to its CA files to serve the SIEM frontend in your web browser. By restricting access, Elasticsearch will not be able to retrieve such files and our deployment will be broken.**

Now, to ensure our posture is correct and our certificates are valid, we will use the `openssl` command to print certificate information to the console with the following command: `sudo openssl x509 -in /etc/elasticsearch/certs/elasticsearch.crt -text -noout`, which will: 
1. Print information for X509 certificates.
2. `-in` specifies the certificate to check out
3. `-text`  ouputs everything as human-readable.
4. `-noout` suppresses the output of the encoded version of the certificate; only shows the human-readbale output above.
![Success.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-chown-success.png)

# Configuring The Services To Run HTTTPS w/ SSL Certificates
Now we are effectively ready to put everything together, and enable SSL on all of our services, thereby enabling HTTPS. 

Our first step is to go configure the `kibana.yml` configuration file in `/etc/kibana/` by running `sudo vim /etc/kibana/kibana.yml`, or with your preferred text editor, where we will copy the following massive text wall and paste it to the <strong>bottom</strong> of our YML file, or edit the values in the YML file accordingly:

```
server.ssl.enabled: true
server.ssl.certificate: "/etc/kibana/certs/kibana.crt"
server.ssl.key: "/etc/kibana/certs/kibana.key"

elasticsearch.hosts: ["https://10.0.2.15:9200"]
elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/ca/ca.crt"]
elasticsearch.ssl.certificate: "/etc/kibana/certs/kibana.crt"
elasticsearch.ssl.key: "/etc/kibana/certs/kibana.key"

server.publicBaseUrl: "https://10.0.2.15:5601"

xpack.security.enabled: true
xpack.security.session.idleTimeout: "30m"
xpack.encryptedSavedObjects:
    encryptionKey: "min-32-byte-long-strong-encryption-key"
```

Some of these properties are very self explanatory and others are not the case. The `server.publicBaseUrl` property is our IP and Kibana port number (5601), which specifies where Kibana is available; basically our SIEM UI. The `xpack` bits are all for securing Elasticsearch. The `xpack.encryptedSavedObjects.encryptionKey` property is for creating an encryption key to encrypt and decrypt sensitive Kibana entities like dashboards, alerts, etc. We use the default specified by [Elastic](https://www.elastic.co/guide/en/kibana/current/xpack-security-secure-saved-objects.html), but if you guys find docs on this, PLEASE send them to me because I cannot find many examples for configuring this property.

The rather confusing matter in all of this is specifying the `elasticsearch.ssl.key/certificate` properties with our Kibana key/certificate. Initially, I was very confused on why we (the LevelEffect Guide linked in Sources) was using the Kibana certificate and key here, but the answer is LITERALLY in the YML file comments. We are SENDING the Kibana certificate and key to Elasticsearch to verify our identity as Kibana. That is it.

Anyways, remember to replace the `10.0.2.15` with your host IPs from before. Save and exit, then open the `elasticsearch.yml` configuration file in `/etc/elasticsearch/` as we did with our `kibana.yml` file and paste the following to the bottom of your YML file:

```
xpack.security.enabled: true
xpack.security.authc.api_key.enabled: true

xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
xpack.security.transport.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
xpack.security.transport.ssl.certificate_authorities: ["/etc/elasticsearch/certs/ca/ca.crt"]

xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
xpack.security.http.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
xpack.security.http.ssl.certificate_authorities: ["/etc/elasticsearch/certs/ca/ca.crt"]
```

For the sake of length and explanation, I will leave [a link to documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#token-service-settings) for you to read. The summary for these settings is that we are enabling a gajillion security features and settings for our setup, particularly to use SSL.

You will 110% notice that we have no quotation marks with each filepath here, and that is, in fact, the correct method to display them here. I believe this is simply a matter of parsing behind the scenes, but it is undeniably a <strong>terrible one.</strong> The `certificate_authorities` list needing quotation marks makes sense, but wadahek Elastic? Were the devs chugging Red Bull while making the backend for this?

Save and exit as usual, and we are ready to load our new configurations to officially enable SSL for our services. If you recall from before, we need to restart our services to enable new configurations by running `sudo systemctl restart <service_name>`. `restart` will both start the system if it is not running and restart it, so we can go ahead and do just that for both Elasticsearch and Kibana:

1. `sudo systemctl restart elasticsearch`
2. `sudo systemctl restart kibana`

Run `sudo systemctl status <elasticsearch or kibana>` if you wanna check the status of either service, though we can be certain they are running at the moment. If anything is amiss, let is be known that the logs for Elasticsearch and Kibana are stored in `/var/log/[elasticsearch or kibana]/[home-lab.log or kibana.log]` respectively.

We don't need `filebeat` running for this next step. Our last check to ensure everything is in order is to `curl` the HTTPS host for Elasticsearch, which in my case is: `curl --get https://10.0.2.15:9200`. Replace your IP as needed, anddd....
![Erm... What?](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/curl-fail.png)

Before you call me a fraud, let me explain! The reason why `curl` failed like this is because our browser does not trust the certificate just yet, or rather, the CA. We can bypass this temporarily and just add the `--insecure` parameter to end of our `curl` command and pipe the output to `jq` to make it nice and pretty as JSON format: `curl --get https://10.0.2.15:9200 --insecure | jq`.
![Yippee!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/curl-success.png)

Nice, we at least got something! To summarize this output, we require credentials to log into Elasticsearch, which we will generate in the next step of our guide. But for now, go ahead and pat yourself on the back; we've got SSL running and security enabled! One step closer to a real homelab setup. We can stop our services by running `sudo systemctl stop kibana` then `sudo systemctl stop elasticsearch`. 

# Generating Authentication Credentials For Elasticsearch
To generate credentials, we will use the [elasticsearch-setup-passwords](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-passwords.html) utility to do so. Documentation is hyperlinked as always for your reading. Simply put, the utility will generate passwords for a cluster and connect via HTTPS using our previously defined `xpack.security.http.ssl` in our `elasticsearch.yml` file. 

Run `sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto` to randomly generate passwords, set the passwords of select users, and output them to the console. Make sure Elasticsearch is running before you do this or else you will get an error. You will see many passwords and names associated with them. For example, `apm_system` is the [APM or Application Performance Montioring System](https://www.elastic.co/guide/en/observability/current/apm.html), which, per its name, collects performance metrics for services and applications running in real-time, and is built on the Elastic Stack. More information about these users can be collected here by [Elastic's official documentation.](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html)

We only need the `kibana_system` password as Kibana authenticates through Elasticsearch and thus grants access to the frontend UI. <strong>However,</strong> we will need to create our own user with prvilieges to login to the SIEM frontend after we initially login using the <strong>`elastic` superuser</strong>. It's credentials will be `elastic` as its username and `<generated_password>` from the previous step when we generated new passwords.

Open the `kibana.yml` file in `/etc/kibana/` with a text editor and find the `elasticsearch.username` and `elasticsearch.password` properties. Uncomment both properties and change the `elasticsearch.password` to the password we just generated and <strong>DO NOT CHANGE THE `elasticsearch.username` PROPERTY. You will get error logs back when the server attempts to launch to host the frontend!</strong>

Save and exit our YML config file and restart Kibana so that our new configuration settings are read properly with `sudo systemctl restart kibana && sudo systemctl restart elasticsearch`.

# Logging into Elastic
Firstly, start up all of our services if you haven't already with `sudo systemctl start elasticsearch`, `sudo systemctl start kibana` and `sudo systemctl start filebeat`. The fun part starts now.

At the moment, we will not be able to access our frontend on a web browser without another port forwarding rule. As we did before when setting up SSH, go create another port forwarding rule, with the following parameters: 
- Protocol: TCP
- Host IP: 127.0.0.1 
- Host Port: 5601
- Guest IP: [kibana IP, which should be your server IP]
- Guest Port: 5601

**Once you've created the rule, you can attempt to head to your frontend in your web browser at `https://127.0.0.1:5601`** if your VM is powered on and your services are running.
![Waiting....](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/frontend-loading.png)

The loading may take a while; just be patient...
![A FRONTEND????????????](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/frontend-loaded.png)

Now we login using our <strong>`elastic` superuser credentials</strong> and NOT the `kibana_system` user.
![IT WORKED!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-login-success.png)

# Intermission: Creating Our Own User
Click the `Explore on my own` button to continue to the Elastic UI. Once we're in, our sole goal is to create an admin user for ourself, so that we don't need to log in to the `elastic` superuser every time.

Given the steps in the [Official Elastic Docs](https://www.elastic.co/guide/en/kibana/8.17/using-kibana-with-security.html), we will go to the Roles management page and create a new user with the `kibana_admin` role; to grant us admin access to the Kibana; by extension, the SIEM frontend.

Scroll down and click the `Manage Permissions` button. On the sidebar to the left, Click `Users` under `Security`. This is where we want to be.
![User page!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/users-page-success.png)

Click the `Create User` button, and set up the credentials as needed. <strong>For privileges, select the following roles:</strong>
- beats_admin
- kibana_admin
- superuser, to access Fleet Management

More information about privileges can be found [here in the Elastic Docs!](https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-roles.html) To ensure that our account actually works, we will attempt to login to it on the Elastic login page. Logout then log back in using our new user.
![Nice!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/new-user-success.png)

If we ever need to modify our account privileges, we will simply login using the `elastic` superuser again and modify our roles as needed.

# Understanding Fleet and Creating a Fleet Server
Open the hamburger menu of the left of our UI and scroll to `Management` and click `Fleet`. 

Fleet management will take some time to load, but once it is, we will configure our fleet server with the following options:
![My fleet config.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/fleet-config.png)

Click the `Add host` button and then generate our service token. I recommend taking a photo or saving this token SOMEWHERE because yes, you WILL forget it. Our next step is to actually get an Elastic Agent set up with Fleet.

In your VM, `cd` into `/etc/fleet` and run the following command: 

`sudo wget https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.17.6-linux-x86_64.tar.gz`

Given that this is a tarball, we will need to extract its contents with the follwoing: 

`sudo tar -xvf ./elastic-agent-7.17.6-linux-x86_64.tar.gz`

Once that's done, we can delete the original tarball if we wish. Return back to our frontend and generate a service token by clicking the big blue button.

`cd` into `elastic-agent-7.17.6-linux-x86_64`. Now it's time to run the below commands to start our Fleet server, as shown on the frontend:
```
sudo ./elastic-agent install --url=https://10.0.2.15:8220 \
  --fleet-server-es=http://localhost:9200 \
  --fleet-server-service-token=YOUR_SERVICE_TOKEN \
  --fleet-server-policy=499b5aa7-d214-5b5d-838b-3cd76469844e \
  --certificate-authorities="/etc/fleet/certs/ca/ca.crt" \
  --fleet-server-es-ca="/etc/elasticsearch/certs/elasticsearch.crt" \
  --fleet-server-cert="/etc/fleet/certs/fleet.crt" \
  --fleet-server-cert-key="/etc/fleet/certs/fleet.key"
```

I recommened pasting that entire command into a separate notepad and editting it; change the IP to match your Fleet server IP that we set earlier as well as the Elasticsearch IP and input your service token from before to replace `YOUR_SERVICE_TOKEN` in the above command. Paste the entire command into your terminal and hit Enter!
![Yatta!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/elastic-agent-success.png)
![Yippee!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/fleet-server-success.png)

You will notice that we are able to upgrade our Fleet server, however I am not sure if that requires upgrades to our other services at this time. You may go ahead and attempt to upgrade the service but make snapshots and take your time!

Let's continue on now. Click the `Fleet Settings` button in the top right corner of the UI. Change the Elasticsearch host from `localhost` to your Elasticsearch host IP. Save and Apply our changes and we're good to go to our next step of Agent Policies.

# Creating Agent Policies To Collect Logs
Navigate to the `Fleet` menu and click `Agent Policies`. Recall that agent policies will define what logs we want our agents to collect. To my knowledge, the Elastic Agent(s) **work with Filbeat** to get log data specified. Both the Filebeat configurations and agent policies work in tandem for log ingestion.

Click `Create Agent Policy`. For the sake of demonstration, we will create a basic policy encompassing Windows Endpoints, since we will be using a Windows VM down the line for our mock incidents.
![Policy configuration.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/policy-config.png)

Once your policy is created, click on it and select `Add Integration`. This is where we get to finally have some fun tweaking how our agents will work! What we're interested in is the `Security` section, but feel free to scroll and check out all the options at your disposal. What we **really** want is the `Endpoint Security` integration. Click on it once you've found it.

Click `Add Endpoint Security` and configure the integration as you please. Click `Save and continue` once your finished.

![Integration configuration!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/integration-config.png)

Once your configuration is saved, you'll get a popup to add Elastic Agents to your host machines. Click `Add Elastic Agent later`; we will handle this soon! Before we continue on, get to **this** menu and click on your `Endpoint Security` integration's name. 
![This lovely thing.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/agent-policy-config.png)

Scroll to the bottom and click `Show advanced settings`. Scroll until you find `windows.advanced.elasticsearch.tls.ca_crt`. This property will house the public CA certificate for elasticsearch on our endpoint(s). I have set the directory for this to be `C:\Windows\System32\ca.crt`; we'll handle getting the cert copied from our server VM to the Windows endpoint in a later step and will require us to tinker with our VM settings. For now, simply save and continue.

Next, we will add the `Windows` integration. Yes, that's its name. Add it just as we did before. The integration itself will log all sorts of behaviour but particularly **Script Block Logging** through the `PowerShell Operational` event log channel, which will log any and all commands ran in PowerShell, provided we have it enabled on our endpoint(s); we'll worry about this when it matters.

Click `Save and continue` then click `Add Elastic Agent later` once more. With all that said and done, we are done tweaking our Agents! Our next step is to install and set up a Windows VM.

# Intermission: Setting up a Windows 10 VM w/ VirtualBox
1. Go get [a multi-edition Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO) here. Pick your language as needed and wait for the download to complete. If you get redirected to a different site because you're already running Windows, <strong>install the media creation tool and download an ISO image in the creation tool!</strong>
2. Go to `Machine` --> `New` and select `Microsoft Windows` and `Windows 10 (64-bit)` as your Type and Version respectively. Select the previously installed Windows 10 ISO as your ISO image. Check the box to skip the 'Unattended Install' as well.
3. Set up your system resources as you wish. Refer to the original Ubuntu VM setup instructions as needed. I have my VM set to run 4GB of RAM and 2 vCPUs/Processors.
4. Set storage for your Virtual Hard Disk; I have allocated 50GB of space.

Our Windows VM is now complete. We can now start it up and proceed to setting up Windows. To keep it short and sweet, simply follow the installer's instructions; <strong>when you are prompted for a product key, click the 'I don't have a product key' prompt.</strong>

We'll be installing Windows 10 Home, no special stuff here. Make sure to select the 'Custom: Install Windows Only' prompt when you reach the below step.
![We want a clean install.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/windows-install-select.png)

Select the unallocated disk space, click 'Next', and wait for the install to complete :) Once the install finishes, Cortana will jumpscare you and the final bits of our Windows setup will commence; setting our region/country, keyboard layout, but most importantly, creating a Microsoft Account. <strong>The easiest way to skip this is to disconnect from the internet or Ethernet and attempt to go back to a previous step, then reconnect to the internet.</strong> This worked for me. Alternatively, you can input <strong>Shift + F10 to open the Command Prompt, then input `oobe/bypassnro`, which will reboot your system and effectively skip this step.</strong> I didn't try this myself, so let me know if it doesn't work and I will revamp this step as needed.

Otherwise, we can continue and make a Local Account with out desired credentials and set up security questions. Then we get to disable all of the spyware that Microsoft attempts to install or enable, <strong>say no to EVERYTHING.</strong> Cortana will annoy us once more before finally leaving us alone and letting the Windows install totally finish. 
![It's finally over...](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/windows-success.png)

Additionally, once you load up Windows, <strong>you can debloat it using [the instructions in this lovely GitHub repo](https://github.com/Raphire/Win11Debloat).</strong> This will improve Windows' performance by removing random apps and disabling telemetry. The README will guide you through everything, as it did for me! Once you're done there, you should check for updates in Settings and install any and all security patches and whatnot.

# Configuring Our Windows (VM) Endpoint
Add our Windows VM to our NAT Network as we did in the beginning with our server VM; go to `Settings` --> `Network` --> `Attached To`, and select `NAT Network`. Now, the two VMs will be able to communicate with each other!

Now that that's all done, start up your VMs and go to your Windows host. Open a PowerShell window as Administrator on your Windows VM and run the following commands:
1. `Test-NetConnection -port 9200 [elasticsearch IP, which should be your server IP]`
2. `Test-NetConnection -port 8220 [fleet IP, which should be your server IP]`

This will confirm that we can connect via TCP from our Windows host to the ELK server for our required communications. 
![test-net good!](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/screenshots/testnet-success.png)

Marvelous. Let's continue!

To be continued...

# Intermission: Simulating Events in a Controlled Environment
The following 2 sections will go over running simulated incidents and triaging them through our SIEM. These sections are highly recommended to go through but totally optional! However, they will prove a fine amount of competence with root-cause analysis, incident response, how to triage attacks and how to map an incident with both the Kill Chain framework and MITRE ATT&CK Entreprise matrix. Simply put, there is a lot of info to unpack, but it is well worth your time!

Read through the attack story, understand the TTPs at play, and run through each pre-incident step in preparation for running the attack! When your ready, I will guide you through the incident; how to simulate, triage and respond to the incident in a professional manner. The last incident is rather difficult; it is far more nuanced and goes over many TTPs and contains more setup. I wouldn't blame you if you skip it, but if you don't you will learn a TON from it.

# Incident #1: Attack Story, TTPs, Kill Chain Diagram and Incident Steps
Sally from Accounting sees an urgent email from her coworker (a similar, but fake email address!) that declares something is wrong with her device, requiring her to 1) Shut off Windows Defender and 2) Run a pre-configured script in her PowerShell terminal... 

Unbeknownst to Sally, the payload she runs in her PowerShell terminal opens a reverse shell on the attackers' machine. SOC was notified of Defender being shut off but also the script being ran in PowerShell. 

## MITRE Layout:
The MITRE Layout segments parts of the attack and associates them with their respective TTP(s) or Tactics, Techniques, and Procedures. This attack contains the following TTPs:

- Impersonation as coworker using fake email. [T1656](https://attack.mitre.org/versions/v14/techniques/T1656/) and [T1566](https://attack.mitre.org/techniques/T1566/).
- Listen on TCP port and open reverse shell. [T1204](https://attack.mitre.org/versions/v14/techniques/T1204/), [T1059.001](https://attack.mitre.org/techniques/T1059/001/) and [T1059.004](https://attack.mitre.org/techniques/T1059/004/).

## Pre-Incident Setup:
- Add `filebeat` input in `filebeat.yml` config to have `PowerShellCore/Operational` as name to support PowerShell >=7.4. Same `event_id` and `type`.
- Add `filebeat` input in `filebeat.yml` config to capture Windows Defender activity; being shut off/disabled. 
- Ensure log data is being sent to Elastic server.
- Ensure your Windows VM is updated and can run PowerShell.
- Know how to create a listener port on your machine; I will be using my host machine running Linux; run `nc -nvlp <port_number>` to listen on Linux.
- Ensure you are running NAT network connection on the VM in VirtualBox settings! Both machines should be on the same subnet as well (your host machine and the VM running it, in my case.)
- Ensure you can `ping` both machines. Ping Windows VM from host; host from Windows VM.
- <strong>Have the payload script ready with the given port and IP of the host machine you are listening on!</strong>

## Kill Chain Diagram, Made Using Excalidraw
![Kill Chain Diagram.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/killchains/ReverseShellKillChain.png)

## Incident Steps
To be continued...

## Incidence Response
To be continued...

## Post-Incident Report
The post-incident report will contain:
1. Summary of the actions taken by SOC as incidence response.
2. Mitigations for similar future attacks; enhance security posture.

# Incident #2: Attack Story, TTPs, Kill Chain Diagram and Incident Steps
An IEX exploit that results in a payload being downloaded and executed on a machine; installing an infostealer. Initial access done by phishing, asserting that a 0day security patch was supposed to be done by a fake IT/Help desk email by Scattered Spider (why not). Coworker deams it as a false positive but you investigate further and isolate the system from the network. You locate the infostealer program, which originally planned to exfil the data to a remote Discord server and delete it before it can do further damage.

## MITRE Layout:
The MITRE Layout segments parts of the attack and associates them with their respective TTP(s) or Tactics, Techniques, and Procedures. This attack contains the following TTPs:

- Posing as IT/Help Desk. [T1656](https://attack.mitre.org/versions/v14/techniques/T1656/).
- User executes IEX command per Help Desk's response. [T1204](https://attack.mitre.org/versions/v14/techniques/T1204/).
- Data is locally collected and stored locally. [T1074.001](https://attack.mitre.org/techniques/T1074/001/).
- Data exfilration to Discord server using internet (Discord on the Web). [T1020](https://attack.mitre.org/techniques/T1020/) and [T1567](https://attack.mitre.org/techniques/T1567/).

## Pre-incident Setup
- Make sure the Windows VM is updated and can run PowerShell.
- Have the payload script ready and configuring to download at the right website; TBD. 
- Ensure log data is being sent to Elastic server.

## Kill Chain Diagram, Made Using Excalidraw
![Kill Chain Diagram.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/killchains/IEX-IWRKillChain.png)

## Incident Steps
To be continued...
 
## Incidence Response
To be continued...

## Post-Incident Report
The post-incident report will contain:
1. Summary of the actions taken by SOC as incidence response.
2. Mitigations for similar future attacks; enhance security posture.

# Incident #3: Attack Story, TTPs, Kill Chain Diagram and Incident Steps
A NTLM brute force attack is attempted on a Domain Controller (DC) in an Active Directory environment. SOC notices an increase in failed login attempts and LDAP queries. Prior to the brute force, the attacker used both `PowerView` and `BloodHound` to perform domain enumeration to 1) Get inital domain information then 2) Perform domain enumeration while avoiding DCs to raise suspicion using `Invoke-BloodHound --ExcludeDCs`. The user had already had Local Admin access on a separate user machine and manages to reach the DC through other trust domains with stored credentials.

## MITRE Layout:
The MITRE Layout segments parts of the attack and associates them with their respective TTP(s) or Tactics, Techniques, and Procedures. This attack contains the following TTPs:

- Domain enumeration using `PowerView` and `BloodHound`. [T1106](https://attack.mitre.org/techniques/T1106/) and [T1599](https://attack.mitre.org/techniques/T1590).
- Compromise domain via Local Admin user with guessed credentials to continue escalating privileges through domain trust. [T1584.001](https://attack.mitre.org/techniques/T1584/001), [T1078.002](https://attack.mitre.org/techniques/T1078/002/) and [T1482](https://attack.mitre.org/techniques/T1482).
- Attempted brute force on Domain Controller server. [T110.001](https://attack.mitre.org/techniques/T1110/003/)

## Kill Chain Diagram, Made Using Excalidraw
![Kill Chain Diagram.](https://github.com/nubbsterr/ELK-SIEM-Setup/blob/main/killchains/BruteForceDCKillChain.png)

## Incident Steps
To be continued...

## Incidence Response
To be continued...

## Post-Incident Report
The post-incident report will contain:
1. Summary of the actions taken by SOC as incidence response.
2. Mitigations for similar future attacks; enhance security posture.

# Sources
- [MITRE ATT&CK Website for TTPs.](https://attack.mitre.org/)
- [Official Elastic Docs on upgrading Elastic components. Highly recommend you consult both this and other sources if you wish to upgrade your lab.](https://www.elastic.co/guide/en/elastic-stack/current/upgrading-elastic-stack.html)
- [Official Elastic Docs on downloading Beats but also for adding the Elastic repo w/ APT or YUM.](https://www.elastic.co/guide/en/beats/filebeat/current/setup-repositories.html)
- [Ofiicial Elastic Docs on using certutil to create certificates for Elastic components.](https://www.elastic.co/guide/en/elasticsearch/reference/current/certutil.html)
- [Official MS Docs on Script Block Logging and PowerShell event IDs.](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging?view=powershell-5.1)
- [Official Elastic Docs on configuring winlog input for Filebeat.](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-winlog.html)
- [Official Elastic Docs regarding the cat indices API.](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/cat-indices.html#cat-indices-api-request)
- A whole lot of ChatGPT and Google Searching. It helps to know the 'why' behind what we're doing especially in cybersecurity!
- [Massive installation and setup guide which I consulted throughout my journey](https://www.leveleffect.com/blog/how-to-set-up-your-own-home-lab-with-elk?utm_source=chatgpt.com)
- [Consult this for VirtualBox downloads. Go straight to the wget command if you're on Ubuntu/Debian-based repos like me, or use my script to install VirtualBox lol](https://www.virtualbox.org/wiki/Linux_Downloads)
- [Consulted for a bug with VirtualBox download; USB enumerate.](https://www.youtube.com/watch?v=7Bdm_-JNiSc&t=146s)
- [An incredible post I consulted for fixing SSH not working!](https://askubuntu.com/questions/1161579/ssh-server-cannot-be-found-even-though-installed)
- [Nice website that gave me for documentation about Port Forwarding.](https://nsrc.org/workshops/2014/sanog23-virtualization/raw-attachment/wiki/Agenda/ex-virtualbox-portforward-ssh.htm)

