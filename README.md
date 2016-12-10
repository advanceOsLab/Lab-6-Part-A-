# Lab_6  PART A 



** ## General Stuff **
- We use the user mode of the Networking stack of Qemu which doesn't require root privilege. 
- The NIC is virtual as well, as Qemu emulates the E1000 intel card. 
- Qemu.pcap file could be used for depugging by either tcpdump or wireshark. 
- Since we are using emulator, we can make the emulated hardware report to us using make E1000_DEBUG.
- QEMU's E1000 implementation in hw/e1000.c in the qemu directory. 
- For us, lwIP is a black box that implements a BSD socket interface and has a packet input port and packet output port. 
- **Lwip** is open source and it is suitable to be used for embedded applications where the resources such as memory are limited   such as only 10 kbytes of RAM. Protocols supported: IP, ICMP, UDP, TCP, IGMP, ARP, PPPoS, PPPoE. Addon applications: HTTP      server, SNTP client, SMTP client, ping, NetBIOS nameserver 
- The same as everything in Unix is a file; in JOS as well, sockets are considered files. 
The ns env. Creates a thread for each request, so that it can serve multiple requests, especially that some of these requests can block. 

** # Files : **
- nsipc.c : This file has the functions that will use ipc to communicate with the network server env. Including send/receive and accept/bind/listen to the socket.  All of these functions will call nsipc() and pass their specific request. nspic() will send a request using ipc to the ns env. And will be waiting for the response. 
- sockets.c : the normal user env. will use the functions in this file which then will call the corresponding function in nsipc.c
- The server : serv.c : For example, waiting fot the transmit request from the nsip, this file will create a new thread and serve the request by forwarding it the lwip. 
In the umain, we will run several programs/environments usign forking such as the input/output/timer. 





** # Other useful information**
** # Thread vs process : ** 
The typical difference is that threads (of the same process) run in a shared memory space, while processes run in separate memory spaces.
Each process has at least one thread. 
Thread context: it has it own register, kernel stack, user stack… etc
Threads have almost no overhead; processes have considerable overhead.
New threads are easily created; new processes require duplication of the parent process.

The whole picture:
The normal user env. Uses the functions in sockets.c to bind/listen/send/receive .. These functions will call one of the functions in nsipc.c which will send a request through ipc to the ns enviorment. 
The ns environment will be created by init.c, and will be running in a while loop, right? So it will be waiting for data sent to it by executing ip_rcv. After it receives the request, it will create a new thread and make it use serve_thread() to execute one of the lwip functions associated with that request. 
The lwip also sends ipc requests, for example to send the packet to the output env. For example, the below snippet is from jif.c in the lwip directory. In other words, the lwip has been costomized or some stuff has been added to it to work with the JOS; there is also jos sub directory in lwip directory. 

E1000.c is the device driver. 
The output env. Receives the ready to transmit packet from the lwip through IPC, then it forwards it to the device driver (e1000.c)




Exercise 1: 
Time.s already implemented functions to increment a counter time_tick(), this function should be called each time the hardware clock timer fires. Then there is time_ms() to give us the time in ms. note that this time is a relative time and started since time_init() was called which will initiate the counter ticks to zero. 
After this exercise we have a syscall that will give us the time in ms. 
Exercise 2 




The NIC has buffer fifo of 64KB. 
The NIC also has a transceiver, and several encodings such as Manchester.
The NIC can drop packets if its fifo buffer is full, but it will have to let the system know through the appropriate statistics registers. 
Memory buffers pointed to by descriptors store packet data. Hardware supports seven receive buffer sizes, but the maximum is 2 kB [ aka the ring buffer] . Buffer size is selected by bit settings in the Receive Control register (RCTL).
There is specific receive descriptor, so that the device driver/kerenl can understand the address where the data is stored, the packet length … etc. 


