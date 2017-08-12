I've been searching for a long time now for the perfect VPN Torrent box and have ultimately not be satisfied with a lot of the solutions. They often fail to meet some of my requirements.

The most common suggested solution is using an internet kill switch with their VPN provider with no other protections. Through my testing, this was not perfect. Some traffic has managed to expose my identity before the software based kill switch detected a disconnect in my VPN. Additionally, this is really annoying to manage on a daily driver PC since I do not prefer an always on VPN for my day to day tasks.

The past few days I've implemented 'a solution I've played around with in my head a bit and it seems to be working quite well. I won't walk you through step by step since my solution is a combination of many other well documented solutions, but I'll lay out my requirements for you and provide you a guideline to follow.

#### Requirements

- Torrent network traffic and data isolated from any of my personal traffic.
- Torrent traffic routed through VPN.
- Completed torrent downloads stored locally on my desktop.
- DNS leak protection.
- No or low additional costs. I already have a subscription to a VPN service.
- Remotely add, remove, and monitor torrents to my queue.
- Highly available.
- Auto start, resilient.

#### Solution

Windows 2012 Hyper-V VM running on my desktop. This is great because I already owned a copy from graduate school, the virtualization is free, and I'm familiar with the OS. One day I would like to convert this solution to unix since I know it will be lighter weight, but this will do for now.

I allocate 1 core and 512 MB of RAM, and a 24GB filesystem to account for torrent space. The filesystem can be expanded if necessary. My desktop is always on. I'll typically bump up the memory to 1024 if I have any work to perform on the box.

#### Implementation

- PPTP VPN using my VPN provider's DNS server.
- uTorrent remote server. This allows me to add torrents when I'm away from home.
- uTorrent to automatically move completed downloads to a fileshare on my personal desktop. This keeps my VM's filesystem relatively empty. I also monitor the folder with Plex so I can have anything immediately available in my library.
- Microsoft advanced firewall. With the exception of'SMB filesharing, DNS querying, a few rules for management, and any traffic produced by the uTorrent application'_only_'when connected to VPN, this box is shut off from the world.
- Additional file copies and health checks using powershell scheduled tasks.

#### Order of operations

This is the process I would suggest following for fewest headaches. Remember to take advantage of Hyper-V's checkpoint system as you move through the steps in case you need to reverse any changes. This is especially true when configuring firewall rules.

1. Install Hyper-V. This is free on Windows 8 and 10, but not enabled by default. I stumbled upon this by accident when I was uninstalling unrelated Windows features and'that is when I had a Doc Brown moment and the vision came to me. You can follow'[this guide to set it up](http://www.howtogeek.com/76532/how-to-install-or-enable-hyper-v-virtualization-in-windows-8/).
2. Create a virtual switch in Hyper-V. Sounds complicated but its super easy.'[Follow this guide](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_virtual_switch). If you are creating a setup like mine, I made an external virtual swich.
3. Get your OS running in Hyper-V (duh). Hopefully you have an image file for this. When I was a student, I was able to get a free copy.
4. Set the VM settings to auto start when the physical computer restarts.
5. Share out a'destination on your host and mount it as a drive within your VM.
6. Install uTorrent and get it configured the way you need it. Key things to focus on:
    - Do'_not_'autostart. Our scheduled task will do that.
    - Move completed downloads to your fileshare.
    - Connection limits.
    - Auto update.
    - Enabling remote access and testing this.

7. Configure your PPTP VPN. There are lots of guides for this, but I followed['this guide](https://www.privateinternetaccess.com/pages/client-support/windows8.1-pptp)created by my VPN provider.
8. DNS- For both my public and private networks, I set the DNS settings to use my VPN provider's as a primary and google as a backup. Prior to doing so, the default settings were using my ISP's DNS.
9. Connection testing. I use'[doileak.com](http://www.doileak.com/)'because it also has torrent testing. Whatever you use, you will probably need to add the site to IE Trusted Sites.
10. Configure your health checking in scheduled tasks.
11. Firewall rules. The hardest and most annoying part. Deserving of its own section.

#### Firewall

If you are not familiar with the order in which Windows will process your firewall rules, you need to get started by reading'[this quick one pager](https://technet.microsoft.com/en-us/library/cc755191(v=ws.10).aspx). The bigest takeaway from this is:

- Block rules. This type of rule explicitly blocks a particular type of incoming or outgoing traffic. Because these rules are evaluated before allow rules, they take precedence. Network traffic that matches both an active block and an active allow rule is blocked.
- Allow rules. This type of rule explicitly allows a particular type of incoming or outgoing traffic.
- Default rules. These rules define the action that takes place when a connection does not match any other rule. The inbound default is to block connections and the outbound default is to allow connections. The defaults can be changed in Windows Firewall Properties on a per-profile basis.

The first thing you'll want to do is modify the default behavior of the firewall in the case that a matching rule can't be found.'You will want to block everything except outbound over the public network. In the next steps we will add on 'allow' rules to let desired traffic through.

For additional protection, I also deleted all the default rules, both inbound and outbound.

Once you've adjusted your default rules, you'll need to add the following:

- PPTP and GRE protocols ' Allow outbound on private network. This is needed to establish the initial VPN connection.
- File and print ' Allow outbound 'on private network. This will allow you to view the fileshare you should have set up on your host machine that uTorrent will move completed downloads to. I used a modified preconfigured rule.
- DNS '' Outbound allow on private network. This may be only necessary if you are using a hostname for your VPN rather than an IP address.'I followed the answer in'[this link](https://social.technet.microsoft.com/Forums/office/en-US/a1240ce7-fe10-4d0f-91e1-ef1a579e9cfc/allow-outbound-dns-lookups-on-firewall?forum=winserversecurity)since I wasn't quite sure.
- Torrent Traffic'''I simply allowed all traffic inbound/outbound from the uTorrent executable. uTorrent also adds its own rules which I disabled.

#### Services

Since this machine is supposed to have as small a footprint as possible, take a few moments to disable any running services you don't think you will be needing.

#### Health Checks

My scheduled task runs at start up and repeats every 5 minutes. It is very minimal at the moment but I did not want to spend much time here just yet. This is what allows the VPN session to auto connect when the VM starts, and reconnect if the connection drops. Additionally, I found that uTorrent remote was not consistently functioning properly if I used the built in auto start feature. Because of this, I disabled built in auto start and implemented a feature in my script that will kill and restart'uTorrent 60 seconds after VPN starts.

$status = rasdial.exe

if ($status -like "*No Connections*")
{
 "VPN Failed, initiating restart"
 "Killing uTorrent"
 Get-Process -Name 'uTorrent' | Stop-Process
 "Waiting 30 seconds"
 Start-Sleep -Seconds 30
 "Starting VPN"
 rasdial.exe "PIA - Swiss" LOGIN PASSWORD
 "Waiting 60 seconds"
 Start-Sleep -Seconds 60
 "Starting uTorrent"
 Start-Process -FilePath 'C:\Tools\uTorrent - Shortcut.lnk'
}

I had to make two sacrifices with this design that I'm still looking for ideas to overcome:

1. rasdial.exe requires you to store your password in clear text. I could not find another way to auto connect VPN using Windows OS tools. It is not a huge sacrifice because it is not a login and password to my VPN provider's management console, only to actually connect the VPN. It does not risk me losing my account if leaked.
2. I would prefer to disable outbound traffic over public networks by default and create custom rules to allow only what is necessary for uTorrent to function, however I've been unsuccessful in finding the right rule set.

I've been using this for over a month. Overall it is a pretty good solution. I'm sure over time I'll have improved iterations. Good luck! Leave feedback!
