# Software

Here, the author describes the hul-common-lib and the cirasame-soft.

## hul-common-lib

[Github repository](https://github.com/spadi-alliance/hul-common-lib)

Since the basic structure of The CIRASAME firmware is the same that of AMANEQ's firmware, the CIRASAME board can be controlled by hul-common-lib.
Before reading this section, see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/software/software/).
In this user guide, the author additionally explains about programs for CIRASAME.

### Executable files

#### set_max1932

It can set 8-bit register value to the internal DAC of MAX1932. Usage is as follows.
```shell
            [IP]          [DAC value]
set_max1932 192.168.10.16 100
```
Please provides the DAC value between 1-255 in decimal notation.
If you set 0, the MAX1932 is turned down.

## cirasame-soft

[Github repository](https://github.com/spadi-alliance/cirasame-soft)

The cirasame-soft provides programs dedicated for the CIRASAME firmware.
It requires hul-common-lib to be complied.

### Executable files

#### ad9220

This program accesses the AD9220 block in the FPGA and reads out the ADC data for the specified number of events.
The data that has just been read is written to the file specified by the argument.
As described in the firmware section, no header or trailer words are added to indicate the end of an event.
Process every 132 words.
**This program should not be terminated by Ctrl-C.**
If users stop it by Ctrl-C, data in the next run will be broken.

Example to read 100 events and store them to hoge.dat.
```shell
       [IP]          [file]   [Events]
ad9220 192.168.10.16 hoge.dat 100
```

#### hgddelay

Please read the Hold Generator sub-section in CIRASAME skeleton firmware section.
In this program, the register value given at 2nd argument is transformed to the mask pattern.
The value of 1 provides the smallest delay amount, and the delay amount increases as the given value increases.
**Please set the register value before calling ad9220 program**

## CitirocControlSoft

[Github repository](https://github.com/J-PARC-High-p/CitirocControlSoft)

The CitirocControlSoft is a software dedicated for using the CITIROC Controller module in FPGA firmware.
It generates the bit pattern to be sent to CITIROCs from configuration files and sends it to FPGA through RBCP.
In CITIROC, there are three categories for control registers.

- **Slow Control:** The biggest category takes an entire ASIC setting such as pre-amplifier gain, shaping constant, discriminator threshold DAC, etc.
- **Read Slow Control:** This register bit selects the ASIC channel to be output from the analog output.
- **Probe Register:** This register bits arranges the signal path to the probe port.

For more details, please read the CITIROC data sheet.

The directory structure is as follows.
Please compile the source codes under the `src` directory, then an executable file, `femcitiroc_control`, will be generated in the `bin` directory.

```
.
└── CitirocControlSoft/
    ├── bin
    ├── default_register
    └── src/
        ├── Makefile
        ├── BitDump.hh
        ├── BitDump.cc
        ├── FPGAModule.hh
        ├── FPGAModule.cc
        ├── RegisterMap.cc
        ├── UDPRBCP.hh
        ├── UDPRBCP.cc
        ├── Uncoyable.hh
        ├── configLoader.hh
        ├── configLoader.cc
        ├── control_impl.hh
        ├── control_impl.cc
        ├── femcitiroc_control_main.cc
        └── rbcp.h
```

Since the Slow Control category contains so many control registers, it is not realistic to manage all of them using a configuration file.
Then, a default register bit pattern for Slow Control is hard-coded in the `configLoader.cc` file.
Users can update only the parts they want to change by configuration files.
The Read Slow Control and Probe Register bit patterns are 0 in default in source code.
They can be also updated by configuration files.

When users execute `femcitiroc_control` without any arguments, usage appears as follows.
```
Usage
  femcitiroc_control [options...]

Options:
 -ip=xxx.yyy.zzz.aaa (Must)
 -yaml=file-path
   If -yaml=auto is designated, this program refers
   the register files in ${HOME}/vme-easiroc-registers,
   which are labeled by the ip address.
   i.e., RegisterValue_aaa.yml, InputDAC_aaa.yml, PedeSup_aaa.yml
 -sc
   Send slow control registers
 -read
   Send read slow control registers
 -probe
   Send probe registers
 -all
   Equal to -sc -read -probe
 -probe-off
   Reset probe registers
 -q
   Quiet. Hide all the message.
```
In this program, the option is specified by a word beginning with a hyphen, unlike hul-common-lib and cirasame-soft where the meaning of option is determined by the position of the argument.
Specification of ip address is mandatory, and configure files to be read are specified with the `-yaml=` option.
Sine you can take the `-yaml=` option as many times as you want, it is possible to specify multiple configuration files.
The options, `-sc, -read, -probe`, specify the register categories to be sent.
As these categories are independent, you can change a certain category while maintaining other categories are unchanged.
Users can take multiple these options at once like `-sc -read`.

### Configuration file

If the user does not specify any configuration file by the option, default configuration files existing in the `default_register` directory are used.
At present, three configuration files, `RegisterValue.yml, InputDAC.yml, and DiscriMask.yml` are prepared.
The developer expects that these are examples to make a new configuration file by users.
The `RegisterValue.yml` file contains the Slow Control, Read Slow Control, and probe registers in a file.
The `InputDAC.yml` and `DiscriMask.yml` contain the Slow Control registers, `Input 8-bit DAC` and `Discriminator Mask`, respectively.
Although these are a part of the Slow Control category, descriptions are divided to independent files because they require a lot of text lines.

#### RegisterValue.yml

```yaml
CITIROC1:
  PreAMP: HGCf_200fF
  PreAMP: LGCf_200fF
  Time Constant HG Shaper: 50.0ns
  DAC1 code: 100
  DAC2 code: 250
  LG PA bias: weakbias
  EN_Low_Gain_PA: Disable
  Fast Shaper on LG: HG
```

This is a part of `RegisterValue.yml` in the default_register directory.
Four CITIROCs are labeled as CITIROC1, 2, 3, and 4.
Registers in CITIROC is specified by keys such as "Time Constant HG Shaper".
Alias such as "50.0ns" are prepared for some register values.
To know the list of keys and alias, please see the source code of `configLoader.cc`.
Since key names come from the register name listed in the CITIROC data sheet, you won't have much trouble finding a desired key.
Users can update the default register setting with the value written here.

Since the CIRASAME is for the streaming TDC, the hard-coded default register setting is dedicated for this purpose.
The functions after the low gain slow shaper are disabled because there is no way to use this signal.
The peak detector is one of feature of CITIROC, but this is also disabled because the charge measurement is not aimed in CIRASAME.
The charge measurement for the high-gain side using SCA with the external HOLD signal is selected in default.
The fast shaper receives the signal from the high-gain pre-amplifier in default, then the low-gain pre-amplifier is disabled.
In addition, weak bias setting is applied to the log-gain pre-amplifier.
If users use the low-gain side, please update these setting.

```yaml
High Gain Channel: 0
Probe Channel: 0
Probe: Out_fs
Probe Mux: 2
```

At the end of `RegisterValue.yml`, the register setting for Read Slow Control and Probe categories are written.
"High Gain Channel" selects the readout channel from the analog output for the high-gain side.
Users can select a value between 0 to 127.

"Probe Channel", "Probe", and "Probe Mux" are registers for the Probe category.
"High Gain Channel" selects the readout channel from the probe output.
Users can select a value between 0 to 127, but users need to additionally set "Probe Mux" register to specify the channel to be output.
"Probe" register specifies the probing point in ASIC.
The list of probing points is as follows

- **None:** Disable probe
- **Out_PA_HG:** Probe the high-gain pre-amplifier output
- **Out_PA_LG:** Probe the low-gain pre-amplifier output
- **Out_ssh_HG:** Probe the high-gain slow shaper output
- **Out_ssh_LG:** Probe the low-gain slow shaper output
- **Out_fs:** Probe the fast shaper output

"Probe Mux" specifies the ASIC number to be output from the analog switch, and users can select the value between 1 to 4.

#### InputDAC.yml

Users can set the Input 8-bit DAC value channel by channel in each CITIROC by this file.
The register can take values between 0 to 255.

#### DiscriMask.yml

Users can mask the discriminator output channel by channel by this file.
If users set 1, the output at that channel is blocked.
Please use this register to block a noisy channel.


### Usage

Users can use this program as follows.
Since this program requires many arguments, the developer recommends to write a shell script to hide them when you need to manage multiple CIRASAME boards.
The developer prepares a simple shell script (runControl.sh) as an example.

```shell
femcitiroc_control -ip=192.168.10.16 -yaml=RegisterValue_16.yml -yaml=InputDAC_16.yml -yaml=DiscriMask_16.yml -sc -read
```

Note that the probe function connects the internal probing points and the the external signal line without a buffer.
Then, the probe line looks like a long stub resulting a performance deterioration.
Please reset the probe output with the `-probe-off` option before doing the measurement.
