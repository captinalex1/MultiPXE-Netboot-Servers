# MultiPXE-Netboot-Servers
This is a quick type-up of my adventure creating multiple netboot servers and having my machines know which netboot file is for it.
I'll make this prettier later when I've organized my findings. 

So I just over engineered the most interesting setup that takes laziness to a new level but also makes things so much nicer when mass deploying stuff. Pixie (PXE) Netboot! :D
But Alex thats easy to setup and write a quick ipxe script file. 
Yes my fwiend it is, but only if you plan to use 1 operating system! :D
But I use, multiple esxi versions, proxmox, different linux distros and windows! So haha! Mine is very not fun! 

Silly talk aside here is what I mean, when net booting linux, esxi, proxmox or any other linux distro they prefer to netboot using BIOS Boot and NFS. They dont like me smashing the iso file into the ipxe code, they need me to extract the files from the iso and all that jazz. 

Then you have windows that wants a WinPE iso file to boot from that will bring up a cli for you to create a network share before finally being able to do setup.exe in the cli that will launch the windows installer from also a extracted folder. But microsoft being microsft they force the use of uefi/efi. I tested and if you boot the windows installer in BIOS it will not let you select any drives that you wanna install windows on. Netboot, or from a bootable USB.

Whats the problem? Inside you network dhcp settings you can only select one netboot file per subnet with one ip and also that file can either be for ONLY bios or efi. Even if you use a dhcp relay for both, the subnet you are booting from can ONLY use one or the other. 
Solution when you dont know something? Ask others or in my case ask some network engineers in a few discords. 
Answer? No one really knew how to do it either. Ok.. ask chatgpt and other AIs out there; answer? Its possible, just switch the file and ip out for the right netboot server. The discords basically said that too.

But I wanted more than just that I wanted the new vm to know from the moment I hit create which boot server it belongs to and how to relay to another one if need be. 

So I sit down look at my network rack and all the ports on my server nics and try and think this through and I came up with a very interesting idea. And Ill need to make a nice documented github with pics and all to make this make sense. 
The Idea: Create the dhcp relay networks, one for EFI and one for BIOS. Test to make sure they work by doing the switching out to make sure every step works and good so far. Now, I create 2 separate subnets on my Unifi Network, with different IPs and 2 separate vlans. One named BIOS Net and one name EFI Net. And in their dhcp settings I add the EFI file to the EFI Net and BIOS file to the BIOS net and the server IPs to said files. 

Pretty much ive doubled everything. Whats really beautiful is this can expand as big as one would want. You want 4? Make 8 virtual networks. 4 dhcp relays and 4 subnets for the 4 netboot servers. 

Now here comes the vlan tagging part. Easier on Ubiquiti devices but I have a Cisco switch so I had to do everything in CLI but basically, I vlan tagged 2 ports on my unifi and on my cisco saying that 1 port will ONLY allow the efi file to pass through and the other for the bios file. Then I plugged in 2 ethernet cables to the back of my server that I needed to see these files. I plugged both into my switch and vlan tagged those two cables. 

Basiclly: From unifi to cisco I have 2 ports tagged with two cables and from cisco to the server same thing. 

That server uses vmware so I had to set the virtual switches up as seen in the pic. Meaning that server now has the capability to grab both files and will grab the file that corisponds to the right boot method. So if I make a VM for ubuntu with the Boot set as BIOS it knows Ubuntu reqires the Bios netboot file. And same for uefi/efi when making a windows vm. 

Now its VM level before we make this work. Everything works so far.
On the VM level I create my VM like normal and I set the Boot setting to the one needed for the vm in mind. Haha and Im happy with this part as now you have 2 extra options shown in the pic. I can now select the network that corresponds with the boot settings of my vm. And as soon as that vm boots, itll grab the ipxe file made for its boot preference and show you all the OS's that boot preference can do. 
And same for the uefi/efi side for windows. 

No need to go into the switch/router or dhcp settings and change stuff around. Just create the vm, select the network and automation takes the rest. 
Once the machine is booted and OS has been installed, you go in and change the network setting to however you want, in my case. Once the OS is installed, I can put the OS on a VM Network, or Minecraft network or any other network I want that VM on. :D

And thats how I over engineered my PXE Netboot environment. Am I the first to probably do this? Most likely not, but scraping through forums, reddit, google and going as far as 10 google pages in I couldnt find anything close to this. There was just no good info. Only info I can find was people talking how to get PXE boot working at all for a few of their OS's of the same Boot settings or architecture. No one mentioned having multiple netboot servers relaying to all their subnets at home and the physical or virtual machines knowing which file is for it. Everyone seemed to wanna stay with one netboot server. 

On the VM side its easy, but I did mention physical. Lets say you have loads of laptops you wanna install windows on and you dont wanna sit there and vlan tag each port, I cant test this cause I dont have the equipment, but vlan tag the whole switch for the bios ipxe file and another whole switch for the efi ipxe file. Then just plug your systems into that switch and bam. UwU
