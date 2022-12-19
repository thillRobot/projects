## setting resolution in Ubuntu

https://unix.stackexchange.com/questions/227876/how-to-set-custom-resolution-using-xrandr-when-the-resolution-is-not-available-i

```
xrandr
Screen 0: minimum 8 x 8, current 3840 x 1080, maximum 32767 x 32767
DP-0 connected 1920x1080+1920+0 (normal left inverted right x axis y axis) 527mm x 296mm
   1920x1080     60.00*+  59.94    50.00  
   1680x1050     59.95  
   1440x900      59.89  
   1280x1024     75.02    71.93    60.02  
   1280x720      60.00    59.94    50.00  
   1152x864      75.00  
   1024x768      75.03    70.07    60.00  
   800x600       75.00    72.19    60.32  
   720x576       50.00  
   720x480       59.94  
   640x480       75.00    72.81    59.94    59.93  
DP-1 disconnected (normal left inverted right x axis y axis)
DP-2 disconnected (normal left inverted right x axis y axis)
DP-3 disconnected (normal left inverted right x axis y axis)
DP-4 connected primary 1920x1080+0+0 (normal left inverted right x axis y axis) 527mm x 296mm
   1920x1080     60.00*+  74.91    59.94    50.00  
   1680x1050     59.95  
   1600x900      74.89    60.00  
   1440x900      59.89  
   1280x1024     75.02    69.83    60.02  
   1280x720      60.00    59.94    50.00  
   1152x864      75.00  
   1024x768      75.03    70.07    60.00  
   800x600       75.00    72.19    60.32    56.25  
   720x576       50.00  
   720x480       59.94  
   640x480       75.00    72.81    59.94    59.93  
DP-5 disconnected (normal left inverted right x axis y axis)
DP-6 disconnected (normal left inverted right x axis y axis)
DP-7 disconnected (normal left inverted right x axis y axis)
HDMI-1-1 disconnected (normal left inverted right x axis y axis)
DP-1-1 disconnected (normal left inverted right x axis y axis)
HDMI-1-2 disconnected (normal left inverted right x axis y axis)
DP-1-2 disconnected (normal left inverted right x axis y axis)
HDMI-1-3 disconnected (normal left inverted right x axis y axis)
thill@BRWN305-D01:~$ xrandr --addmode DP-0 3840x2160
xrandr: cannot find mode "3840x2160"
thill@BRWN305-D01:~$ xrandr --newmode 3840x2160
xrandr: failed to parse '3840x2160' as a mode specification
Try 'xrandr --help' for more information.
```

Calculate the Modeline for 3840x2160@60Hz

```
gtf 3840 2160 60

  # 3840x2160 @ 60.00 Hz (GTF) hsync: 134.10 kHz; pclk: 712.34 MHz
  Modeline "3840x2160_60.00"  712.34  3840 4152 4576 5312  2160 2161 2164 2235  -HSync +Vsync

```

```
cvt 3840 2160 60
# 3840x2160 59.98 Hz (CVT 8.29M9) hsync: 134.18 kHz; pclk: 712.75 MHz
Modeline "3840x2160_60.00"  712.75  3840 4160 4576 5312  2160 2163 2168 2237 -hsync +vsync
```



Create a new mode 
```
xrandr --newmode "3840x2160_60.00"  712.34  3840 4152 4576 5312  2160 2161 2164 2235  -HSync +Vsync
```

```
xrandr --newmode "3840x2160_60.00"  712.75  3840 4160 4576 5312  2160 2163 2168 2237 -hsync +vsync
```


Add the new mode
```
xrandr --addmode DP-0 "3840x2160_60.00"
```

```
xrandr --addmode DP-0 "4k"
```




