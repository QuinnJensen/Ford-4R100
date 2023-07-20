# My diagnosis and troubleshooting Ford F350 4R100 transmission P0705 fault
Occasionally, the transmission would disengage shortly after shifting into 3rd.
Then the overdrive light would flash, suggesting that the PCM had thrown a code.
The code read out was P0705.

This PCM was "chipped" almost 20 years ago, with the intermittent symptoms surfacing only in the last 2 years.  The current transmission, a factory rebuilt, was installed about 8 years ago.  Current truck odometer is about 360,000 miles.

## DTC code P0705 - Digital TR circuit failure
This fault implies a short to ground or open circuit in one of the 4 TR signals coming
from the transmission digital range sensor to the PCM.  These signals are TR1, TR2, TR3A, and TR4.  There is some redundancy in the encoding, so the PCM can apparently detect inconsistencies that imply a short or an open signal among the 5 signals (a flakey ground might be the worst case).
I tried these procedures:
- Replaced and recalibrated the digital range sensor.  Checked connector contacts for corrosion or deterioration.  Re-seated DTR sensor and PCM connectors.  Symptoms remained.
- Ohmed out the wiring harness to the PCM connector.  Wiggle tested harness wires at both ends while checking conductivity.  All TR signals checked out good per the table below. Note: OEM manual wire color specs are inconsistent between diagrams.  I frankly don't know which is right, since the wires are quite old and the colors are faded.
- Disconnected batteries for a time to reset PCM.  Symptoms remained.
- Sent PCM in for repair.  No issues found.  Firmware reflashed and upgraded.  Factory default tuning restored ("chipped" tuning discarded).
- Before re-installing the PCM, tapped into the 5 TR signals at the PCM for measurements on the fly and in-circuit to determine which signal goes awry intermittently.
- Reinstalled PCM. **Symptoms gone**.  Could there have been bit-rot in the 24-year-old firmware?  Some corner case pathology in the chipped tuning that became an issue with aging components or transmission?  Different placement of PCM wiring harness mitigating an intermittent a dirty, open or short circuit?

![TR signal table](https://github.com/QuinnJensen/Ford-4R100/blob/main/DTR-sensor-table.jpg)

## ELM327 OBD2 to USB-serial chip commands to check TR and TR_D PIDs
These PIDs are monitored because one deep scan suggested that in the failure mode, the PCM was reporting that the range sensor was reporting "reverse" through "neutral" and "drive" selections.

### PID 2211B6 - TR (transmission selector position input status)
```
at e1 (turn on echo)
at l1 (turn on line feeds)
at h1 (turn on headers)
at sp 1 (set protocol to J1850 PWM - 1999 Ford)
at sh c4 10 f1 (set header: priority c4, target address 10, return address f1)
22 11 b6 (TR PID)
(repeat for each gear selector position)
64 F1 10 62 11 B6 0E ss (park) (ss = checksum, not recorded)
64 F1 10 62 11 B6 0C ss (reverse)
64 F1 10 62 11 B6 0A ss (neutral)
64 F1 10 62 11 B6 08 ss (drive od on)
64 F1 10 62 11 B6 06 ss (drive od off)
64 F1 10 62 11 B6 04 ss (2)
64 F1 10 62 11 B6 02 ss (1)
```

### PID 2216B5 - TR_D (transmission selector position input status - digital)
```
22 16 b5 (TR_D PID)
(repeat for each gear selector position)
64 F1 10 62 16 B5 00 ss (park) (ss = checksum, not recorded)
64 F1 10 62 16 B5 0C ss (reverse)
64 F1 10 62 16 B5 06 ss (neutral)
64 F1 10 62 16 B5 0F ss (drive - overdrive switch doesn't affect)
64 F1 10 62 16 B5 09 ss (2)
64 F1 10 62 16 B5 03 ss (1)
```

## Derived Scangauge II X-Gauge commands for TR PIDs

### TR - PID 2211B6 - Derived gear selector position
- 7 = Park
- 6 = Reverse
- 5 = Neutral
- 4 = Overdrive
- 3 = Drive
- 2 = 2
- 1 = 1
```
txd c410f12211b6
rxf 0462 0000 0000
rxd 3403
mth 0001 0001 0000
nam <TR
```

### TR_D - PID 2216B5 - direct TR1-4 signals from range sensor
- 0000 = Park
- 000C = Reverse
- 0006 = Neutral
- 000F = Drive, Overdrive
- 0009 = 2
- 0003 = 1
```
txd c410f12216b5
rxf 0462 1000 0000
rxd 3404
mth 0001 0001 0000
nam TRD
```

### GEAR - PID 2211B3 - transmission actual gear status
- 4 = Overdrive
- 3 = Drive
- 2 = 2
- 1 = 1 (also shown for Park, Neutral)
```
TXD C410 F122 11B3
RXF 0462 0000 0000
RXD 3007 
MTH 0001 0001 0000
NAM GEA
```

### CCS, SS1, SS2 - PID 221105 - Coast clutch solenoid, Shift solenoid 1, 2

- "the shift solenoids provide gear selection of 1st thru 4th by controlling the pressure to the shift valves"
- 0x10 = shift solenoid 1 on
- 0x20 = shift solenoid 2 on
- 0x40 = shift solenoid 3 on (not present in 4R100)
- 0x80 = coast clutch solenoid on - coast clutch provides engine braking below 4th gear (overdrive). If engaged in 4th gear, transmission will be damaged.
```
gear   ss1     ss2     css     tcc     221105
-----------------------------------------------
prk/neu 1       0       0       0       0x10
rev     1       0       0       0       0x10
od-1    1       0       tcs     1/0     0x10,0x90 (tcs on)
od-2    1       1       tcs     1/0     0x30,0xB0 (tcs on)
od-3    0       1       tcs     1/0     0x20,0xA0 (tcs on)
od-4    0       0       0       1/0     0x00,0x80 (only when tcs off)
man-2   1       1       1       1/0     0xB0
man-1   1       0       1       0       0x90
-------------------------------------------------
tcc = torque converter clutch solenoid - set by PCM when conditions are right
tcs on = overdrive disabled, "OFF" illuminated
```
#### X-Gauge programming for PID 221105
```
TXD C410 F122 1105
RXF 0462 1000 0000
RXD 3008
MTH 0001 0001 0000
NAM SOL
```


