
<h1> The worst web wrangler </h1>

Part of a web browser written in Basic09, poking at the bare metal of CoNect's 'CoCoIO' card. You'll need one of those, working OS9, and an available network connection;-). Nowadays, you'd like a hires mouse also. 

<B>Setup</B>

'www' depends on two data files loosely based on their Linux equivalents. Edit these and config is done.

<b>/DD/SYS/hosts</b> - Hosts is the dns of last resort, a list of IP address and hostname pairs, starting with the address of the local machine. We don’t have a dns utility yet, so follow that with additional lines of webservers & etc. The 2nd and 3rd lines are a good start!  

	192.168.0.7  rickslaptop.org
	104.192.7.102 play-classics.net
	206.163.230.108 www.lcurtisboyle.com 
	

<b>/DD/SYS/interfaces</b> - Interfaces describes the ethernet connection. We still don’t have dns, so ‘static’ is the only style allowed. 
macaddr is not burned into the hardware, set the last 3 bytes to some unique hex values of your choosing. 
Change phyaddr if you change the address jumper in CoCoIO 

	iface eth0 inet static
	address 192.168.0.7
	gateway 192.168.0.1
	netmask 255.255.255.0
	macaddr 5C:26:0A:01:02:03
	phyaddr $FF6B


The Basic09 utilities <i>(all b09 except the main routine 'www')</i> don't change very often, ball them into one file.  I will put the wad someplace. 

The optional utils <i>(owend,propoff,revoff,undoff)</i> can be run if you crash out of basic while in an overlay.

	merge  drawTable.b09 getdns.b09 getBookmark.b09  gotoHost.b09 initeth.b09 
 	putBookmark.b09 stateth.b09 owend.b09 propoff.b09 revoff.b09 undoff.b09 > wwwutil.b09

<B>Using the program. </B>

Once the program starts, choose mouse or keyboard based input for the main screen. Keyboard uses an end of screen menu, mouse has dropdowns or hotkeys <i>(ALT-letter)</i>. Underlined links on screen work, to avoid scrolling the <L>inks menu has the last 10 links. Popup menus are still text only. Coming soon. 

Anyway, that first text popup will be the bookmark list. Select by number, key in a URL, or input 99 to not select anything and get our home page.  Enter to close the bookmark menu. If you typed in a page, use B to bookmark it now...

The first 22 or so lines of a web page are displayed, with mouse or end of screen menu. ENTER for the next screen. There is no way to back up one screen yet, however R Reload will start again from the top. 

You can also G Goto a previously bookmarked page. Finally, the L Links menu offers the last 10 links that have been encountered in a keyboard friendly menu. 

Here are a few workarounds for open bugs. This list should change RSN. 

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
	
	drawTable		- create string array and display popup columns
	get/putBookmark		- overlay menus for bookmarks and typed in URLs
	getdns			- has no dns skills, only looks at a /dd/SYS/hosts file
	initeth,stateth		- initialize and debug the local connection
	www			- main exe 
