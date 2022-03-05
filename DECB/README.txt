Here is the early DECB version of a web browser prototype. All it's going to do is connect to a hardcoded IP address, ask for a page and stream it to the screen. When we get to the end of the page, we won’t respond, the web server will get made at us and hang up. So, as simple a working example as I could come up with. 

To configure, edit these lines:

Local address
1120 replace decimal numbers 192 168 0 1 with IP address of your router
1160 replace decimal numbers 192 168 0 7 with an unused IP address @home

Remote 
  20 replace ‘/coco.html’ with name or path to target web page
  22 replace ‘rickLT.conect2020.org’ with the web server hostname
1250 replace decimal numbers 192 168 0 124 with IP address of web server

To read, here is a variable decoder ring. 

LOCAL VARS
   A,B,C,D,E,F,G,W$,X$ - TEMP RESULT
   I,J     - LOOP INDEX
   S     - STATUS

NATIONAL VARS
   DL       - DATA TO READ IN (obs?)
   EB       - END OF BUFFER
   NP       - NEXT POINTER (obs?)
   SG       - DATA SEGMENT SIZE
   SS       - STRING START (slicing)

GLOBAL VARS
   RB, WB   - BASE ADDRESS R,W BUFFER
   RL,WL    - DATA LENGTH
   RM, WM   - MASKOF R,W BUFFER
   WP,RP    - R,W POINTER (from 5100)
   RZ, WZ   - R,W SIZEOF DATA

and a sort of outline.

open a local socket, connect to a remote one
     1310 POKE &HFF69,&H04 : POKE &HFF6A,&H01 : POKE &HFF6B, 1   'SN-CR=OPEN SOCKET
     1320 POKE &HFF69,&H04 : POKE &HFF6A,&H03 : S = PEEK(&HFF6B) 'SN-SR<-STATUS
     ...
     1340 POKE &HFF69,&H04 : POKE &HFF6A,&H01 : POKE &HFF6B, 4 'TRY CONNECT   
     1350 POKE &HFF69,&H04 : POKE &HFF6A,&H03 : C = PEEK(&HFF6B) 'GET STATUS
     ...

make a server request ---------------------------------------------------
* this code should use the multi-segment algo in the read data routine following

compare data to write with xmit buffer, wait until there is space
     1540 WZ = LEN(W$)                                        'WZ = WRITE SIZE
     1550 POKE &HFF69,&H04 : POKE &HFF6A,&H20                 'SN-CR
     1552 F = (PEEK(&HFF6B)*256)+PEEK(&HFF6B) 
     1554 IF F < WZ THEN GOTO 1550                            'LOOP UNTIL FREE SIZE OK

get write pointer from SN_TX-RD, AND with write mask to get abs address
       40 WB=&H4000 : WM=&H7FF : WE=WB+WM                     'WRITE BASE, MASK
     1560 POKE &HFF69,&H04 : POKE &HFF6A,&H22                 'SN-TX-RD
     1570 WP = (PEEK(&HFF6B)*256)+PEEK(&HFF6B)                'WP=WRITE POINTER
     1572 B = INT(WP/256) : C = WP-(256*B)
     1574 D = INT(WM/256) : E = WM-(256*D)
     1576 F = B AND D : G = C AND E
     1580 WL = ((F*256)+G)+WB                                 'WL=WP AND MASK+BASE

set data port to absolute address just found, poke data chars until done
     1600 SG = WZ                                             'PROPOSE ALL BYTES 
     1800 REM *** WRITE DATA ***
     1810 B = INT(WL/256) : C = WL-(256*B)                    'WRITE LOC IN 2 BYTES 
     1820 POKE &HFF69,B : POKE &HFF6A,C                       'SET IT

poke each character of w$ into data port, then last used+1 to wizzy
    1830 L = LEN(W$)
    1840 FOR I = SS TO L
    1850 W = ASC(MID$(W$,I,1))
    1860 POKE &HFF6B,W
    1870 NEXT I
    1875 SS = L+1                                            ' NEXT STRING START
    1880 POKE &HFF69,&H04 : POKE &HFF6A,&H01 : POKE &HFF6B,&H20  'SN_CR=SEND IT

accept server response -----------------------------------------------------

wait for data to read
     2520 POKE &HFF69,&H04 : POKE &HFF6A,&H26                 'SN-RX-RSR
     2530 RZ = (PEEK(&HFF6B)*256)+PEEK(&HFF6B)                'RZ=READ SIZE
     2550 IF RZ = 0 THEN 2520 ELSE PRINT                      'DO UNTIL DATA 

get write pointer from SN_TX-RD, AND with write mask to get abs address
     2570 GOSUB 2900                                          'COMPUTE RP, RL
     2900 POKE &HFF69,&H04 : POKE &HFF6A,&H28 
     2910 RP = (PEEK(&HFF6B)*256)+PEEK(&HFF6B)                'RP = READ POINTER
     2912 B = INT(RP/256) : C = RP-(256*B)
     2914 D = INT(RM/256) : E = RM-(256*D)
     2916 F = B AND D : G = C AND E
     2918 RL = ((F*256)+G)+RB                                 'RP AND MASK+BASE


if data is not segmented, skip pass 1, else print it
       50 RB=&H6000 : RM=&H7FF : RE=RB+RM                     'READ BASE, MASK
     2560 SG=RZ                                               'PROPOSED=ALL BYTES
     2580 IF RZ <= RE-RL THEN GOTO 2640                       '1 SEG FITS, DO LAST
     2590 SG=RZ-RM :  RZ=RZ-SG                                'SEG = REST OF BUFFER 
     2810 B = INT(RL/256) : C = RL-(256*B)                    'READ LOC IN 2 BYTES 
     2820 POKE &HFF69,B : POKE &HFF6A,C                       'SET ADDR ON WIZNET
     loop length of segment SG
     2610 SG=RZ-SG

pass 2- get write pointer, AND with write mask,print additional segment
     2900 POKE &HFF69,&H04 : POKE &HFF6A,&H28 
     2910 RP = (PEEK(&HFF6B)*256)+PEEK(&HFF6B)              'RP = READ POINTER
     2912 B = INT(RP/256) : C = RP-(256*B)
     2914 D = INT(RM/256) : E = RM-(256*D)
     2916 F = B AND D : G = C AND E
     2918 RL = ((F*256)+G)+RB                               'RP AND MASK+BASE
     2810 B = INT(RL/256) : C = RL-(256*B)                    'READ LOC IN 2 BYTES 
     2820 POKE &HFF69,B : POKE &HFF6A,C                       'SET ADDR ON WIZNET
     loop length of segment SG                             

poke last used address +1 to wizzy, set read done
     2650 POKE &HFF69,&H04 : POKE &HFF6A,&H28                 'SN-RX-RD    
     2652 RZ=RZ+1 : B=INT(RZ/256) : C=RZ-(256*B)              '1 PAST LAST READ       
     2654 POKE &HFF6B,B : POKE &HFF6B,C            
     2660 POKE &HFF69,&H04 : POKE &HFF6A,&H01 : POKE &HFF6B,&H40 'SN-CR=READ DONE


repeat send or recv as needed ---------------------------------------------