```
sudo get-edid | edid-decode -c
This is read-edid version 3.0.2. Prepare for some fun.
Attempting to use i2c interface
No EDID on bus 0
No EDID on bus 1
No EDID on bus 3
No EDID on bus 4
No EDID on bus 5
No EDID on bus 6
No EDID on bus 7
No EDID on bus 8
No EDID on bus 9
No EDID on bus 10
No EDID on bus 11
No EDID on bus 12
No EDID on bus 13
No EDID on bus 14
No EDID on bus 15
No EDID on bus 16
No EDID on bus 17
1 potential busses found: 2
256-byte EDID successfully retrieved from i2c bus 2
Looks like i2c was successful. Have a good day.
edid-decode (hex):

00 ff ff ff ff ff ff 00 30 ae f7 61 01 01 01 01 
0c 1e 01 03 80 35 1e 78 2e 05 65 a7 56 52 9c 27 
0f 50 54 bd cf 00 71 4f 81 80 81 8c 95 00 b3 00 
d1 c0 01 01 01 01 02 3a 80 18 71 38 2d 40 58 2c 
45 00 0f 28 21 00 00 1e 00 00 00 ff 00 56 4b 43 
46 36 31 31 35 0a 20 20 20 20 00 00 00 fd 00 32 
4b 1e 53 11 00 0a 20 20 20 20 20 20 00 00 00 fc 
00 4c 45 4e 20 54 32 34 69 2d 32 30 0a 20 01 78 

02 03 1e f1 4b 01 02 03 04 05 14 11 12 13 90 1f 
23 09 07 07 83 01 00 00 65 03 0c 00 10 00 01 1d 
00 72 51 d0 1e 20 6e 28 55 00 0f 28 21 00 00 1e 
8c 0a d0 8a 20 e0 2d 10 10 3e 96 00 0f 28 21 00 
00 18 8c 0a d0 90 20 40 31 20 0c 40 55 00 0f 28 
21 00 00 18 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ce 

----------------

EDID version: 1.3
Manufacturer: LEN Model 25079 Serial Number 16843009
Made in week 12 of 2020
Digital display
Maximum image size: 53 cm x 30 cm
Gamma: 2.20
DPMS levels: Off
RGB color display
Default (sRGB) color space is primary color space
First detailed timing is preferred timing
Color Characteristics
  Red:   0.6523, 0.3359
  Green: 0.3212, 0.6103
  Blue:  0.1533, 0.0605
  White: 0.3134, 0.3291
Established Timings I & II
    720x400    70.082 Hz   9:5    31.467 kHz  28.320 MHz (IBM)
    640x480    59.940 Hz   4:3    31.469 kHz  25.175 MHz (DMT)
    640x480    66.667 Hz   4:3    35.000 kHz  30.240 MHz (Apple)
    640x480    72.809 Hz   4:3    37.861 kHz  31.500 MHz (DMT)
    640x480    75.000 Hz   4:3    37.500 kHz  31.500 MHz (DMT)
    800x600    60.317 Hz   4:3    37.879 kHz  40.000 MHz (DMT)
    800x600    72.188 Hz   4:3    48.077 kHz  50.000 MHz (DMT)
    800x600    75.000 Hz   4:3    46.875 kHz  49.500 MHz (DMT)
   1024x768    60.004 Hz   4:3    48.363 kHz  65.000 MHz (DMT)
   1024x768    70.069 Hz   4:3    56.476 kHz  75.000 MHz (DMT)
   1024x768    75.029 Hz   4:3    60.023 kHz  78.750 MHz (DMT)
   1280x1024   75.025 Hz   5:4    79.976 kHz 135.000 MHz (DMT)
Standard Timings
   1152x864    75.000 Hz   4:3    67.500 kHz 108.000 MHz (DMT)
   1280x1024   60.020 Hz   5:4    63.981 kHz 108.000 MHz (DMT)
   1280x1024   72.000 Hz   5:4    76.824 kHz 132.752 MHz (GTF)
   1440x900    59.887 Hz  16:10   55.935 kHz 106.500 MHz (DMT)
   1680x1050   59.954 Hz  16:10   65.290 kHz 146.250 MHz (DMT)
   1920x1080   60.000 Hz  16:9    67.500 kHz 148.500 MHz (DMT)
Detailed mode: Clock 148.500 MHz, 527 mm x 296 mm
               1920 2008 2052 2200 ( 88  44 148)
               1080 1084 1089 1125 (  4   5  36)
               +hsync +vsync
               VertFreq: 60.000 Hz, HorFreq: 67.500 kHz
Display Product Serial Number: VKCF6115
Display Range Limits
  Monitor ranges (GTF): 50-75 Hz V, 30-83 kHz H, max dotclock 170 MHz
Display Product Name: LEN T24i-20
Has 1 extension block
Checksum: 0x78

----------------

CTA-861 Extension Block Revision 3
Underscans PC formats by default
Basic audio support
Supports YCbCr 4:4:4
Supports YCbCr 4:2:2
1 native detailed modes
26 bytes of CTA data blocks
  Video Data Block
      640x480    59.940 Hz   4:3    31.469 kHz  25.175 MHz (VIC   1)
      720x480    59.940 Hz   4:3    31.469 kHz  27.000 MHz (VIC   2)
      720x480    59.940 Hz  16:9    31.469 kHz  27.000 MHz (VIC   3)
     1280x720    60.000 Hz  16:9    45.000 kHz  74.250 MHz (VIC   4)
     1920x1080i  60.000 Hz  16:9    33.750 kHz  74.250 MHz (VIC   5)
     1920x1080i  50.000 Hz  16:9    28.125 kHz  74.250 MHz (VIC  20)
      720x576    50.000 Hz   4:3    31.250 kHz  27.000 MHz (VIC  17)
      720x576    50.000 Hz  16:9    31.250 kHz  27.000 MHz (VIC  18)
     1280x720    50.000 Hz  16:9    37.500 kHz  74.250 MHz (VIC  19)
     1920x1080   60.000 Hz  16:9    67.500 kHz 148.500 MHz (VIC  16, native)
     1920x1080   50.000 Hz  16:9    56.250 kHz 148.500 MHz (VIC  31)
  Audio Data Block
    Linear PCM, max channels 2
      Supported sample rates (kHz): 48 44.1 32
      Supported sample sizes (bits): 24 20 16
  Speaker Allocation Data Block
    Speaker map:
      FL/FR - Front Left/Right
  Vendor-Specific Data Block, OUI 0x000c03 (HDMI)
    Source physical address 1.0.0.0
Detailed mode: Clock 74.250 MHz, 527 mm x 296 mm
               1280 1390 1430 1650 (110  40 220)
                720  725  730  750 (  5   5  20)
               +hsync +vsync
               VertFreq: 60.000 Hz, HorFreq: 45.000 kHz
Detailed mode: Clock 27.000 MHz, 527 mm x 296 mm
                720  736  798  858 ( 16  62  60)
                480  489  495  525 (  9   6  30)
               -hsync -vsync
               VertFreq: 59.940 Hz, HorFreq: 31.469 kHz
Detailed mode: Clock 27.000 MHz, 527 mm x 296 mm
                720  732  796  864 ( 12  64  68)
                576  581  586  625 (  5   5  39)
               -hsync -vsync
               VertFreq: 50.000 Hz, HorFreq: 31.250 kHz
Checksum: 0xce

----------------

edid-decode SHA: not available

Failures:

Block 0 (Base Block):
  Basic Display Parameters & Features: sRGB is signaled, but the chromaticities do not match
Block 1 (CTA-861 Extension Block):
  Both the serial number and the serial string are set
All Blocks:
  One or more of the timings is out of range of the Monitor Ranges:
    Vertical Freq: 50 - 75 Hz (Monitor: 50 - 75 Hz)
    Horizontal Freq: 28.125 - 79.976 kHz (Monitor: 30.000 - 83.000 kHz)
    Maximum Clock: 148.500 MHz (Monitor: 170.000 MHz)

EDID conformity: FAIL
```