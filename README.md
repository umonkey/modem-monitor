# Modem tools

Here I have two little scripts:

1. `bin/modem-monitor`, which uses the control connection to display (and log) runtime modem information.
2. `bin/con2`, to keep the connection up.


## Modem monitor

The script `bin/modem-monitor` finds the proper `ttyUSB` device, reads data
from it and display it.  Uses the additional port, so works well with `pppd`
etc.

Example output:

```
Path         : /dev/ttyUSB4
Device       : huawei E3131 rev. 21.158.13.00.209 IMEI=353101043024719
Device lock  : unlocked
RSSI / RSCP  : -95 / -91 dBm, ECIO = -3 dBm :), RSSI=9
SIM status   : valid, data only
Network      : MegaFon, CellID: 8160074, +sms, +cb, LTE
Operators    : no data
Connected for: 7:09:58 IP address: 100.127.194.46
Traffic      : 1490.83 mb received, 2496.47 mb sent
Speed        : 0.48 dl, 1.84 ul
Top speed    : 4.86 dl, 2.35 ul
Last error   : none
Last update  : 2020-03-14 20:16:48
```

Here you can see:

- Port being used.
- Device identification data, lock status.
- Signal quality, obtained using the best available method.
- Network status, cell ID.
- Operators (needs to be started before connecting).
- Traffic statistics, speed.
- Last update (to be sure that it doesn't just hang).

RSSI, network mode and ISP changes are logged into a separate file, which looks like this:

```
2020-06-20 21:34:27 19 9 4 25001
2020-06-20 21:34:37 20 9 4 25001
2020-06-20 21:34:47 17 9 4 25001
2020-06-20 21:35:07 20 9 4 25001
```


## Connection manager

I use a mobile connection and have two SIM cards from different operators, for
falback purpose.  They fail at times, modem hangs, other shit happens.  I got
so fucked up by managing all this crap manually on my Linux server, that I
wrote this little program `bin/con2`, which does the following:

1. Waits for a USB device with one of pre-configured ids.
2. Sends USB mode switch for devices in wrong mode.
3. Creates a symlink `/dev/modem` to the proper TTY.
4. Gets [SIM card id](https://www.imei.info/faq-what-is-ICCID/), matches it to the ISP name.
5. Runs `pppd` to connect to the selected ISP.
6. If `pppd` fails due to modem error, resets the modem: first with `AT^RESET`, if not working -- with a USB reset.
7. Reconnects with a delay of 5 seconds.

Config file is read from `~/.config/con2.toml` and looks like this:

```
valid_devices = [
    '12d1:1506',
]

switch_devices = [
    '12d1:14fe',
    '12d1:1f01',
]

[iccid_map]
"89701983875679234987" = "mts"
```

Command output:

```
$ bin/con2
2020-06-20 21:57:53,344 INFO Read config from /root/.config/con2.toml
2020-06-20 21:57:53,433 INFO Using device with usbid 12d1:1506 as /dev/ttyUSB0
2020-06-20 21:57:53,436 INFO Hard reset with: usbreset /dev/bus/usb/001/099
2020-06-20 21:57:53,440 INFO Sending AT^ICCID? to /dev/ttyUSB0
2020-06-20 21:57:53,940 DEBUG modem response: AT^ICCID?
2020-06-20 21:57:53,951 DEBUG modem response: ^ICCID: 89701983875679234987
2020-06-20 21:57:53,972 INFO Sim card 89701983875679234987 found, using ISP mts.
2020-06-20 21:57:56,179 INFO pppd: chat:  Jun 20 21:57:56 CONNECT
2020-06-20 21:57:56,183 INFO pppd: Script /usr/sbin/chat -v -f /etc/ppp/peers/mts.chat finished (pid 3994), status = 0x0
2020-06-20 21:57:56,188 INFO pppd: Serial connection established.
2020-06-20 21:57:56,191 INFO pppd: using channel 140
2020-06-20 21:57:56,194 INFO pppd: Using interface ppp0
2020-06-20 21:57:56,207 INFO pppd: Connect: ppp0 <--> /dev/modem
2020-06-20 21:57:57,192 INFO pppd: sent [LCP ConfReq id=0x1 <asyncmap 0x0> <magic 0x23159fd8> <pcomp> <accomp>]
2020-06-20 21:57:57,196 INFO pppd: rcvd [LCP ConfReq id=0x1 <accomp> <pcomp> <asyncmap 0x0> <mru 1500> <magic 0x548> <auth chap MD5>]
2020-06-20 21:57:57,199 INFO pppd: No auth is possible
2020-06-20 21:57:57,204 INFO pppd: sent [LCP ConfRej id=0x1 <auth chap MD5>]
2020-06-20 21:57:57,209 INFO pppd: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0> <magic 0x23159fd8> <pcomp> <accomp>]
2020-06-20 21:57:57,213 INFO pppd: rcvd [LCP ConfReq id=0x2 <accomp> <pcomp> <asyncmap 0x0> <mru 1500> <magic 0x548>]
2020-06-20 21:57:57,217 INFO pppd: sent [LCP ConfAck id=0x2 <accomp> <pcomp> <asyncmap 0x0> <mru 1500> <magic 0x548>]
2020-06-20 21:57:57,222 INFO pppd: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
2020-06-20 21:57:57,226 INFO pppd: rcvd [IPCP ConfReq id=0x1]
2020-06-20 21:57:57,230 INFO pppd: sent [IPCP ConfNak id=0x1 <addr 0.0.0.0>]
2020-06-20 21:57:57,234 INFO pppd: rcvd [IPCP ConfNak id=0x1]
2020-06-20 21:57:57,238 INFO pppd: sent [IPCP ConfReq id=0x2 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
2020-06-20 21:57:57,242 INFO pppd: rcvd [IPCP ConfReq id=0x2]
2020-06-20 21:57:57,247 INFO pppd: sent [IPCP ConfAck id=0x2]
2020-06-20 21:57:57,249 INFO pppd: rcvd [IPCP ConfNak id=0x2 <addr 10.212.26.107> <ms-dns1 217.66.153.254> <ms-dns2 217.66.153.250>]
2020-06-20 21:57:57,250 INFO pppd: sent [IPCP ConfReq id=0x3 <addr 10.212.26.107> <ms-dns1 217.66.153.254> <ms-dns2 217.66.153.250>]
2020-06-20 21:57:57,253 INFO pppd: rcvd [IPCP ConfAck id=0x3 <addr 10.212.26.107> <ms-dns1 217.66.153.254> <ms-dns2 217.66.153.250>]
2020-06-20 21:57:57,254 INFO pppd: Could not determine remote IP address: defaulting to 10.64.64.64
2020-06-20 21:57:57,258 INFO pppd: local  IP address 10.212.26.107
2020-06-20 21:57:57,261 INFO pppd: remote IP address 10.64.64.64
2020-06-20 21:57:57,264 INFO pppd: primary   DNS address 217.66.153.254
2020-06-20 21:57:57,266 INFO pppd: secondary DNS address 217.66.153.250
2020-06-20 21:57:57,269 INFO pppd: Script /etc/ppp/ip-up started (pid 4442)
2020-06-20 21:57:57,271 INFO pppd: Script /etc/ppp/ip-up finished (pid 4442), status = 0x0
```
