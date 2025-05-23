# Practical usage

Please read the AMANEQ user guide.
All the things written in the AMANEQ practical section are also important for CIRASAME.

## Initial startup procedures

When powering up CIRASAME for the first time, please initialize CDCE62002 according to the following procedure

- Download CIRASAME skeleton firmware to FPGA
- Set the following registers using set_cdce62002 in hul-common-lib

```set_cdce62002 [ip address] in_125_out_500_125```

- Confirm with a tester that 1.5V is output from +1V5D.
- Turn off the power and short-circuit J1 as shown in the [figure](#J1).

![J1](J1.png "Short-circuit J1"){: #J1 width="70%"}
