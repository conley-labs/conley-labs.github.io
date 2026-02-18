---
layout: default
title: Proxmox Homelab
---

# Proxmox

A journey into homelabbing using proxmox as the chosen hypervisor.

## Background

Let's start with hardware. As someone that is looking to homelab as cheaply as possible, I am always on the hunt for something to get the job done without breaking the bank. That being the case, working in tech at a large corporation has some understated benefits. Namely, when I mentioned that I wanted to get into homelabbing, there was a member of my team ready to offer up some old hardware he had gotten for free along with way more storage than I'll ever need to go with it.

Enter: my two new-to-me cisco UCS 240M3v2 hosts with dual Xeon processors and north of 19TB of hard drives installed!

### The Hardware Saga Begins

First thing's first: I have servers that once belonged to someone else, and they were being used. SO, there is data/applications installed that I will no longer need.

#### Goal #1 - Wipe

And so begins the process of ironing out how to get a clean slate on these servers so I can do what I really was aiming for: installing some kind of VM hypervisor so I can make all the things and do all the stuff without taxing my gaming PC.

Problem #1: I don't know the root password

This is problematic because remote access to the machines would be impossible without it. Meaning I probably won't be able to wipe it without hooking up a bunch of peripherals (monitor, KB, mouse, etc.) UGH

Problem #2: Even once I know the root password, how does one even wipe the machine?

I have some experience deleting file systems from machines, but I don't have a ton of experience outside of that. Words/phrases I knew might be related: "Format the hard drive", "Delete the file system", "Restore to system defaults"

This is where the googling begins. My thought process was that changing the root password so I could ssh was all fine and dandy, but then what?

I located the admin manual and began to read up. Here's some things I discovered:

> 1. Cisco UCS units have a Management Interface that is firmware-bound and runs at a level below the host OS on the server. It is available over HTTPS and can be accessed via the browser (so long as you have the root password :) ). Promising, but not entirely useful at this moment. *Foreshadowing*
> 2. This server has an integrated Hardware RAID controller onboard (YAY). This means all the drives I have installed can be arranged to have RAID by default and be abstracted to the OS, freeing up CPU cycles for more important things. [Note: if you intend to run something like trueNAS or unraid for a type of NAS solution, the HW RAID controller will be a burden. Make sure you know what it is you would like to do before you continue.]
> 3. There are several ethernet ports on the back, they represent several interfaces available to the host. Only problem going to need one of them though. Gave me good info on which one I should use for my purposes.
> 4. KVM is definitely something I am going to need. Can't see what I'm doing otherwise. (especially early on)

Armed with this information, I set out on my goal to reset the root password by using the CIMC to set the machine back to factory defaults and then reassinging the root password to something of my choosing. The first step of that was the same as using any other computer: I gotta have control and be able to see what I'm doing.

***Peripherals, ASSEMBLE***

I grabbed the first HDMI cable I could find and.....of course these servers only have VGA ports on the back...

Next problem, none of the monitors on my desk even have a VGA port. BUT my wife's PC does, and she's definitely not using it. ***Monitor: acquired.***

Oh yeah, probs need a mouse and keyboard too. Why would I give up mine? That would be silly. Stole them from my wife too. ***Keyboard: acquired. Mouse: acquired.***

Now about the cable: neighbor had a spare he wasn't using. ***VGA cable: acquired.***

