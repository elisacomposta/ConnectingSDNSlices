# ConnectingSlices

### TABLE OF CONTENTS<br>
#indice

### COMPONENTS USED<br>
•**Open vSwitch**<br>
•**RYU controller**: defined in ComNetsEmu<br>
•**Host and switches**: defined in ComNetsEmu<br>
•**OpenFlow 1.0** (both RYU controllers and OvSwitches need to be set up working with this version)<br>


## 1st topology

### STATEMENT (GENERAL IDEA)<br>
•Here we have a cycle. Every host can communicate with the others, but when a flood starts we enter in an infinite loop.
<img src="https://user-images.githubusercontent.com/98694899/153767602-b65255fd-3629-4aeb-96d0-10c3fbfa93dc.jpg" width="30%" height="30%">

<br>•A topology slicing avoids infinite loops by separating the cycle into two trees, controlled by two controllers; the two slices cannot communicate <br>
<img src="https://user-images.githubusercontent.com/98694899/153768938-7c482ef2-37e2-470a-ac6d-09e848209593.jpg" width="30%" height="30%">

<br>•We interconnected the two slices by adding a common root (s9); a third controller manages inter-slices communication<br>
<img src="https://user-images.githubusercontent.com/98694899/153768987-246ebf73-190a-48b1-8ac3-832fb7aaf4c5.jpg" width="50%" height="50%">


<br>•The provider doesn’t want the slices to send UDP packets; s9 sends the inter-slices UDP packets to a server that filters (and then drops) the packets<br>
<br>
### TOPOLOGY<br>
<img src="https://user-images.githubusercontent.com/98694899/153769039-62ec651f-a413-4250-b5c8-20fa28471f52.jpg" width="50%" height="50%">

We realized three different slices (topology slicing):<br>
-**slice1**: a controller allows communication between: h1, h2, h5, h6<br>
-**slice2**: a controller allows communication between: h3, h4, h7, h8<br>
-**connecting_slice**: a controller allows non-UDP packet inter-slices transmission, and filters UDP packet to server1 and server2 (if sent by slice1 or slice2 respectively)<br>
<br>_Note_: server1 and server2 are configured not to send any packet. They can only receive UDP packets that are filtered by s9<br>
<br>
### DEMO<br>
**Set up the environment:**<br>
```vagrant up comnetsemu	#start the VM up```<br>
```vagrant ssh comnetsemu	#log into the VM```<br>

**Set up the topology in mininet**<br>
```sudo mn -c``` 			#flush any previous configuration<br>
```sudo python3 network.py```	#build the topology<br>

**Set up the controllers** (in a new terminal)
```./runcontrollers.py```		#script to run all the controllers in a single shell<br>

Create a new terminal for future flow table test<br>

**Test reachability**<br>
By running a ```pingall``` we obtain the following result:<br>
<img src="https://user-images.githubusercontent.com/98694899/153769411-7121dee2-2ae4-4369-9dd4-355b68d1915d.png" width="30%" height="30%">
_Note_: ping and pingall send ICMP packets.<br>
_Note_: server1 and server2 never send and receive ICMP packets<br>

Perform ping between host 1 and host 2<br>
```h1 ping h2```<br>
<img src="https://user-images.githubusercontent.com/98694899/153769590-d612d838-37de-40ea-aab7-c5e69427884a.png" width="60%" height="60%">

Perform ping between host 3 and host 4<br>
<img src="https://user-images.githubusercontent.com/98694899/153769600-5a9740c9-7ab2-4820-8cff-3872887ca61d.png" width="60%" height="60%">
<br><br>
Intra-slice communication works correctly.<br>

Show s4 flow table<br>
```sudo ovs-ofctl dump-flows s4```<br>
<img src="https://user-images.githubusercontent.com/98694899/153769807-7fe49a24-de63-453c-a20b-c13f79780c2b.png" width="100%" height="100%">

Host 1 can sand TCP packets to Host 4<br>
```h4 iperf -s &```<br>
```h1 iperf -c 10.0.0.4 -t 5 -i 1```<br>
<img src="https://user-images.githubusercontent.com/98694899/153769940-f1cd07a1-a8a1-418e-af0c-e7e4bd2c4231.png" width="40%" height="40%">

Host 1 cannot sand UDP packets to Host 3<br>
```h3 iperf -s -u &```<br>
```h1 iperf -c -u 10.0.0.3 -u -t 5 -i 1```<br>
<img src="https://user-images.githubusercontent.com/98694899/153769890-606e8440-030c-4cbb-8e5e-a7b023b3fe4a.png" width="40%" height="40%">

	
Show s9 flow table (path depends on protocol)<br>
```sudo ovs-ofctl dump-flows s9```<br>
<img src="https://user-images.githubusercontent.com/98694899/153770285-680e26cf-61c8-4198-a31a-6316ea2802c2.png" width="100%" height="100%">



**Close and clean everything up**<br>
It’s better to flush the topology with ```sudo mn -c``` and to stop the VM with ```vagrant halt comnetsemu```	
