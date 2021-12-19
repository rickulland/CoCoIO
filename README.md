
<h1> The worst web wrangler </h1>

Part of a web browser written in Basic09, poking at the bare metal of CoNect's 'CoCoIO' card. You'll need one of those, working OS9, and an available network connection;-)


<B>Setup</B>

'www' depends on two data files loosely based on their Linux equivalents. Edit these and config is done.

<b>/DD/SYS/hosts</b> - Hosts is the dns of last resort, a list of IP address and hostname pairs, starting with the address of the local machine. We don’t have a dns utility yet, so follow that with additional lines of webservers & etc.  

	192.168.0.7  rickslaptop.org
	104.192.7.102 play-classics.net


<b>/DD/SYS/interfaces</b> - Interfaces describes the ethernet connection. We still don’t have dns, so ‘static’ is the only style allowed. macaddr is not burned into the hardware, set the last 3 bytes to some unique hex values of your choosing. Change phyaddr if you change the address jumper in CoCoIO 

	iface eth0 inet static
	address 192.168.0.7
	gateway 192.168.0.1
	netmask 255.255.255.0
	macaddr 5C:26:0A:01:02:03
	phyaddr $FF6B

If you haven't merged All The Things into the gfx2 module, consider it. 

	cd /dd/cmds
	rename gfx2 gfx2.org
	merge gfx2.org inkey syscall > gfx2
	
It's also handy to ball up all of the Basic09 except the main program into a 'util(date)' file. 
Those last three <i>(owend,propoff,revoff)</i> can be run if you crash out of basic while in an overlay. 

	merge GetMouse.b09 initeth.b09 stateth.b09 getdns.b09 getBookmark.b09 
 	putBookmark.b09 gotoHost.b09 owend.b09 propoff.b09 revoff.b09 > util1212.b09



You will need to start in an 80 column graphics window. If you don't have one handy use the ‘mgw’ shellscript to convert the window you're in. Then load the bits. For example:

	mgw
	basic09 #32k
	$dir        (* I never remember the file names)
	load util????.b09
	load grap???.b09
	run www



<B>Using the program. </B>

Once the program starts, choose mouse or keyboard based input for the main screen. Popup menus are text only for now. Both inputs can use the end of screen menu, but the mouse can also hit underlined links on screen. 

That first text popup will be the bookmark list. Select by number, key in a URL, or input 99 to not select anything. First run, that gets you our home page.  Enter to close the bookmark menu. Then B to bookmark this most excellent homepage.

The first 20 or so lines of a web page are displayed, followed by an end of screen menu. ENTER for the next screen. There is no way to back up one screen yet, however R Reload will start again from the top. 

You can also G Goto a previously bookmarked page. Finally, the L Links menu offers the last 10 links that have been encountered in a keyboard friendly menu. 

Here are a few workarounds for open bugs. This list should change often. 

	nothing happens after a 1 letter command is given, press enter.
	the page doesn’t appear but the menu comes back, try R Reload
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