Now that the gang is all here, time to hook it u... oh wait I have two servers and unplugging all this crap is gonna suck. Went ahead and spent $20 on a cheap KVM switch. ***KVM Switch: acquired*** [note: this was a waste of money, don't do this]

Alright, now we cook.

Curiosity seizes me first, so I just let the host boot up to see what's installed so I know what I'm blowing away. (ya know, in case there's something I want or something idk). Wrote all that down so I knew where things were and could more tactfully get rid of things rather than go scorched earch on it. (ended up going scorched earth on it anyways, but hey you never know) 

After losing my patience with trying to identify what exactly was on these boxes, I rebooted, and, following the BIOS menus, I found the option for the CIMC Configuration. Once that loaded up, we were smooth sailing! I marked the option to reset to factory defaults and set the default password to a value of my choosing. (note: this is the worst password I picked out of all of these because I had no way to copy/paste the value from a place like my password manager.) I also (for whatever reason) went ahead and assigned an IP address and a default gateway (two mistakes, I hadn't hooked them up to the firewall/DHCP server yet), as well as a hostname of my choosing.

***Goal Acheived: Root Password set and restoring Factory Defaults***

My next objective was to somehow wipe the storage on the machines so I could install my own programs on them. According to my reading, this could be done via the CIMC management console -- naturally, that is where I went next.

#### cue wasting 3 hours trying to get to the management console on a machine that requires Flash Player by intentionally installing old browsers and spinning up docker containers/VMs with the old versions supported

I think this goes without saying, but don't do what I did. Don't be a dummy.

I did end up getting it to work, though. For a moment I was logged into the admin console and able to execute some commands (power on, power off, you know, the fun ones). Then I noticed the KVM button. However, when i clicked it, it tried to open Java (WTF). This didn't work, so I tried to install java. Little did I know that in that process I was going to accidentally patch the machine and now Flash no longer works ***Arthur clenched fist meme***

This is when I discovered the amazing human who uploaded a video to YouTube on how to use KVM with this older CIMC/Cisco hardware. Linked in the description were copies of the KVM file that is downloaded when you click the button in the GUI. Replace a few fields, plug in the admin passwords, and run with the Java Web Launcher that comes with Java by default.

The only thing I had to do was go into the Java settings on windows and allow access to/from the IPs I had assigned in the CIMC config. Super easy, only took a second.

GREAT SCOTT I'VE DONE IT

I see the glorious video output of the server on the screen! Guess that KVM cable was a waste of money... Anyways, I have the ability to manipulate the servers from my PC! Woohoo! 

We're making great progress at thing point.

Next up: the LSA storage controller. Step 1: reboot. Step 2: Use the function keys during startup to get to the hardware controller. BOOM, winning. I did some light googling to figure out what the options were and what they meant. From there, it was as simple as removing the existing partitions and re-formatting the disks again in the recommended configuration (i.e. the defaults). In my case, the controller ID'd the storage difference in the drives and created two partitions, one was smaller in RAID 1 and the remainder were lumped into a giant RAID 6 array. In my mind, I separate these into storage that I can use that needs backed up, and a larger array that should be used for more mass storage.

***Storage: wiped.***

#### Goal #2 - Install Proxmox

This one is actually quite simple!

I started by downloading the ISO of the community edition (read: free edition) from the official proxmox website. From there, Balena Etcher was useful in helping to create the bootable media that I inserted the USB drive into the back of the first server. Using the BIOS options via the KVM video feed, I selected the USB drive as the boot drive and installed the softare on the volume I created using the LSA RAID controller.

Once all the questions were answered, the server rebooted and booted right into proxmox as expected. It was available on the IP I had assigned on a port that seemed real funky, but whatever.

**<--insert here the number of hours I then spent trying to figure out what was happening when I could not reach the service.-->**

TLDR; decide from the jump whether you are going to use DHCP or Static IPs for your HOST (read: server), and then decide it again for the GUEST (read: Proxmox). Not understanding that the HOST will have a separate IP/MAC that is abstracted by the NIC absolutely sent me in a tizzy.

No amount of competency (or so I thought) with computers helped me from having to install proxmox three times on the same machine before finally getting it somewhere close to correct.

#### Clustering

What I thought would be a relatively simple operation turned out to be perilous and more of a headache than I had initially anticipated.

I installed proxmox on the second of the two servers I had, and seemingly there was a simmple button to press to join the two together in holy matrimony. But nay, for there is a special process one must follow to make sure you don't screw it up forever.

After a little bit of tinkering, I was able to finally get the clustered nodes talking to one another and playing nice how I wanted.

The End Result:
- A proxmox cluster with two hosts, 4 CPU, and roughly 600GB of RAM between them.

#### Holy Hell These Things Are Loud

Yeah labbing is not a quiet exercise, the server fans and HDDs are loud, and admittedly the power consumption makes these the type of thing that you leave turned off unless you are playing with them. They certainly would not be a good fit for any kind of service I might like to run in  the house. That's a bummer, but a necessary evil as energy bills is the least fun way to spend money.


### Let's do some labbing
Now that the cluster was configured and all the memory and storage was ironed out, I had a lot to learn about networking and trying to get things talking in some kind of consistent manner.

Standing up the first VM was trivial, I was able to accomplish this by painstakingly finding the .iso of the image I wanted to spin up and uploading into proxmox for use in image selection during VM creation.

Another example was the Kali Linux install, as that was a QEMU install rather than a normal VM creation. Basically, you had to hack the filesystem to place a VM in it without using the UI. It then would show up and you could boot up and use it from there. This was a pain, and I still don't understand qcow and qemu file types.

The first real lab experiment I wanted to run was the GOAD (Game of AD) - this would involve spinning up AD domain controllers and some host machines with the goal of having a test range for ethical hacking purposes. I found a guide online and have been following those instructions to set everything up. However, knowing that I'm the type of person that resists the urge to just follow instructions, I chose to try and freestyle it so I could use the FortiGate as the only FW solution I needed. This created complexity, but I do feel like I understand better how these things operate under the hood now, even if I landed on just running with the virtual firewall instead.

### Side Quest: Upgrade to Prox 9.x
In between my endeavors to launch GOAD through proxmox, I decided to investigate how one would go about launching containers on PVE. Turns out pve 8.x does not support launching OCI registry container images natively in the UI. However, it was included as a new feature on PVE 9. Therefore, I would need to go through the process of upgrading the proxmox cluster to the new version to unlock this functionality.

The guide in proxmox documentation was pretty straight forward. Along the way I learned that the process of installing upgrades was as simple as it is with any other linux distro by leveraging the package manager. What was different was the fact that the apt repositories configured on the instance by default are aligned with the enterprise license, which I do not pay for and thus do not have a repository key to download artifacts for installation. Via the UI I was able to enable the non-enterprise repository for the current version (debian bookworm) and get the updates as required. Once completed, my cluster was then updated to 8.4 (from 8.2.2).

I then ran the pve8to9 script that comes packaged with the pve 8.4 release to check whether the system was ready for upgade or not. Everything seemed to check out with exception to the neeed to install the intel-microcode package (which required me to enable non-free-firmware in the package repository sources). I don't think this was mandatory by any stretch, but was something I went ahead and did to make the number of errors smaller. I was not particularly intrested in undoing all the work I had done until this point.

Then I got cold feet and decided that I needed to practice on another host before I would try it on my 'prod' cluster. The docs recommend this. Thinking about it, there was not really anything in the cluster that would be overly terrible if it were lost, but still. I cannibalized the PC my wife was no longer using *with her permission*, installed the same base version of pve on it, updated to 8.4, ran the pve8to9 script, and validated the entire procedure from start to finish. I also managed to make this happen while remotely managing the host via SSH after the proxmox install. I am really starting to get the hang of managing the network portions of this lab!

After the upgrade was completed and validated, I proceeded to execute the steps on the two prox clusters. Both were completed without issues and the cluster was operable after the upgrade.

After logging back in to the admin console post-upgrade, I was able to see the new dialogs in the UI to add OCI repo-based container images. I install Arch (base) as the first container. It launched without issues and the Arch command prompt was available when opening the console window for it. Great success!