Receive descriptor status field :
DD bit : whether this descriptor has been used or not. 
EOP : it means this descriptor contains the end of the packet in case the packet traverse through more than 1 deriptor. 
It also has other status, such as ip and tcp checksum if they are enabled. There is also an error field. 
The size of Tx and Rx descriptors are 16 bytes each. 
Ring buffer: the OS takes from the tail, while the HW puts at the head and then increment it.  Head = tail, means the ring buffer is empty. 
The receive descriptor ring is described by the following registers: 
• Receive Descriptor Base Address registers (RDBAL and RDBAH) ⇒ 2 registers to store 64 bit addresses, but here we need 32bit only, so we use RDBAL only. 
Receive Descriptor Length register (RDLEN) This register determines the number of bytes allocated to the circular buffer. This value must be a multiple of 128.
• Receive Descriptor Head register (RDH) ⇒ offset from the base.  RDT is similar but for the tail. 


Receive Interrupts :-
: • Receiver Timer Interrupt (ICR.RXT0) • 
Small Receive Packet Detect (ICR.SRPD) •
 Receive Descriptor Minimum Threshold (ICR.RXDMT0)  ⇒ warning to avoid overrun
 Receiver FIFO Overrun (ICR.RX0)


Receive Interrupt Delay Timer / Packet Timer (RDTR) : initilized its time a packet received, to reduce initiating a lot of packets during bursts.


Receive Interrupt Absolute Delay Timer (RADV) The Absolute Timer ensures that a receive interrupt is generated at some predefined interval after the first packet is received. So an interrupt will be raised in case of high traffic. 




The NIC can check for valid IP/TCP headers and their checksum, not necessarily the addresses though.


Transmit Descriptors : The legacy one!


// To select legacy mode operation, bit 29 (TDESC.DEXT) should be set to 0b. 





 If a TX descriptor have the RS bit in the command byte set (TDESC.CMD), then the DD bit in the status field (TDESC.STATUS) is written when the hardware processes them.


The Tx descriptor has a command byte CMD  and status field STAT
In the command byte: 
The extended bit  DEXT is there which used to choose legacy or extended layout. 
The RS [report status] : when this set, it will provide status in the status field, for example, the HW will set the DD bit in the status field when the transmitted packet put into the NIC FIFO. 
The EOP (end of packet) is in the CMD field as well. 
Status field :
DD (Descriptor done)


Similar to the RX part, we have TDBAL, TDBAH, TDLEN, 


The OS will write at the current position of the tail, then increment it by 1. So the current value of the tail is the free descriptor where the OS should write. 
The hardware will increment the head, right?


Transmit Interrupts 
Hardware supplies three transmit interrupts.
• Transmit queue empty (TXQE) — 
• Descriptor done [Transmit Descriptor Write-back (TXDW)] — Set when hardware writes  back a descriptor with RS set. 
-- beside other couple interrupts. 




PCI:
Mandatory PCI registers : include device ID, vendor ID, stauts reg, command reg and 6 base addresses, besides the interrupt pin and the interrupt line.




Important Registers:
Transmit Control Register TCTL :-- it has bits such as 
Transmit Enable: 1 to enable, 0 for disable [ even for packets already in the device’s FIFO]
Pad Short Packets : make the packets at least 64  bytes long
Collision detection : how many times the NIC should try to retransmit before given up. [ bit 4 to 11] 
Collision distance: how often retransmits should be performed for the packets that suffered collision. 


IPG (Inter Packet Gap) register:
A small pause sometimes is required before sending packets, which allows  devices to prepare for the receive of the next packet.


Transmit Descriptor Base Address Low TDBAL (03800h)
Transmit Descriptor Base Address High TDBAH 
Transmit Descriptor Length TDLEN : This register determines the number of bytes allocated to the transmit descriptor circular buffer. This value must be a multiple of 128 bytes (the maximum cache line size). Since each descriptor is 16 bits in length, the total number of receive descriptors is always a multiple of eight.
Transmit Descriptor Head TDH : Hardware controls this pointer. The only time that software should write to this register is after a reset.
Transmit Descriptor Tail TDT:  It holds a value that is an offset from the base,
This is the location where software writes the first new descriptor. Software writes the tail pointer to add more descriptors to the transmit ready queue. Hardware attempts to transmit all packets referenced by descriptors between head and tail.
Packet Buffer Allocation ⇒ a space to store the packets [not the descriptors, right?] for both tx and rx,  each could be up to 64 kB i guess. 






