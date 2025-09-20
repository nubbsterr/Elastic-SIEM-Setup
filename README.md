> [!CAUTION]
> Guide is being constantly updated! Please stay up to date and check back often for new commits and additions to the guide content :) 

# Elastic-SIEM-Setup
A shrimple guide to deploying the Elastic Stack to create your own local SIEM setup for shrimple Windows event log shipping and analysis; for simulations and more.

> [!IMPORTANT]
> You can use VMware to set up this entire lab. I would actually recommend it if you have access to the software since VirtualBox Shared Clipboard doesn't work even with Guest Additions installed, from my experience at least. Your next best option is using SSH so you can copy commands from your host "onto" the VM.

# Some Tips Before Diving In
Please check out the <strong>[Sources section](#sources)</strong> of this guide for any important documentation as needed. 

# What You Will Learn!
> [!IMPORTANT]
> This section is being reworked as the entire project is being revamped lol. A lot of things have been phased out but it 

By the end of this guide, <strong>you are going to know a lot of stuff</strong>; you'll:
- Know how to use VirtualBox to create VMs and configure networks and port forwarding
- Understand the workings of a SIEM, from ingest to visuallizing the data.
- Establish simulated attacks in a controlled environment and employ root cause analysis to gain a grounded understanding of a security incident.
- Understanding how Atomic Red Team tests can be used for defensive purposes to enhance security postures.
- Understand [MITRE ATT&CK](https://attack.mitre.org/) and using it to show TTPs of an attack.
- Understand the [Kill Chain framework](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html); understand the attacker POV.
- Understand incident response measures for triaging security incidents effectively.

# Q&A
> [!IMPORTANT]
> Below is a list of some questions I have answered preemptively to save time for both me and **you**:

> [!TIP]
> Q: *How can I reach you?* A: DM me on Discord (nubbbieeee)!

> [!TIP] 
> Q: *How do I stop all my services?* A: **They will stop on their own when you shutdown your VM,** there is no need to stop them manually. if anything, it may lead to goofing things up.

> [!TIP] 
> Q: *Why use this guide when others exist?* A: The rest of them, if any exist, **suck**. Most 'ELK setup' guides use Elastic Cloud, which is totally fine, but hosting everything and going step-by-step just makes you that much more informed and knowledgable. On top of me providing DOCUMENTATION for you to read AS YOU WISH. I don't know of many guides that do that. And no, those Arduino project guides don't count because they all copy each other line for line, letter for letter (lol).

# VirtualBox installation
First things first is to actually get a setup going to deploy this infrastructure. We'll use VirtualBox to deploy two VMs; one as our server and another as a host that will ship logs back to the server for analysis.  

I'm deploying this all on NixOS, so we need to add two lines to our `configuration.nix` file:

```nix
virtualisation.virtualbox.host.enable = true;
users.extraGroups.vboxusers.members = [ "user-with-access-to-virtualbox" ];
```

**You also need to disable KVM module as it will conflict with your CPU's virtualization features. You can do so by running `sudo modprobe -r kvm_<intel or amd, depending on manufacturer>`. You will need to do this before running your VMs everytime you reboot, unless you opt to create a config file to prevent the module from loading; whatever works best for you really.**

Now rebuild your system w/ `sudo nixos-rebuild switch`, and VirtualBox will be installed, though we cannot start our VMs since the VirtualBox kernel module is not loaded, so no virtualization will work yet. 

Reboot your host so the VirtualBox kernel module loads properly, then continue.

# Setting up your server
We'll use [Ubuntu Server](https://ubuntu.com/download/server) as our distro of choice for our server VM. There is nothing to note to be done differently from the install except that you may wish to install OpenSSH to SSH into the VM from your host, since shared clipboard is ALWAYS iffy for me, it is a lot easier to just set up a [port forwarding rule to access the VM through SSH](https://travishorn.com/mastering-port-forwarding-in-virtualbox-unlocking-connectivity) and run commands that way.

Our VM should have the following recommended specs:

- 2 vCPUs
- At least 4GB (4096 MB) of RAM
- ~25GB of disk space

Start the VM and begin the installation guide. There isn't anything to change past the defaults except installing OpenSSH which is up to you. 

Once the installation is complete, reboot the VM.

# Installing Elasticsearch and Kibana
Now we're going to basically download all we need to host our SIEM. I will leave comments on each command such that you know what is being done and why. Some steps will require us to configure the aforementioned services (Elasticsearch/Kibana) through their configuration files (`elasticsearch.yml` and `kibana.yml`).

```bash
# system upgrade and get goodies to install elasticsearch packages (gpg key and https for apt, which is installed per the elastic docs + to get the elastic packages over https; correct me if i'm wrong on that!)
sudo apt update && sudo apt upgrade -y
sudo apt install apt-transport-https wget gnupg -y

# get the elastic gpg key and add the elastic repo to our host
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list

# install elasticsearch, sudo apt update is needed so it can find the newly available elastic packages
sudo apt update
sudo apt install elasticsearch -y
```

Now go and edit the Elasticsearch config file at `/etc/elasticsearch/elasticsearch.yml` (using whatever text editor you wish; vim, nano, gedit, etc.) and edit/add the following fields:

- `cluster.name: srv` (can be whatever you want honestly)
- `xpack.security.enabled: false`
- `network.host: 0.0.0.0`  # recieve connections from any machine; e.g. our 2nd VM to ship logs
- `discovery.type: single-node`  # only one node in our cluster 
- **Remove all mentions of `cluster.initial_master_nodes` as it conflicts with `discovery.type`!**

Let's go ahead and start the Elasticsearch service now, and test its functionality:

```bash
# reload daemons so systemd can get elasticsearch's config file properly, then start the service
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service

# curl the elasticsearch endpoint
curl http://localhost:9200
```

You should see some JSON output featuring your cluster name and more. If so, you're in the right place. If you got errors when trying to start up the service, refer to your cluster's logs, at `/var/log/<cluster-name>.log`.

Let's continue on and install Kibana:

```bash
sudo apt install kibana -y
```

**DO NOT TERMINATE THE INSTALLATION IF IT FREEZES AT ~20% UNPACKING! This is known behaviour. Just give it some time to complete the installation. For me, it took about 1hr!**

Edit the Kibana config file at `/etc/kibana/kibana/yml` and edit/add the following fields:

- `server.name: kibsrv` (can be whatever you want)
- `server.host: "0.0.0.0"  # allow remote access to kibana
- `elasticsearch.hosts: ["http://localhost:9200"] 

Now enable and start the service as we did with Elasticsearch:

```bash
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```

If no errors occured, we can access our Kibana UI at `localhost:5601` on our server VM (through `curl`, unless you installed a web browser on your VM). To access it on our host (running the VMs), we can set up a [port forwarding rule to forward our Guest Port 5601 to any available Host Port for our server VM](https://travishorn.com/mastering-port-forwarding-in-virtualbox-unlocking-connectivity).

Give Kibana some time to setup and load the frontend. You should be able to log in automatically and access the SIEM frontend after a minute or two!

# Creating our Windows VM and installing Winlogbeat and Sysmon to ship basic event logs 
We now need to make a Windows 10 VM. Go grab an ISO from Microsoft's website [here](https://www.microsoft.com/en-us/software-download/windows10ISO) and create a new VM with the exact same specs as we have for our server VM (maybe have 30-40GB of storage just cuz Windows is massive). 

During this time, you can stop your server VM to save system resources if you wish.

Progress through the installation as you would. I'll be installing Windows 10 Home but you can install whatever you want and it shouldn't make a difference.

Bear in mind though, that we will soon need to make a NAT network between the two VMs such that they can communicate between each other; this will be done after our setup of the Windows VM is complete in this portion of the guide.

Make sure to silence Cortana because she's mad annoying and nobody likes her. **Once you reach the point where you are forced to setup/add a Microsoft account, go into your VM settings, and set your network adapter to `Not Attached`. Now click the back arrow to force refresh the page, which will fail, and allow you to create a local account (Works as of Sept 18 2025)!** Make sure to disable all Microsoft telemtry/optional thingies/**spyware** as you continue the installation. Once finished, we can continue on with our current task of the guide.  

**As a bonus, you may debloat Windows to save some system resources and have an overall more minimal system. I recommend using [Raphire's Win11Debloat](https://github.com/Raphire/Win11Debloat).**

**Additionally, make sure to remove the ISO attachment from your SATA drive on your VM by going into your Storage Settings, selecting the ISO you used, and clicking Remove Attachment. You can only do this once the VM is powered off!**

Download Winlogbeat on the Windows VM by going to [this link](https://www.elastic.co/downloads/beats/winlogbeat) and downloading the Windows ZIP file. Unzip the file to `C:\Program Files\Winlogbeat` (by creating a new folder in `C:\Program Files` whilst unzipping). Once complete, open a PowerShell session as Administrator and `cd` into `C:\Program Files\Winlogbeat\<unzipped folder name>`, then run:

```powershell
Set-ExecutionPolicy bypass
.\install-service-winlogbeat.ps1
```

Edit the `winlogbeat.yml` config file (in the same current working directory) **as Administrator** in Notepad as follows:

```
output.elasticsearch:
  hosts: ["<server-vm-ip>:9200"]

setup.kibana:
  host: "<server-vm-ip>:5601" 
```

We're now at the point where we need to create a NAT network for our VMs, so they can communicate between each other. To do so, navigate to `File --> Tools --> Network Manager --> NAT Networks` then hit Create, and we're done lol. All that needs to be done now is to edit the Network Adapter of both our server and Windows VM to be connected to our newly created NAT network (also make sure to allow the Windows VM to be recognizable on the network, when prompted). Bear in mind that we also need to remake our Kibana port forward rule again. This time however, make sure to specify the server VM IP as the Guest IP, so the right connection is forwarded. 

To get the server IP, start up the server VM and run `ip addr` to get the server IP on the NAT network. This will be very handy for us in the following steps.

The last thing for us to do before starting the Winlogbeat service is to install Sysmon, just so we can get more precise telemetry of our system. We can do so through the following steps:

1) [Download Sysmon on the Windows VM](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
2) Extract the ZIP archive, then open a PowerShell session as Administrator in the unzipped directory.
3) Go to [this GitHub repo](https://github.com/SwiftOnSecurity/sysmon-config) and grab [this Sysmon configuration](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml). Move this configuration file to the unzipped Sysmon directory (or modify the following command).
4) Run `Sysmon64.exe -accepteula -i sysmonconfig-export.xml` to execute Sysmon with the config file!

With that out of the way, we can continue on with setting up Winlogbeat.

We won't be enabling any modules for Winlogbeat, so we can go ahead and start up the service.

```powershell
.\winlogbeat.exe setup --dashboards 
Start-Service winlogbeat
Set-Service -Name winlogbeat -StartupType Automatic
```

`setup --dashboards` will load some preconfigured Kibana dashboards, which will be very nice to look at on our frontend. You can check them out through the sidebar under Analytics and clicking on Dashboards.

If all is good, we should be able to check our SIEM frontend and see our logs start flooding in!!

**Give Winlogbeat some time to get going, since we're running a minimal amount of resources for this setup. Also, logs in Elasticsearch will show up in UTC time, so make sure to set your time range accordingly (15 hours back preferably)!**

**Startup time overall can take a while and everything gets pretty slow given how low our spces are! If it takes a while to access Elastic, it is probably because of everything lagging behind and starting up on the server VM.**

# Testing the setup!
Now that our setup is up and running, let's have some fun with it! To do so, we're going to be using [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team/wiki) to simulate some attacks on our Windows VM. Not only will this provide you with baskground on various attacks done on Windows, but also expose you to [MITRE ATT&CK](https://attack.mitre.org/) as well. You'll also be able to see how attacks look from the perspective of the Blue Team, and gain a solid understanding of how DFIR operates during/after an incident.

For this guide, we will do 2 mock incidents. The first will be some [WMI User Reconnaissance (T1047)](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1047/T1047.md#atomic-test-1---wmi-reconnaissance-users), and the second will be classic [LSASS Credential Dumping (T1003.001)](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1003.001/T1003.001.md#atomic-test-2---dump-lsassexe-memory-using-comsvcsdll), but we'll be using `comsvcs.dll` to dump LSASS (Check out [LOLBAS](https://lolbas-project.github.io/lolbas/Libraries/comsvcs/) for more details).

To get started, let's install Atomic Red Team onto our Windows VM with the below commands. **Make sure to disable Windows Defender as it will flag the Atomic Red Team files lol**:

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
cd C:\AtomicRedTeam\atomics
Import-Module .\Invoke-AtomicRedTeam.psd1
```

This will install the `Invoke-AtomicRedTeam` Execution Framework alongside all the tests we need. 

# Mock Incident 1: WMI Reconnaissance
To be continued.

# Mock Incident 2: LSASS Dump
To be continued.

# Sources
- [Elastic Docs for installing Elasticsearch](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-with-debian-package)
- [Elastic Docs for installing Kibana](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-kibana-with-debian-package)
- [Elastic Docs for installing Winlogbeat](https://www.elastic.co/docs/reference/beats/winlogbeat/winlogbeat-installation-configuration)
- [Atomic Red Team Docs (a lot to digest)](https://github.com/redcanaryco/atomic-red-team/wiki)
