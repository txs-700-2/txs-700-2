# Information on TXS-700-2 Wrist Tag

This project aims to gather publicly available information and observations about the TXS-700-2 Wrist Tag, branded ElmoTech Ltd / 3M Electronic Monitoring Inc.

This information is made available for research purposes only. This project is not endorsed by the creators of the device.

## Public details

* FCC ID: LSQ-TXS-700-2
* FCC documents: https://fccid.io/LSQ-TXS-700-2
* Operational description: 433.92 MHz carrier with FSK data, 5 mSec transmit at pseudo-random 18–22 s intervals.
* Message contents: identifier and status code.
* Test data: TX Period 5.2 ms, peak at 433.8825 MHz (433.92 MHz - 38.5 kHz), bandwidth 232 MHz.
* Details from the images in FCC documents: CC1050 transmitter. 2 pins to each metal strap.

## Ideas

* 2 pins connected to the straps can be short circuited, allowing the strap to be removed.
* Transmitter protocol can be reverse engineered.
* Range extender: Battery-powered device could be used to monitor the tag, with another device to retransmit the data at home.
* Complete overhaul: The tag could be bricked, allowing continuous use of another device to transmit generated (or previously recorded) data at home.

## Gathering data

* CC1101 is readily available from resellers.
* Raspberry Pi can be used for programming.
* Capture data with high bit rate such as 256 kbps. `a630808237fffffff804000007fff80007fff81007f00ff017f00ff00f710ff01ff08ef00ff00ff00ff00ff00ff00ffff90fe0001fe01fffe0001de01fe01fffe0001ffee0001be01fe01fe01fffc03fc03ec100bff7c03fc0003fc13fffc11dc0443fffc1003ffda47f807f81005fff807f80006f807fff807f80007fff807f00ff00fe00ff00bf00ff00ff00ff00ff0000ffff0800ff01be01fe01fffe0001fe01fe01fffe0001fe017cfe01fe0003fffc0083fffc03fc03dc0000000000000000118008ea87bba5`
* Infer actual bit rate from runs of 0's and 1's. Seems to be around 32 kbps.
* Capture data with the new bit rate. `f0ccaaaaaab4b2b32ad4d2d33534b4d555532acacb4cd400`
* Recognize and decode Differential Manchester encoding. `f00056789abcd58038cd79`

## Analyzing data

* Only three last bytes change between messages.
* Last byte is a CRC check sum (dallas-1-wire without reflecting output) over six previous bytes (bytes `9abcd58038cd` above).

    `pycrc.py --width 8 --poly 0x31 --reflect-in True --xor-in 0x00 --reflect-out False --xor-out 0x00`

* Second last byte loops through values 0–255 in a predetermined scrambled order.

    `counter(i) = 0x1f ^ ((i & 0x60) << 1) ^ ((i & 0x01) << 2) ^ ((i & 0x02) << 3) ^ ((i & 0x10) >> 1) ^ ((i & 0x8c) >> 2)`

* Third last byte indicates movement in scale 0–100, always divisible by 8 except for 100.
* Later observation: fourth last byte changes permanently from `80` to `8c` if the strap is disconnected.

## Conclusion

The device is very simple and can be reimplemented easily with Raspberry Pi and CC1101 or a similar transmitter.
