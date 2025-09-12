> [!CAUTION]
> This guide WILL be getting revamped very soon to run off a simple Docker Desktop deployment method, which will DRASTICALLY downsize this guide and allow us to immediately get into DFIR simulations! See documentation [here](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/local-development-installation-quickstart).

# Elastic-SIEM-Setup
A shrimple guide to deploying the Elastic Stack to create your own local SIEM setup; for simulations and more.

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
First things first is to actually get a setup going to deploy this infrastructure. For a quick and easy deployment, we're going to use Docker Desktop in tandem with the [quickstart guide](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/local-development-installation-quickstart) provided by Elastic. 

Before that, we need a VM to run everything. For this, we'll install VirtualBox. Because I'm on Arch, I can shrimply do this with:

```bash
# MAKE SURE TO SELECT THE RIGHT KERNEL MODULE FOR YOUR KERNEL!!! Most likely, you need the virtualbox-host-modules-arch package!
sudo pacman -S virtualbox
```

And kaboom, VirtualBox is installed. Of course, refer to documentation for your distribution and the internet as a whole for how to install VirtualBox for your OS. 

Last VERY important thing to do before actually getting a server going is to add ourselves to the `vboxusers` group, otherwise we get a very annoying "Cannot enumerate USB devices" error when using VirtualBox.

```bash
sudo usermod -aG vboxusers $USER
```

Restart your system and then you're good to continue to the next step.

# Setting up your server
Because we're super cool people, we're going to use the super cool Debian 13 as our distro of choice, although you should be fine with literally anything, [as long as it can run Docker Desktop](https://docs.docker.com/desktop/setup/install/linux/#supported-platforms). Go ahead and install the installation media [here](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.1.0-amd64-netinst.iso), then we'll go ahead and create a new VM in VirtualBox (remembering to tick OFF unattended install) with the following specs:

- 2 vCPUs
- At least 4GB (4096 MB) of RAM, though I will be using 6GB (6027MB)
- ~30GB of disk space

That should be sufficient. We can go ahead and create the VM, then start it up. Proceed with the graphical installi; you can leave most things blank except your hostname, username, passwords, etc. For the purposes of this guide, we will install the GNOME desktop environment per the Docker Desktop docs so our setup functions properly (MATE is ugly and KDE is going to be way too slow for us imo).

The installation takes a very long time, so feel free to do other stuff on the side as you painfully wait for everything to install.

Once the Debian installation is complete, we'll go ahead and reboot our VM and see our lovely GNOME DE.

# Setting up Docker Desktop
[There are a few things to check off](https://docs.docker.com/desktop/setup/install/linux/debian/) before we can actually run the Quickstart script for the Elastic stack. Firstly, we need to enable KVM kernel modules, per Docker Desktop docs:

```bash
sudo modprobe kvm
```

Next we need to install Qemu. Before that, we'll do a system upgrade to keep our packages up-to-date:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install qemu
```

Next we need to install some GNOME extensions, which is pretty shrimple, by simply [following this link](https://extensions.gnome.org/extension/615/appindicator-support/) and installing the extension on our VM.

Finally, we can go ahead and [install Docker Desktop](https://docs.docker.com/desktop/setup/install/linux/debian/#install-docker-desktop):

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Now, go ahead and download the [latest .deb package for Docker Desktop](https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64), then install it as follows:

```bash
sudo apt-get update
sudo apt-get install ./docker-desktop-amd64.deb
```

# Setting up the Elastic stack w/ Docker Desktop
It is important to note that this Docker Desktop setup of the Elastic Stack comes with a 30-day trial license, after which everything just implodes. As such, it is recommended to create a snapshot of your server VM after you have completed this section of the guide so you can always restore to a fresh state of the Elastic Stack deployment.

To be continued.

# Sources
- [Elastic Quickstart Guide](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/local-development-installation-quickstart)
- [Docker Desktop Docs](https://docs.docker.com/desktop/setup/install/linux/)
- [MITRE ATT&CK Website/Framework](https://attack.mitre.org/)
