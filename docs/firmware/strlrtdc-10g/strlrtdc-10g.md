# Streaming low-resolution TDC with SiTCP-XG

## Overview

The CIRASAME streaming low-resolution TDC with SiTCP-XG (CIRASAME Str-LRTDC 10G) is a continuous readout TDC with 1ns timing precision.
This firmware is basically the same as CIRASAME Str-LRTDC, but the data link protocol is replaced with SiTCP-XG.
In this user guide, the CIRASAME Str-LRTDC 10G is simply called as Str-LRTDC-10G.

[Github repository](https://github.com/spadi-alliance/CIRASAME-StrLRTDC-10G)

```
- Unique ID:                  0x7041

- Number of inputs:           128
- Timing measurements:        Both edges
- TDC precision:              1ns
- Double hit resolution:      ~8ns
- Max TOT length:             4000ns

- Link protocol:              SiTCP-XG
- Default IP:                 192.168.10.10
- Data link speed:            10Gbps

- Data word width:            64bit
- Acceptable max input rate:  ~125MHz/board
- System clock freq.:         125MHz
```

### History

|Version|Date|Changes|
|:----:|:----|:----|
|v2.1|2025.3.5|- Bugfix version of v2.0. <br> - Enabling the function to generate data words with input throttling type-2 start/end data types. |
|v2.0|2025.1.7|First release version|

## Functions

The internal function is completely the same as that of CIRASAME Str-LRTDC, but some unnecessary functions such as ADC readout are omitted.
See the description about that.

## Local bus modules

The Str-LRTDC-10G includes 8 local bus modules.
The local bus address map is as follows.

|Local Module|Address range|
|:----|:----|
|Mikumari Utility        |0x0000'0000 - 0x0FFF'0000|
|CITIROC Controller      |0x3000'0000 - 0x3FFF'0000|
|IO Manager              |0x4000'0000 - 0x4FFF'0000|
|Streaming TDC           |0x5000'0000 - 0x5FFF'0000|
|Scaler                  |0x8000'0000 - 0x8FFF'0000|
|MAX1932 Controller      |0x9000'0000 - 0x9FFF'0000|
|Self Diagnosis System   |0xC000'0000 - 0xCFFF'0000|
|Flash Memory Programmer |0xD000'0000 - 0xDFFF'0000|
|Bus Controller          |0xE000'0000 - 0xEFFF'0000|

