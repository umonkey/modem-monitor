# RSSI monitor

This is a python script that monitors your USB modem status.
Displays signal strength, data flow, network status and other options.
It uses the additional command port and doesn't block the normal modem work.

Usage is, in any order:

- Connect to the internet.
- Run this script.


## Detecting the modem

It looks up in `/sys/bus` for something that uses the option1 driver.  You know a better way?  Tell me.


## Example output

```
Path         : /dev/ttyUSB4
Device       : huawei E3131 rev. 21.158.13.00.209 IMEI=353101043024719
Device lock  : unlocked
RSSI / RSCP  : -95 / -91 dBm, ECIO = -3 dBm :), RSSI=9
SIM status   : valid, data only
Network      : searching, CellID: 8160072, +sms, +cb, WCDMA / HSPA+
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
