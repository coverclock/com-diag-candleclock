# com-diag-candleclock

Candleclock is a work in progress.

## Copyright

Copyright 2018 by the Digital Aggregates Corporation, Arvada Colorado, USA.

## License

Licensed under the terms of the FSF GPL v2.

## Abstract

Candleclock is my implementation of a stratum-1 NTP server based on
a Raspberry Pi 3 with a NaviSys GR-701W GPS receiver. The GR-701W is
pretty unique among its competitors in that it is a GPS receiver that
emits its NMEA sentences over a USB serial connection (that's common),
and generates a one hertz (1Hz) Pulse Per Second (1PPS) indicator by
toggling the (simulated) model control signal Data Carrier Detect (DCD).
1PPS is used to closely phase-lock the system clock to the GPS clock.
I more or less followed Eric Raymond's "Stratum-1-Microserver HOWTO"
with a few changes here and there to either customize it for my network
or to fix a minor issue here and there (e.g. his udev rule didn't work
for me without modification).

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

    sudo modprobe configs

    scons \
    	timeservice=yes \
        magic_hat=yes \
    	nmea0183=yes \
    	prefix="/usr" \
    	fixed_port_speed=9600 \
    	fixed_stop_bits=1 \
    	pps=yes \
    	ntpshm=yes

    ./configure \
        --prefix=/usr \
        --enable-all-clocks \
        --enable-parse-clocks \
        --enable-SHM \
        --disable-debugging \
        --sysconfdir=/var/lib/ntp \
        --with-sntp=no \
        --with-lineeditlibs=edit \
        --without-ntpsnmpd \
        --disable-local-libopts \
        --enable-ntp-signd \
        --disable-dependency-tracking \
        --enable-ATOM \
        --enable-linuxcaps

    sudo /usr/sbin/gpsd -b -n -N -D 5 /dev/gps0 /dev/pps0

    ntpq -c peer -c as -c rl

    ntpq -p candleclock

    sudo ntpd -gdn

    sudo date -u $(ssh jsloan@mercury bin/dateu)

    sudo raspi-config

    sudo apt-get install i2c-tools

    sudo apt-get install build-essential python-dev python-mmbus python-pip git
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

## Example

