# CoCoIO
What we have for now is basic text formatting - lines, headings, and paragraphs presented in 80x23 pages, with bold, reverse and alt font body text. Mouse and text menus, one at a time because I can’t get inkey and syscal to work inside the same loop. Maybe someday? 

Setting and loading bookmarks works, I forgot delete. In page links and downloads are unfinished but quasi-working. Something goofy in the page reload scheme, esp. with typed in URLs.

I want to define the interface with and between modules to they can be upgraded individually. The ‘www’ exec will be broken up some more once new functionality is defined and we can depend on a stable interface. 

You will need to merge All The System Things, in other words

    cd /dd/cmds
    
    rename gfx2 gfx2.org
    
    merge gfx2.org inkey syscall >gfx2
  
you may also want to merge everything except www.b09 into one glob, the utilities do not change as often as www.b09.

* In most modules, search for “debug” and set debug:=TRUE for additional output
* Search for the string “ DB “ for any specific debug help 
* less significant variable assignments are indented and use generic names 
  (int1, etc)
