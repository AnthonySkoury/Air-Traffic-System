# If tag is shown then readme is incomplete (90% complete)

Air-Traffic Dump1090 Receiving & Parsing Script
===
[![Python Version](https://img.shields.io/badge/Python-3.7-brightblue.svg)](https://python.org)
[![Dump1090 Version](https://img.shields.io/badge/Dump1090-black.svg)](https://github.com/antirez/dump1090)

Purpose
---

To receive ADS-B data from each aircraft in the vicinity of the scan radius.



In the dump1090 directory being utilized, run the command:
---

./dump1090 --interactive --net

***Before running command, ensure the RTL-SDR module is connected through USB to the Raspberry Pi and is detected***

As the script is currently operating in Python 3.7, the command utilized to run the script is:
---

python3 data_acquisition.py

***Run this command in a separate terminal window as Dump1090 interactive is running***

Functions
---

* Pulls JSON request from Dump1090 stream (currently from localhost)

* Parses desired information from datastream into format to be stored in a chosen backend server (identification, location, movement)

* Organize timestamp into a form that can be attached to the data

What data is stored
---

* ICAO (International Civil Aviation Organization) Tag - A unique 24 bit transponder code specifically assigned to each aircraft for monitoring aircraft

* Squawk

* Callsign

* Latitude - Latitudinal position of aircraft

* Longitude - Longitudinal position of aircraft

* Altitude - Elevation of aircraft

* Ground Speed - Speed of aircraft in reference to the grounds 

* Timestamp

Notes
---

* Dump1090 original repository was modified to include timestamp feature in GMT time

* Optimization from serial to parallel to be determined



Acknowledgments
---
Thank you to the creators for Dump1090 as well as the Python Requests library