RX registers:
Receive Control Register RCTL (00100h): Receiver Enable bit, Long Packet Reception Enable : otherwise packets > 1522 will be discarded.,  Receive Buffer Size: the buffer size for each descriptor (not the size of the descriptor though), the initial default value 0x0 gives you 2048 bytes.
 Receive Descriptor Base Address Low RDBAL, 
Receive Descriptor Length RDLEN 
Receive Descriptor Head RDH ⇒ Don’t touch, it is for the harware. The initial default value is 0 .
RDT :  Software writes the tail register to add receive descriptors to the hardware free list for the ring.
























Others:
The size chosen for the head and tail registers permit a maximum of 64 K descriptors, or approximately 16 K packets for the transmit queue given an average of four descriptors per packet.
As for receives, the number of on-chip transmit descriptors buffer space is 64 descriptors. 
Most of the registers in the Ethernet controller are defined to be 32 bits


The linux pads the packets to be at least 60 bytes long [maybe]


How to check if the ring buffer for descriptors is full?
In Tx, check the DD bit, or head = tail ? 




The initialization should be done as in chapter 14 of the NIC manuale 
















For the JOS to be able to network, we need: a driver, a network stack, and a network server. 
We have no hardware, so Qemu will emulate the NIC. However, we need a router/network to test our system, so Qemu also provides a virtual router, I also think Qemu runs a server as well. 


Since the address assigned is not a real IP address, we can connect to JOS as a server, because the server will need a real IP address, so instead we connect JOS to a server running on Qemu.


The JOS server will run on 2 ports: echo and HTTP. 



Pcap : stands for packet capture.  Unix-like systems implement pcap in the libpcap library; Windows uses a port of libpcap known as WinPcap.


$ tcpdump -XXnr qemu.pcap
-n : Don’t change the ports/ip addresses into names. 
-r : means read from the file (specified above) that was created by -w option earlier. 
-XX: means print the payload along with the headers, as well as the link layer header. 


The benefit of using Emulated Hardware is that it can report and help us in debugging which is real hardware normally cannot do. 


Bare metal: computer without operating system. 


Debugging:
Use the  E1000_DEBUG flag  , Note that this will only work on the Qemu provided by MIT, bcz if u look at the Qemu’s emulation of the NIC, u will find this flag. So I think we will not find this flag in the normal version of Qemu. 



The Network stack for this lab is a block box provided by lwip.


The driver is in the kernel, the interaction between the driver and the NIC is done by DMA to send/rx from ring buffers; and Memory mapped I/O for reading/writing to the registers. 


Memory mapped I/O vs port mapped I/O:-
Port mapped is has a separate part of the memory and accessed by in and out instructions. 
While memory mapped are assigned to the same program memory address space and can accessed by pointers, arrays, structures … etc. 
The above if from the microprocessor/microncontroller view, while both methods are idetical for the device such as NIC. 


Volatile : used with MMIO, bcz we don’t want the compiler optimization to store the variable… as it might change at the device at any moment, of course asynchronously with the code. 


Note: the lwIP is in the user space (at least in our use of it).


In init.c, we created a Network server environment as we have created a file system environment. 


The size of nsipcbuff is one page. 

lib/nsipc.c file contains the necessary ipc to connect/bind socket, send,receive ...etc. Where all of these functions sends requests to the network server environment via IPC interface. 




The select() system call tells you whether there is any data to read on the file descriptors that you're interested in. for example, to know whether a read operation on the file descriptor will block or not.


If you execute read() on a file descriptor — such as that connected to a serial port — and there is no data to read, then the call will hang until there is some data to read. Programs using select() do not wish to be blocked like that.


 $ make INIT_CFLAGS=-DTEST_NO_NS run-testtime  ⇒ if this flag passed, don’t create NS environment. 



The E1000 is a complicated device with a lot of advanced features, we will be using only the very basic things though. 


