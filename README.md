# Homelab
This repo will showcase my own dedicated homelab that is setup through VirtualBox.

I recently competed in a blue team infrastructure competition known as CNY Hackathon, and I had a very fun experience competeting. So much so, that I wanted to create my own lab so I can do it again.

Here I will be writing out my documentation on my own home lab, which has an internal and external network. Consisting of the following:

Internal:
- Kali Linux
- Apline Linux
  - Web Server
  - SSH Server
  - Files
  - DNS
- Windows Active Directory 
- Router
  -pfSense 
- IDS/IPS
  -Suricata 
  
External:
- Metasploitable VM
- Zeek
- SIEM
  - Gravwell 
- Secondary DNS

*This entire network is ran off of my desktop with VirtualBox*

***Router Setup***

For the router I will be running pfSense. I will use this set up the networking for the internal infrastructure.

I first downloaded an ISO of pfSense from their website and I will be running it as its own dedicated vm.

During setup when selecting the ISO, I chose the following options:
- OS: BSD
- Type: FreeBDS
- Version: FreeBSD x64
- Memory: 1024 MB
- Disk Type: VDI
- Storage: 10GB

I provided the VM to network interfaces, I used the following options:
- Adapter 1
  - NAT
  - Our WAN interface 
- Adapter 2
  - Internal
  - Our LAN interface 

After doing this I started the VM, here is the first page of the installer:

