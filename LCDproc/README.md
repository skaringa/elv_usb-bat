# USB-BAT for LCDproc

## Prerequisite:  
 * must-have:  
``` 
> apt-get install build-essential automake autoconf gawk gnu-standards pkg-config libusb-0.1-4 libusb-dev  
```          
 * ONLY, if you use freetype2 and TTF-fonts (monospace needed), which I would NOT recommend (buildin font is better to read):  
``` 
> apt-get install libfreetype6 libfreetype6-dev ttf-mscorefonts-installer  
``` 

## get LCDproc v0.5.7  
   1) download from <http://sourceforge.net/projects/lcdproc/files/lcdproc/0.5.7/>  
   2) unpack in /usr/local/src/LCDproc (tar -xvzf lcdproc-0.5.7.tar.gz)  
  
## copy my files from github to LCDproc
 
## configure
``` 
> autoreconf -v
> ./configure --disable-libftdi \
              --disable-libhid \
              --disable-libX11 \
              --disable-libusb-1-0 \
              --disable-ethlcd \
              --disable-freetype \
              --disable-libpng \
              --prefix=/usr/local \
              --enable-drivers=glcd \
              --enable-libusb
``` 

##### Note: Enabling the debug() function only in specific files:  
   1) Configure without enabling debug (that is without --enable-debug).  
   2) Edit the source file that you want to debug and put the following line at the top, before the #include "report.h" line: #define DEBUG.  
   3) Then recompile with 'make'.  
    This way, the global DEBUG macro is off but is locally enabled in certain parts of the software.  
  
## build  
        > make  
##### Note:  added compilerflag "-std=c99". This will produce errors during compilation of other drivers (usleep)!

## test (See additional notes below!)  
``` 
> server/LCDd -c LCDd.conf  
            ==> the display is showing the status screen of LCDproc. See additional note below.
> clients/lcdproc/lcdproc -c clients/lcdproc/lcdproc.conf 
            ==> the display is showing some infos about the machine running on
> clients/lcdexec/lcdexec -c clients/lcdexec/lcdexec.conf  
            ==> custom menu is available. See additional note below.  
``` 

#### Notes:
 * check if parameter "DriverPath" in the "Server" section of 'LCDd.conf' is set according to your setup
 * when lcdexec throws a critical error: check your config file 'lcdexec.conf'.  
  
## Quick Demo of Commands:  
```  
## This assumes LCDd is running, bound to IP 127.0.0.1 on port 13666. From the board, type in:  
telnet localhost 13666 
## after it connects, type in :  
hello  
## you should receive: connect LCDproc 0.5.7 protocol 0.3 lcd wid 20 hgt 4 cellwid 6 cellhgt 8  
 
## Type in the following:  
backlight flash  
## View the LCD   
backlight blink  
## View the LCD  
backlight on  
## View the LCD  
backlight off  
## View the LCD  
backlight on  
## View the LCD  
```  
  
now let's get serious, type in the following:   
```
client_set -name Example  
screen_add myscreen  
screen_set myscreen -duration 10 -priority 2  
widget_add myscreen simplewidget string  
widget_set myscreen simplewidget 1 1 "Testing 1 2 3 !"  
## View the LCD  
widget_set myscreen simplewidget 1 2 "Testing 1 2 3 !"  
## View the LCD  
widget_add myscreen scrolling scroller  
widget_set myscreen scrolling 7 1 14 1 v 5 "1234567890abcd"  
## View the LCD  
widget_set myscreen scrolling 7 1 14 1 m 3 "  Welcome...I hope you have a pleasant day! :)  "  
## View the LCD
## Quit your LCDproc session (hex value 0x94)  
”  
## Quit your telnet session 
quit  
```

## installation  
        > make install  

* initscripts  
  Note: Consider the dependencies, LCDd has to be started before lcdpro and lcdexec!  
  Therefore I changed a line in the startup scripts of 'lcdproc' and 'lcdexec' provided with lcdproc:  
```
  # Required-Start:    LCDd $syslog $local_fs $network $remote_fs
```

  copy my scripts (debian):  
  
```sh
cp scripts/init-LCDd.debian /etc/init.d/LCDd  
cp scripts/lcdproc.debian.me /etc/init.d/lcdproc  
cp scripts/lcdexec.debian.me /etc/init.d/lcdexec
chmod u+x /etc/init.d/LCD*   
chmod u+x /etc/init.d/lcd*  
update-rc.d LCDd defaults  
update-rc.d lcdproc defaults  
update-rc.d lcdexec defaults  
```
### Make sure that parameter "DriverPath" in the "Server" section of "/usr/local/etc/LCDd.conf" is set according to your setup before firing up the server.

#### Note: CPU-use is between 1.5% and 2% (at least on my XEON E3-1220L V2 @ 2.30GHz). Energysaving will suffer from polling the device.

## usage  
    A-Key: switches the backlight on/off  
    B-Key: menu/esc - start up menu selection, escape menu selection  
    encoder wheel: up/down  
    encoder button: enter/select  