The NIC has a FIFO buffer. This buffer could be used for retransmission if the fault was on the wire such as collision. 


The NIC has a flash and EEPROM, probably to store some configuration. 




PCI  Peripheral Component Interconnect is a bus for attached hardware devices to a computer 
On boot, the OS will detect which devices are connected to the bus and how much memory and I/O they need, then it would assign to them. It also reads some info such as vendor and device IDs to help the OS choose the right driver. 


The CPU and the PCI devices need to access memory that is shared between them. This memory is used by device drivers to control the PCI devices and to pass information between them. Typically the shared memory contains the configuration header : which has control and status registers for the device. It also has interrupt pin related to the device : A, B, C … and an interrupt line which will let the OS knows which handler to run when this interrupt occur. Besides the base register. 
The device will tell the CPU through its registers what kind of space it want: PCI memory or PCI I/O. then the cpu/OS puts all 1s in that register, when it reads again, the device will put the size it wants. 









The device id, vendor id, base registers are all registers. There are 6 base registers.











………………..
3.2 Packet Reception:  In the general case, packet reception consists of recognizing the presence of a packet on the wire, performing address filtering, storing the packet in the receive data FIFO, transferring the data to a receive buffer in host memory, and updating the state of a receive descriptor. 
3.2.1 Packet Address Filtering: Hardware stores incoming packets in host memory subject to the following filter modes. If there is insufficient space in the receive FIFO, hardware drops them and indicates the missed packet in the appropriate statistics registers.


Memory buffers pointed to by descriptors store packet data. Hardware supports seven receive buffer sizes: probably the driver will take care of that if we changed a value using ethtool. 


The Circular ring buffer:- 
 HARDWARE OWNS ALL DESCRIPTORS BETWEEN [HEAD AND TAIL]. Any descriptor not in this range is owned by software.


There are several registers: such as the rx ring head, tail, the ring base address and its length. 


There is a timer in NIC to say how often HW interrupts should be raised for rx packets. 


The packet Timer is started counting when the last byte is written to the buffer, but it can be disabled. The packet timer is restarted each time a new packet is received, but there is also an absolute timer. 


The Absolute Timer ensures that a receive interrupt is generated at some predefined interval after the first packet is received. The absolute timer is started once a packet is received and transferred to host memory (specifically, after the last packet data byte is written to memory) but is NOT reinitialized / restarted each time a new packet is received. When the Absolute Timer expires (no receive interrupt has been generated for the amount of time defined in RADV) the Receive Timer Interrupt is generated. 


The Absolute Timer is reinitialized (but not started) when the Receive Timer Interrupt is generated due to a Packet Timer expiration or Small Receive Packet Detect Interrupt.






Reasons of interrupt:-
The presence of new packets is indicated by the following: • Absolute timer (RDTR) — A predetermined amount of time has elapsed since the first packet received after the hardware timer was written (specifically, after the last packet data byte was written to memory); this also flushes any accumulated descriptors to memory. Software can set the timer value to 0b if it wants to be notified each time a new packet has been stored in memory. 
• Receive Descriptor Minimum Threshold (ICR.RXDMT) The minimum descriptor threshold helps avoid descriptor underrun by generating an interrupt when the number of free descriptors becomes equal to the minimum. It is measured as a fraction of the receive descriptor ring size.
 • Receiver FIFO Overrun (ICR.RXO) FIFO overrun occurs when hardware attempts to write a byte to a full FIFO. An overrun could indicate that software has not updated the tail pointer to provide enough descriptors/buffers, or that the PCI bus is too slow draining the receive FIFO. Incoming packets that overrun the FIFO are dropped and do not affect future packet reception.


Unlike the TCP checksum, the UDP checksum is optional. Software must set the TXSM bit in the TCP/IP Context Transmit Descriptor to indicate that a UDP checksum should be inserted. Hardware does not overwrite the UDP checksum unless the TXSM bit is set.


Three specific types of checksum are supported by the hardware in the context of the TCP Segmentation offload feature: 
• IPv4 checksum (IPv6 does not have a checksum) 
• TCP checksum
 • UDP checksum
