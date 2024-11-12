Here we will be setting up our DNS server using Alpine Linux

First we are going to click on New in Oracle VirtualBox and to createa a new virtual machine.
Next we are going to select our ISO image for Alpine

Here are the initial configurations I made for this VM:
- Name: DNS
- Type: Linux
- Subtype: Other Linux
- Version: Other Linux (64-bit)
- Base Memory: 2048 MB
- Processors: 1 Core
- Create a virtual hard disk now
- Storage: 8 GB

After finishing the first set of options, select your VM and go into 
Settings >> Settings >> Network >> Adapter 2 >> Attached to Internal Network

*Make sure the name of the internal adapter is the the same as the one we created before with our router*

Once Alpine boots up, you can login with the following credentials:
- Username: root
- Password:

Since root has no password, we are going to utilize the **passwd** command to add one to our user.

After assigning a password, we need to assign the box an IP address, and we set this up through DHCP.

First we need to turn on our network interfaces through the CLI, we can use the commands:
- ip link set eth0 up
- ip link set eth1 up

Now we need to add the interface and how it will retrieve its ip in /etc/network/interfaces
- iface eth1 inet dhcp

Now we are going to restart the service to have the configuration take effect
- /etc/init.d/networking restart

This process is slightly different since Alpine doesn't have dhclient but it instead has udhcpc.

We are going to run the command:
- udhcpc -i eth1
Now we need to verify that the command with:
- ip a show eth1

![dns2](https://github.com/user-attachments/assets/7ac5191c-4d93-4602-bb5f-5cd29231e89e)

Once completing that, we are going to add two repositories to our repositories file in /etc/apk/repositories

We can do this through the echo command or vi, for simplicity we'll be using vi.

Run the command vi /etc/apk/repositories

Once the text editor opens, press the I key to insert text and add the following two lines to the file:
- http://dl-cdn.alpinelinux.org/alpine/latest-stable/main
- http://dl-cdn.alpinelinux.org/alpine/latest-stable/community

After adding these two line in we are going to update our DNS box with the command:
- apk update

Then we are going to install *bind* for the DNS server
- apk add bind

![dns3](https://github.com/user-attachments/assets/7e838d57-f265-4f22-8ea0-506754d62a94)

After installing bind, we can then edit the bind configuration file at /etc/bind/named.conf. We can use the *vi* command to edit the conf file on Alpine.

![dns4](https://github.com/user-attachments/assets/407d5eb4-92c5-4db9-ab6b-88490e444c20)

Now that we configured our named.conf and have specified our dns zones at /etc/bind/zones/db.home.local, lets edit that file

*Note: I had to first create the directory of /etc/bind/zones with the mkdir command.*

![dns5](https://github.com/user-attachments/assets/1e95b650-bbe0-434c-b2d9-d16b552d5e62)

For this file we will add an A record for our nameserver so that we can access each of our services through hostname instead of IPs. Now we will apply our configurations through restart and enabling the service.

![dns7](https://github.com/user-attachments/assets/615183a4-6c00-47f5-b6b1-44ccc77bf93c)

**Before we continue, let's perform a backup for the VM since Alpine does not save when shutting down. We can achieve this by first editing the /etc/lbu/lbu.conf file. Once edited, run the command mkdir /root/config-backups , then run lbu commit to save our configurations.**

![dns8](https://github.com/user-attachments/assets/15eadc01-b8bb-4450-a70f-6fd6f2b6905f)

Now that we have our backup, let's go ahead and test our DNS. First we will test DNS locally. We can perform this running the *dig* command and testing it on both our nameserver and google's.

![dns9](https://github.com/user-attachments/assets/508c1601-9587-4e6b-bd26-afc25d9882f3)

After looking at our output, we can see that the A records are visible showing that DNS is functional.
Now we're going to edit pfSense to work with our DNS as well. To do this, we'll switch back to our Kali VM and head to the web interface.

![dns10](https://github.com/user-attachments/assets/3801dca8-f026-4db5-9385-b06c5a95a8b6)

On the web interface we can easily add our DNS server by providing its IP and hostname to pfSense. Once those are written you can scroll to the bottom of the page and save your new changes. Now we're going to head to Services >> DHCP Server >> LAN

![dns11](https://github.com/user-attachments/assets/26f61c94-47f2-448a-a4f9-028684eb7fa9)

Here in this section we can provide pfSense the IP of our DNS server so that it can update DHCP.

While we're here in our Kali VM, we'll use nslookup and dig to see what results we get from this VM.

![dns12](https://github.com/user-attachments/assets/68773bd4-d141-4ad4-ae05-f175f251cd93)

Now that DNS is fully functional internally, the last thing we'll do here is setup forwarding so that we can resolve internal and external domains with just our internal DNS server. To do so we will go back to our Alpine VM and edit the /etc/bind/named.conf file

![dns13](https://github.com/user-attachments/assets/da5a1272-c919-4b52-8c94-c37e9a50bd35)

After adding in the lines into the options section, we can now resolve both internal and external domains with just our DNS. Before testing our changes, let's apply our changes here and save our config. We can do this with the command: *lbu include /etc/bind/* and *lbu commit*. We can also add the command *rc-update add named* to have Bind start on boot.

Now that we are here on Kali lets run nslookup for google with our DNS IP and look at the results.

![dns14](https://github.com/user-attachments/assets/60f6da3a-2ca8-4bab-b79a-ce080a7a5906)

As we can see from our results, forwarding is functional and DNS is up and operational!

We will be back here to add new A records for our new servers that will be coming live soon.

Steps to ensure that configurations and backups are consistent:
1. Verify if lbu is configured correctly
   - cat /etc/lbu/lbu.conf
   - vi /etc/lbu/lbu.conf
     - LBU_BACKDIR=/root
3. Mount a writable backup directory
   - mkdir -p; /media/usb
   - vi /etc/lbu/lbu.conf
     - LBU_BACKUPDIR=/media/usb
4. Run lbu to save the config
   - lbu commit
   - lbu status (make sure there is an output)
5. If nothing else works switch from ram to sys mode
   - setup-bootable

Note: None of the Lbu commands were working and persisting so I instead saved a snapshot. Work smarter not harder :|

