# ConnectingSlices

### TABLE OF CONTENTS<br>
#indice

### COMPONENTS USED<br>
•	Open vSwitch<br>
•	RYU controller: defined in ComNetsEmu<br>
•	Host and switches: defined in ComNetsEmu<br>
•	OpenFlow 1.0 (both RYU controllers and OvSwitches need to be set up working with this version)<br>


## 1st topology

### STATEMENT (GENERAL IDEA)<br>
•	Here we have a cycle. Every host can communicate with the others, but when a flood starts we enter in an infinite loop.
<img src="https://user-images.githubusercontent.com/98694899/153767602-b65255fd-3629-4aeb-96d0-10c3fbfa93dc.jpg" width="30%" height="30%">

•	A topology slicing avoids infinite loops by separating the cycle into two trees, controlled by two controllers; the two slices cannot communicate [2]<br>
•	We interconnected the two slices by adding a common root (s9); a third controller manages inter-slices communication [3]<br>
•	The provider doesn’t want the slices to send UDP packets; s9 sends the inter-slices UDP packets to a server that filters (and then drops) the packets<br>

### TOPOLOGY<br>
[4]
We realized three different slices (topology slicing):<br>
-slice1: a controller allows communication between: h1, h2, h5, h6<br>
-slice2: a controller allows communication between: h3, h4, h7, h8<br>
-connecting_slice: a controller allows non-UDP packet inter-slices transmission, and filters UDP packet to server1 and server2 (if sent by slice1 or slice2 respectively)<br>
Note: server1 and server2 are configured not to send any packet. They can only receive UDP packets that are filtered by s9<br>

### DEMO<br>
-set up environment:<br>
```vagrant up comnetsemu	#start the VM up```<br>
```vagrant ssh comnetsemu	#log into the VM```<br>

-set up the topology in mininet<br>
```sudo mn -c``` 			#flush any previous configuration<br>
```sudo python3 network.py```	#build the topology<br>

-set up the controllers<br>
	_in a new terminal_<br>
```./runcontrollers.py```		#script to run all the controllers in a single shell<br>

-create a new terminal for future flow table test<br>

-test reachability<br>
By running a ```pingall``` we obtain the following result: [5] (da fare)<br>
	
Note: ping and pingall send ICMP packets.<br>
Note: server1 and server2 never send and receive ICMP packets<br>

```h1 ping h2```	#perform ping between host 1 and host 2<br>
```h3 ping h4```	#perform ping between host 3 and host 4<br>
	_intra-slice communication works correctly_<br>

```sudo ovs-ofctl dump-flows s4```	#show s4 flow table<br>

```h4 iperf -s &```<br>
```h1 iperf -c 10.0.0.4 -t 5 -i 1```<br>

```h3 iperf -s -u &```<br>
```h1 iperf -c -u 10.0.0.3 -u -t 5 -i 1```<br>
	
```sudo ovs-ofctl dump-flows s9```	#show s9 flow table (path depending on protocol)<br>



-close and clean everything up<br>
_It’s better to flush the topology with sudo mn -c and to stop the VM with vagrant halt comnetemu_	
