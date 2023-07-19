# My diagnosis and troubleshooting Ford F350 4R100 transmission P0705 fault
Occasionally, the transmission would disengage shortly after shifting into 3rd.
The overdrive light would blink, suggesting that the transmission had thrown a code.
The code was P0705

## P0705 - Digital TR circuit failure
This fault implies a short to ground or open circuit in one of the 4 TR signals coming
from the transmission digital range sensor to the PCM.  These signals are TR1, TR2, TR3A, and TR4.
I tried these procedures:
- Replaced and recalibrated the digital range sensor.  Checked connector contacts for corrosion or deterioration.  Symptoms remained.
- Ohmed out the wiring harness to the PCM connector.  All TR signals checked out good per the following table. Note: OEM manual wire color specs are inconsistent between diagrams.  I frankly don't know which is right.

![TR signal table]



## ELM327 OBD2 to USB-serial chip commands
