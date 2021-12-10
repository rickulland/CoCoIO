This post summarizes parts of the W5100s datasheet and the Uthernet manual after translation to CoCo/6x09. You’ll want to grab the Uthernet doc for extra info. Just remember, when looking at Apple code, they are swapping byte order between the CPU and the WizNet. CoCo code doesn’t need to do that. BIG ENDIAN ADDRESSING IS COMPLETELY NORMAL!!!

Another oddity, note the Wiznet docs describe it’s lowest system side address as the ‘Mode Register’. They also describe the lowest address in the internal buffer as the ‘Mode Register’, and the lowest address in each of 4 socket config sections is also the ‘Mode Register’. So don’t jump to any conclusions based on that name.

<B>Chip Addressing</B>

The CoCoIO can be jumpered to one of two base addresses, FF60 or FF70. Just swap 6 for 7, for example the WizNet section has a 4 byte I/O range:

	FF68 or FF78  -  Mode Register 
	FF69 or FF79  -  Address High  
	FF6A or FF7A  -  Address Low
	FF6B or FF7B  -  Data Port     

Writing to the external Mode Register at address &ff68 (or $ff78) controls chip startup, and reading that address returns the active mode. These bits are used:

	7 – Software reset (auto cleared after reset)
	4 – Ping block - ignore pings when using TCP/IP
	3 – PPPoE – ADSL without a router, if you’re into that
	1 – Address auto increment – strongly encouraged
	0 – Indirect bus mode – MUST be set for CoCoIO 


<B>Internal RAM</B>

CoCoIO always uses auto-increment mode to access the Wiznet internal RAM. Just  write the starting internal address to FFx9 and FFxA, then read or write the appropriate number of bytes at the data port FFxB. It’s up to you to know how big each register is and when it’s time to jump to wrap around to the start of the buffer. We’ll cover data buffer handling a bit later (evil grin).


<B>Common Registers</B>

These control the local side network setup. Many can be left at default until you know they need changing. These are a must do setup for TCP/IP: 

			    Start   Length
	Mode Register (MR)	$0  1
	Gateway Address 	$1  4
	Subnet Mask 		$5  4
	My MAC Address		$9  6
	My IP Address 		$F  4


See pg 18 of the WizNet 5100s datasheet for a complete list.


<B>Socket Registers and Connection Procedure</B>

Socket registers control up to 4 ethernet connections. Socket 0 uses internal addresses $400-$4FF, with the other three starting at $500, $600, and $700. We’ll list socket 0 only as we segue into procedural mode. 
	
