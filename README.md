# com-diag-candleclock

N.B. Candleclock is a work in progress.

## Copyright

Copyright 2018 by the Digital Aggregates Corporation, Arvada Colorado, USA.

## License

Licensed under the terms of the FSF GPL v2.

## Abstract

Candleclock is my implementation of a stratum-1 NTP server based on
a Raspberry Pi 3 B+ with a NaviSys GR-701W GPS receiver. The GR-701W
is pretty unique among its competitors in that it is a GPS receiver
that emits its NMEA sentences over a USB serial connection (that's
common), and generates a one hertz (1Hz) Pulse Per Second (1PPS)
indicator by toggling the (simulated) model control signal Data Carrier
Detect (DCD) (not so common).  1PPS is used to closely phase-lock the
system clock to the GPS clock.  I more or less followed Eric Raymond's
"Stratum-1-Microserver HOWTO" with a few changes here and there to
either customize it for my network or to fix a minor issue here and there
(e.g. his udev rule didn't work for me without modification).

## Operation

When the system boots up, the real-time hardware clock is read and the
Linux system clock set to that time. This provides a usable system clock
until the GPS board can establish a lock on enough satellites to compute
a solution. The NTP daemon on Hourglass will keep the Linux system
clock closely synchronized to UTC (GPST plus leap seconds) as provided
by the GPS daemon. The GNU library will handle the conversion from UTC
to local time, including any necessary Daylight Saving Time conversion,
providing the system time zone is administered appropriately. A Python
script runs in the background, polling the system clock five times a
second, and updating the LCD using the Adafruit LCD library with the
local date and time. If you press the SELECT button on the LCD board,
the Python script updates the real-time hardware clock to the system
clock, which will have been continuously synchronized via GPS and hence,
over time, more accurate than the RTC. The RTC has a battery backup so
that it maintains the time even if Hourglass is powered off.

## References

<https://www.etsy.com/listing/501829632/navisys-gr-701w-u-blox-7-usb-pps>

<http://blog.dan.drown.org/pps-over-usb/>

<http://paul.chavent.free.fr/pps.html>

<https://www.kernel.org/doc/Documentation/pps/pps.txt>

<https://www.ntpsec.org/white-papers/stratum-1-microserver-howto/>

<http://www.satsignal.eu/ntp/Raspberry-Pi-quickstart.html>

<http://www.catb.org/gpsd/gpsd-time-service-howto.html>

<http://doc.ntp.org/4.1.0/ntpq.htm>

<https://git.savannah.gnu.org/git/gpsd.git>

<https://gitlab.com/NTPsec/ntpsec.git>

<https://learn.adafruit.com/adding-a-real-time-clock-to-raspberry-pi?view=all>

<http://www.elevendroids.com/2012/12/setting-up-hardware-rtc-in-raspbian/>

<https://blog.remibergsma.com/2013/05/08/adding-a-hardware-clock-rtc-to-the-raspberry-pi/>

<https://thepihut.com/blogs/raspberry-pi-tutorials/17209332-adding-a-real-time-clock-to-your-raspberry-pi>

<https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/overview>

<https://github.com/adafruit/Adafruit_Python_CharLCD>

<https://docs.python.org/2/library/time.html>

## Contact

Chip Overclock    
<mailto:coverclock@diag.com>    
Digital Aggregates Corporation    
3440 Youngfield Street    
Suite 209    
Wheat Ridge CO 80033 USA    

## Notes

    sudo screen /dev/gpsd0 9600 8n1

    sudo modprobe configs

    cd
    cd src
    apt install git scons ncurses-dev python-dev bc
    git clone https://git.savannah.gnu.org/git/gpsd.git
    cd gpsd
    scons \
    	timeservice=yes \
    	nmea0183=yes \
    	fixed_port_speed=9600 \
    	fixed_stop_bits=1 \
    	pps=yes \
    	ntpshm=yes
    scons install

    cd
    cd src
    apt install bison libevent-dev libcap-dev libssl-dev libreadline-dev
    git clone https://gitlab.com/NTPsec/ntpsec.git
    cd ntpsec
    ./waf configure --refclock=shm
    ./waf build
    ./waf install

    sudo adduser --home /home/ntp --shell /bin/false --uid 111 --gid 65534 --disabled-password ntp

    sudo /usr/sbin/gpsd -b -n -N -D 5 /dev/gps0 /dev/pps0

    ntpq -c peer -c as -c rl

    ntpq -p candleclock

    sudo ntpd -gdn

    sudo date -u $(ssh jsloan@mercury bin/dateu)

    sudo raspi-config

    sudo apt-get install screen

    sudo apt-get install i2c-tools

    sudo apt-get install build-essential python-dev python-smbus python-pip git
    sudo pip install RPi.GPIO
    git clone https://github.com/adafruit/Adafruit_Python_CharLCD.git
    cd Adafruit_Python_CharLCD
    sudo python setup.py install

    ntpq -c peer -c as -c rl localhost
    # \b        reject (unreachable or otherwise unusable)
    # x         falseticker (intersection algorithm)
    # .	        excess
    # -	        outlyer (cluster algorithm)
    # +	        candidate
    # #	        selected (but not amongst top six)
    # *	        peer (primary reference)
    # o	        PPS peer

    cd
    mkdir src
    cd src
    git clone https://github.com/coverclock/com-diag-diminuto
    cd com-diag-diminuto/Diminuto
    make
    cd ../..
    git clone https://github.com/coverclock/com-diag-hazer
    cd com-diag-hazer/Hazer
    make

    sudo mkdir -p /var/log/ntpstats
    sudo chown ntp /var/log/ntpstats

## Example

    > lsb_release -a
    No LSB modules are available.
    Distributor ID: Raspbian
    Description:    Raspbian GNU/Linux 9.4 (stretch)
    Release:        9.4
    Codename:       stretch

    > uname -a
    Linux candleclock 4.14.34-v7+ #1110 SMP Mon Apr 16 15:18:51 BST 2018 armv7l GNU/Linux

    > cd
    > cd src/com-diag-hazer/Hazer
    > . out/host/bin/setup
    > gpstool -D /dev/gpsd0 -b 9600 -8 -n -1 -E -c
    $GPGSV,4,3,14,27,28,047,30,28,36,247,41,30,49,307,36,46,38,215,44*7F\r\n
    $GPGSV,4,3,14,27,28,047,30,28,36,247,41,30,49,307,36,46,38,215,44*7F\r\n
    MAP 2018-05-21T16:35:22Z 39*47'39.34"N,105*09'12.07"W  5564.56' N     0.049mph PPS 1
    GGA 39.794263,-105.153355  1696.100m   0.000*    0.043knots [11] 9 10 5 0 4
    GSA {   8   7   9  30  51  48  27  23  28  11   5 } [11] pdop 1.75 hdop 1.05 vdop 1.40
    GSV [01] sat   5 elv 10 azm 296 snr 19dBHz con GPS
    GSV [02] sat   7 elv 72 azm 351 snr 43dBHz con GPS
    GSV [03] sat   8 elv 56 azm  88 snr 35dBHz con GPS
    GSV [04] sat   9 elv 50 azm 186 snr 41dBHz con GPS
    GSV [05] sat  11 elv 19 azm 141 snr 29dBHz con GPS
    GSV [06] sat  16 elv  0 azm  56 snr 15dBHz con GPS
    GSV [07] sat  18 elv  5 azm 124 snr 20dBHz con GPS
    GSV [08] sat  23 elv 18 azm 162 snr 41dBHz con GPS
    GSV [09] sat  27 elv 28 azm  47 snr 31dBHz con GPS
    GSV [10] sat  28 elv 36 azm 247 snr 41dBHz con GPS
    GSV [11] sat  30 elv 49 azm 307 snr 36dBHz con GPS
    GSV [12] sat  46 elv 38 azm 215 snr 44dBHz con GPS
    GSV [13] sat  48 elv 36 azm 220 snr 39dBHz con GPS
    GSV [14] sat  51 elv 44 azm 183 snr 32dBHz con GPS

    > ntpq -c "rv 0 reftime" -c peers -c associations
    reftime=dead9d04.c219aa53 2018-05-21T19:31:16.758Z
         remote                                   refid      st t when poll reach   delay   offset   jitter
    =======================================================================================================
    *SHM(1)                                  .PPS.            0 l   12   64  377   0.0000  15.8385  10.5187
    xSHM(0)                                  .GPS.            0 l   47   64  377   0.0000 -137.548  11.7069
     us.pool.ntp.org                         .POOL.          16 p    -  256    0   0.0000   0.0000   0.0010
    +horp-bsd01.horp.io                      146.186.222.14   2 u   57   64  377  54.6031  11.4205   8.5216
    +awesome.bytestacker.com                 216.218.254.202  2 u   48   64  377  30.2116  19.7003  11.5323
    +stratum-1.sjc02.svwh.net                .CDMA.           1 u   60   64  377  39.2161   6.2334  10.0937
    +time.no-such-agency.net                 128.227.205.3    2 u   60   64  377  49.9095  22.9024  12.3353
    -208.88.126.235                          128.227.205.3    2 u   62   64   33  54.1146 -40.0667  50.9546
    +ntp2.wiktel.com                         212.215.1.157    2 u   63   64  377  55.1068  19.9537  10.3241
    +74.120.81.219                           129.250.35.250   3 u   62   64  377  30.7050  19.3142  11.4608
    +tick.no-such-agency.net                 162.213.2.253    2 u   61   64  377  40.4906  19.9575  12.0436
    
    ind assid status  conf reach auth condition  last_event cnt
    ===========================================================
      1 21602  961a   yes   yes  none  sys.peer    sys_peer  1
      2 21603  9114   yes   yes  none falsetick   reachable  1
      3 21604  8811   yes  none  none    reject    mobilize  1
      4 21605  1414    no   yes  none candidate   reachable  1
      5 21606  141a    no   yes  none candidate    sys_peer  1
      6 21607  141a    no   yes  none candidate    sys_peer  1
      7 21608  1414    no   yes  none candidate   reachable  1
      8 21609  1314    no   yes  none   outlier   reachable  1
      9 21610  1414    no   yes  none candidate   reachable  1
     10 21611  1414    no   yes  none candidate   reachable  1
     11 21612  1414    no   yes  none candidate   reachable  1


