# Network

> **Network forensics** is a sub-branch of [digital forensics](https://en.wikipedia.org/wiki/Digital\_forensics) relating to the monitoring and analysis of [computer network](https://en.wikipedia.org/wiki/Computer\_network) traffic for the purposes of information gathering, legal evidence, or [intrusion detection](https://en.wikipedia.org/wiki/Intrusion\_detection). Unlike other areas of digital forensics, network investigations deal with volatile and dynamic information. Network traffic is transmitted and then lost, so network forensics is often a pro-active investigation

#### Wireshark

* Analyze the network dumps

Ubuntu

```
sudo apt-get install wireshark
wireshark file.pcap
```

Windows

```
https://www.wireshark.org/download.html
```

#### Network Miner

{% file src="../../.gitbook/assets/NetworkMiner_2-7-2.zip" %}

#### Tshark

```
# Install
sudo apt install tshark
(https://cheatography.com/mbwalker/cheat-sheets/tshark-wireshark-command-line/)

# Commands
tshark -r dump.pcap
tshark -nr payload.pcapng -Y 'frame contains "flag"' -T fields -e text
tshark -nr payload.pcapng -Y 'dns' | head
tshark -nr payload.pcapng -Y 'dns && ip.src == 10.10.10.10 && frame contains "local" && ip.dst==10.10.10.11'

# Extract Websocket (payload)
tshark -r something.pcap -Y websocket.payload -E occurrence=l -T fields -e text 
```

#### USBMS

```
tshark -r data.pcapng -T fields -e usb.capdata > file
cat file | tr "\n" " " | sed 's/ //g' > flag
xxd -r -p flag flag.bin
```

Pcap

```
https://packettotal.com/
https://www.hybrid-analysis.com/
https://iris-h.services
https://f00l.de/hacking/pcapfix.php
```
