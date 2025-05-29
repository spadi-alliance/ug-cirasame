# Release notes

## May 29, 2025

### New + Bugfix

- Bugfix for previous version
    - Rename "trigger emulation mode" as "trigger assisted mode".
    - The delay function of the trigger gate generator for the trigger assisted mode does not work due to bug. It is fixed.
- New firmware
    - CIRASAME-StrLRTDC v2.8
    - CIRASAME-StrLRTDC-10G v2.2

## Mar. 5, 2025

- Bugfix for previous version
    - All firmware released on Jan. 7 have critical bugs.
    - So far the function to insert throttling start/end words in the input throttling type-2 unit has not been working. Now that the feature is enabled, a word is generated with input throttling type-2 start/end data types to indicate that the throttling feature is working.
- New firmware
    - CIRASAME-StrLRTDC v2.7
    - CIRASAME-StrLRTDC-10G v2.1

## Jan. 7, 2025

### New+Improved

- New function of LACCP distributing frame flags
    - LACCP can distribute 2-bits frame flags. The flags are embedded into the flags regions in the heartbeat frame delimiter word.
    - The users can embed external condition such as gates into the heartbeat frame.
- Gated scaler
    - Two scaler units gated by the frame flags are added. Totally, 3 scaler units exist in firmware.
    - Gated scaler counts up while the frame flag takes 1.
- New firmware
    - CIRASAME-StrLRTDC v2.6
    - CIRASAME-StrLRTDC-10G v2.0 (New release)

## Jun. 4, 2024

### New

- De facto release version
    - CIRASAME Skeleton v2.0
    - CIRASAME-StrLRTDC v2.5