![pfs1](https://github.com/user-attachments/assets/b44b6ba6-d00f-4163-9ce4-64fc89fc8a37)

After pressing enter to accept, the next page will appear asking to either install pfsense or launch a shell.
We will be installing pfsense so we will press enter.

![pfs2](https://github.com/user-attachments/assets/5a4d7485-692b-4aa1-b1d3-fbac838d98a4)

After pressing enter to start the install, it will bring us to this page where we can accept and start the networking installation.

![pfs3](https://github.com/user-attachments/assets/8f16ddea-d395-41c1-a2fd-9cf0c552169b)

Press enter to start the install.

![pfs4](https://github.com/user-attachments/assets/002884d7-493c-4b7a-be44-4659849c935d)

In our case, we will use em0 as our interface for WAN, we will press enter and select the interface.

![pfs5](https://github.com/user-attachments/assets/59634d98-2f7e-4140-9b8d-42c17c026ad6)

After selecting our interface for WAN, we will be prompted to assign the following:
- Interface mode (DHCP or Static IP assignment)
- VLAN Settings (Enable or Disabled VLAN Tagging)
- Use local resolver (True or False)

I will be using the default options provided and continue to the next page.

![pfs6](https://github.com/user-attachments/assets/4ca881e3-9a0c-4a97-97ae-11bc61eb9c7b)

We will now select em1 as our LAN interface to continue with our installation.

![pfs7](https://github.com/user-attachments/assets/643d6a81-9a1d-4068-ac92-7035f87bc408)

*Note: My IP address for the router is 10.0.0.1 not 0*

***I realized that when configuring the pfSense router, it didn't properly keep these configs as some were incorrect, so don't worry if you make a mistake, you can fix them later***

On this page we will assign configuration to our LAN interface. Here are the options:
- Interface Mode
  - Static
  - Dynamic 
- VLAN Settings
  - VLAN Tagging (Enable / Disable)
  - VLAN Tag #
  - Priotity #

- IP Address
- DHCPD Enabled (True / False)
- DHCPD Range Start
- DHCPD Range End

The screenshot above showcases the options I selected.

![pfs8](https://github.com/user-attachments/assets/5922af9a-9d05-4505-a959-441d8f3a70e8)

After configuring your two interfaces, you will be prompted if you need to change anything or if you would like to continue. 

![pfs9](https://github.com/user-attachments/assets/e2cfb693-073a-4c4b-8b82-4ae7d2197ebf)

I continued and it brought me to the network install where the VM is checking if I Community Edition installed.

![pfs10](https://github.com/user-attachments/assets/168ca69f-efe7-42d3-bb7f-58c0054c5afc)

We will then be prompted with install options, I simply chose the default options as they were good for me.

![pfs11](https://github.com/user-attachments/assets/b0e24217-63b0-48de-a7e0-219837aeb748)

After accepting these install options you are then prompted to chose virtual device configuration type. Note for the next few options I just chose the default setting.

![pfs12](https://github.com/user-attachments/assets/8a4a8135-c1e6-4f7f-ad93-16a68e5d707b)

![pfs13](https://github.com/user-attachments/assets/0c839bec-ba37-4542-83e8-f46372dc931a)

![pfs14](https://github.com/user-attachments/assets/2f403b32-de13-41c2-8bae-e671f0716781)

After selecting all of the harddrive related options, you will be promted to install a version of community edition, I chose the stable option.

![pfs15](https://github.com/user-attachments/assets/2096cdfb-3aba-4c61-a367-5ce47c1741a2)

After doing so, your install will begin and pfSense will start to download all necessary dependencies.

![pfs16](https://github.com/user-attachments/assets/7fb34f25-51af-4ade-84fe-343bf7d3c58e)

After a minute or two, the install setup will be complete and you can select done.

![pfs17](https://github.com/user-attachments/assets/d757e16f-f85b-4ced-b1d7-ed91501171c0)

After the download finishes, you will be asked to reboot to confirm the install. I accidentally got out of the wizard so intstead of seeing a reboot option I was sent to the CLI. 

**Please note that before rebooting you must remove the ISO of pfSense from your mount or you will need to redo the entire setup from before.*

This can be done through here *Devices >> Optical Drives >> 'Name of pfSense ISO'*

![pfs18](https://github.com/user-attachments/assets/aaab2206-b614-4b29-982c-0900e147cb3c)

After rebooting pfSense, you will be able to access the Console for VM

![pfs20](https://github.com/user-attachments/assets/ae392e0e-c0c5-4188-9a8f-b8855789ec70)

I had a few issues but after figuring out I now have access to the main console. Yet my WAN has a designated Class A address but my LAN doesnt. I'm going to now verify interfaces and reassign them incase they changed.

What I did pressed 1 to configure em1 for LAN, then I pressed 2 and configured it by statically assigned the ip address, but said yes to dhcp on the server, where it prompted me for my IP range and I provide the following:

- IP Address
  - 10.0.0.1
  - 28 bits (/28 Subnet Mask) 
- DHCP Range Start
  - 10.0.0.2
- DHCP Range End
  - 10.0.0.15 

After completing that, I then prssed 5 to reboot the system to apply new changes.

I then turned on my Kali VM and ran dhclient to assing myself an IP in the same subnet as pfSense, once doing that I was able to ping from both devices to each other.

![pfs25](https://github.com/user-attachments/assets/c4855d52-2b5d-41ae-8228-7dd6d09fcfb5)

Now that has been accomplished, you can use Firefox in Kali and access the following url:
- **http://10.0.0.1**

![pfs21](https://github.com/user-attachments/assets/c2b24703-db3b-4afd-8f52-f70006c914d1)

Once on the webpage login, you can run input the following default credentials:
- User: admin
- Password: pfsense

![pfs22](https://github.com/user-attachments/assets/b675edcf-96e7-49e3-a9d1-cd964a22f51b)

Now we're going to run through the wizard and select the defaults for most options and continue until we're finished.

When done you should have a screen like below and you have successfully setup your pfSense router.

![pfs23](https://github.com/user-attachments/assets/cd9a8e07-a72c-4a79-8d29-4a644a22df1b)

Now currently for me I wanted to setup other services like a DNS and NTP server so when it comes adding those, we will be making adjustments by then.

The only extra step as of now (11/10/2024) that I will be doing is creating another user with admin and disabling the admin account.

You can do this by going to System >> User Manager >> Users >> + Add

![pfs24](https://github.com/user-attachments/assets/662c4e3f-ff45-4a64-9b47-a7076ff1fec2)

On this page, you can provide account info, what groups the user is apart of (I put myself in admin), and any SSH keys they can use to remote into the router.



