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
- Type `!`. It switches to root shell.

#### Extract Certificates
- In BGW210, run the following commands in order. Make sure you are in root shell.
  ```
  mount mtd:mfg -t jffs2 /mfg
  cd /tmp
  tar cf cert.tar /etc/rootcert/
  cp cert.tar /www/att/images
  ```
- Download http://192.168.1.254/mfg.dat and http://192.168.1.254/images/cert.tar to your **local** device.


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

### FAQ
1. Q: **Slow Speed**: The speed doesn't reach to the speed that I subscribed to.    
   A: Please make sure the NAT acceleration is enabled. (Web GUI -> Tools-> HW acceleration). If it says *incompatible with*, you need to turn off some services.


### To-dos
- [ ] Cross compile *wpa_supplicant*, so we don't need to install *Entware*.
- [ ] Ask Merlin to update *wpa_supplicant*.
- [ ] Try to use Openwrt/ddwrt to bypass AT&T's RG.
- [ ] Write a doc for compiling Entware packages from the source.

### Donation
- Bitcoin: 18hUjgNARRKWXr7hG9n62pWscZ4862TL6Q

[Back to menu](#menu)
## Credits & References
- [devicelocksmith](https://www.devicelocksmith.com/2018/12/eap-tls-credentials-decoder-for-nvg-and.html): EAP-TLS credentials decoder and the method to extract */mfg/mfg.dat*
- [earlz](http://earlz.net/view/2012/06/07/0026/rooting-the-nvg510-from-the-webui): Rooting The NVG510 from the WebUI
- [nomotion](https://www.nomotion.net/blog/sharknatto/): NVG589 root exploit
- [dslreports.com](https://www.dslreports.com/forum/uverse): A great forum with many useful information.
- [jsolo1@dslreports.com](https://www.dslreports.com/profile/422016): Provides many helpful & useful suggestions.

[Back to menu](#menu)