Mode Register ($400) – First, define the new socket.  This register’s lower four bits define the socket type:

	  0  Closed
	  1  TCP
	  2  UDP
	  3  IP Raw
	  4  MAC Raw  (socket 0 only)
	  5  PPPoE (Socket 0 only

and the upper 3 bits optionally enable certain features:

	 32  disable delayed ACK (TCP) or set IGMP1 (multicast)
 	 64  promiscuous mode (Socket 0, MAC raw mode only)
	128  enable multicasting (UDP sockets only)

Command Register ($401) – Next write to the Socket Command Register to make it do something.

	1   Open
	2   Listen (wait for SYN)
   	4   Connect (send SYN)
   
Status Register ($403) - Finally, check the socket Status Register to see what happened.

	00	Closed
	13	TCP opened
	14	TCP Waiting for peer
	17	TCP connected to peer
	1C	TCP got disconnect request
	22	UDP opened
	32	IPRAW opened
	42	MACraw opened


<B>Receiving Data and Data Buffering</B>

With the socket opened, it’s time to move some bytes. Thw WizNet has 16K of internal data space, $4000-5FFF for Transmit buffers and $6000-7FFF for receive. By default, this is split into 2K x 4 sockets xmit and 2K x 4 sockets receive, but you can change that. Whatever the size, the 8 buffers share a common address range. You must provide a starting address and check your work with a mask byte (buffer size -1) for each buffer.

The W5100 does not automatically manage buffer rollover. It's up to you to wrap around, and to reset the read pointer location to reflect the number of 'auto-inc' reads made from the dataport. So a receive loop would be:

	1. Use IRQ or poll the Received Size Register. When >0 ...
	2. Get read or write pointer, AND with buffer mask (buff size -1) 
	   then add buffer base address. This is the start location.
	3. Get rx/tx data size, subtract this from end of buffer. 
	   This is sizeof first segment. Read that many bytes from the dataport.
	4. Reset the rx/tx pointer to the start of the buffer 
	   and read the remaining bytes from the dataport. 
	5. Set the read pointer one address past the last read address
	6. Send RECV ($40) to the socket command register

<B>close the connection</B>

Only two registers, and the TCP/IP connection is closed. 

	Command Register ($401) – Set to $08 (DISCON)
	Status Register ($403) – loop until = $18 (SOCK_FIN_WAIT)



<h2>Example Program</h2>

To illustrate all this, we have part of a web browser in Basic09. Get the code from https://github.com/rickulland/CoCoIO


<B>Setup</B>

The ‘worst web wrangler’ depends on two data files loosely based on their Linux equivalents. Edit these and config is done.

<b>/DD/SYS/hosts</b> - Hosts is the dns of last resort, a list of IP address and hostname pairs, starting with the address of the local machine. We don’t have a dns utility yet, so follow that with additional lines of webservers & etc.  

	192.168.0.7  rickslaptop.org
	192.168.0.8  anotherhost.com


<b>/DD/SYS/interfaces</b> - Interfaces describes the ethernet connection. We still don’t have dns, so ‘static’ is the only style allowed. macaddr is not burned into the hardware, set the last 3 bytes to some unique hex values of your choosing. Change phyaddr if you change the address jumper in CoCoIO 

	iface eth0 inet static
	address 192.168.0.7
	gateway 192.168.0.1
	netmask 255.255.255.0
	macaddr 5C:26:0A:01:02:03
	phyaddr $FF6B

If you haven't merge All The Things into the gfx2 module, consider it. 

	cd /dd/cmds
	rename gfx2 gfx2.org
	merge gfx2.org inkey syscall > gfx2
	
It's also handy to ball up all of the Basic09 except the main program into a 'util(date)' file 

	merge GetMouse.b09 initeth.b09 stateth.b09 getdns.b09 getBookmark.b09 
 	putBookmark.b09 gotoHost.b09 owend.b09 propoff.b09 revoff.b09 > util1212.b09

Those last three (owend,propoff,revoff) can be run if you crash out of basic while in an overlay. Because you will need to start in an 80 column graphics window. If you don't have one handy use ‘makegw’ to convert the window you're in. Then load the bits. For example:

	makegw
	basic09 #32k
	$dir  #(I never remember the file names)
	load util1212.b09
	load www1212.b09
	run www



<B>Using the program. </B>

Once the program starts, choose mouse or keyboard based input for the main screen. Popup menus are text only for now. Both inputs can use the end of screen menu, but the mouse can also hit underlined links on screen. 

That first text popup will be the bookmark list. Select by number, key in a URL, or input 99 to not select anything. Enter to close the bookmark menu.

The first 20 or so lines of a web page are displayed, followed by an end of screen menu. <ENTER> for the next screen. There is no way to back up one screen yet, however <R>eload will start again from the top. 

You can also Bookmark the current page or Goto a previously bookmarked page. Finally, the Links menu offers the last 10 links that have been encountered in a keyboard friendly menu. 

Here are a few workarounds for open bugs. This list should change often. 

	nothing happens after a 1 letter command is given, press enter.
	the page doesn’t appear but the menu comes back, try <R>eload
	she’s dead, Jim. ESC then ‘run www’
	crash to basic inside a menu, type ‘run owend’ then ‘run www’



<B>Digging in:</B>

This part is coming RSN… 

	In most modules, setting debug:=TRUE provides additional output
	Search for the string “ DB “ for any additional debug help 
	Note ‘subtotal’ assignments are indented in the code

Utility modules:
	
	get/putBookmark	- T&M menus for bookmarks and typed in URLs
	getdns		- has no dns, only looks at a /SYS/hosts file
	initeth,stateth	- initialize and debug the local connection
	getMouse,setMouse	- set up or read the mouse
	www			- main exe includes alpha features
