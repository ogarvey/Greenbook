# CD-I Image Specifications

## 3.4.1 DYUV Encoding Model

The final encoded image has 8 bits/pixel, or 16 bits per pixel pair, which is the basic
indivisible encoding unit. For disc and decoder formats see V.6.5.1 and V.4.4.2
respectively.

# 3.6 Graphics and Text
## 3.6.1 General
The methods specified for CD-I are given in VI.3.1 (text) and VII.2.3.2.9 (UCM).
For the direct downloading of graphic images, both direct absolute RGB coding and
indirect CLUT (Color Lookup Table) coding are available.

## 3.6.2 Absolute RGB
Images can be coded in R, G, B to an accuracy of 5 bits for each component, plus a
remaining bit reserved for overlay information.
For each color component, the encoding law is:
C = (219 * C + 16)/8, rounded to the nearest integer A
where C = A color component (one of R,G,B)
C = The corresponding analog component, in the range 0 to 1 A
C = 2 therefore represents the black level.
For disc and decoder formats see V.6.5.2 and V.4.4.4 respectively.

## 3.6.3 Color Lookup Table
This technique allows selection of a given number of colors from a much larger
palette. The set of colors to be used for the image is loaded into a Color Lookup
Table (CLUT) in the decoder.
Each pixel is then coded with the position in the CLUT of the color for that pixel.
The colors in the CLUT may be chosen from a palette of 2 colors, i.e. R, G and B are 24
each defined at 8-bit accuracy.

The CD-I system uses the following CLUT sizes
Figure V.22 CLUT Sizes
Mechanism Colors Available path Purpose
8-bit 256 colors path 0 only CLUT8
7-biy 128 colors path 0 and/or path 1 CLUT7, RL7
Dual 7-bit 128 x 2 colors path 0 only CLUT7, RL7
4-bit 16 colors path 0 and/or path 1 CLUT4
3-bit 8 colors path 0 and/or path 1 RL3
The available number of colors may be expanded by dynamic loading of new colors
on a line by line basis.
For disc and decoder formats see V.6.5.3 and V.4.4.5 respectively.

# 3.7 Runlength Coding
Images may be runlength coded, with either 7 bits or 3 bits of color. The high degree
of data compression may be used for animation of cartoon style images.

## 3.7.1 Run-coded 7-bit CLUT (RL7)
Single pixels are coded as single bytes, giving the CLUT code for that pixel. A run
of pixels of the same color is coded as two bytes, the CLUT code and the length of
the run (from 2 to 255). A length of zero is used to indicate that the run extends until
the end of the current line. All lines must be terminated with this code as an end
marker. The coding is strictly on a line basis, so that there is no wraparound to the
next line.
This scheme ensures that the compression ratio is never less than 1. It is typically
of the order of 10 for cartoon style images.
RL7 may be used with a single 7-bit CLUT or with dual 7-bit CLUTs (see V.5.5.3).
Dynamic CLUT updates (see V.5.5.2) may be used to increase the number of colors
in a run-coded image.

## 3.7.2 Run-coded 3-bit CLUT (RL3)
The same coding scheme is used as for 7 bit runlength coding, except that the pixels
are coded in pairs, one byte holding two 3-bit CLUT codes. The runlength specifies
the number of times the pixel pair is repeated.
For disc and decoder formats see V.6.5.4 and V.4.4.6 respectively.

# 4.4 Image Representation and Decoding
## 4.4.1 General
### 4.4.1.1 Representations in Memory
Each horizontal line of pixels is represented in memory by a contiguous sequence
of pixel codes at increasing memory addresses from left to right. However,
consecutive lines of an image are not necessarily contiguous in memory.

In the case of RGB555 images which require combined data streams from the two
image stores, each pixel code is split into two halves, each of which is arranged in
such a contiguous sequence.

Notation: In this section, pixels are numbered consecutively from left to right as
displayed. For bit and byte ordering conventions see V.1.2.

### 4.4.1.2 RGB Levels
All images, encoded by whatever method, are decoded (in terms of this model) to
a uniform 8-bit linear representation of the Red, Green and Blue color components.
For each component, black level is at 16 and nominal peak (white) level is at 235.
The analog output is given by:
$$C_A = (C-16)/219$$
where 
$$ C_A $$ 
= An analog component (one of R,G,B) 

= The corresponding digital component.

As the range of C is from 0-255, the range of 
$$C_A$$ 
is -0.073 to 1.091.

---

## 4.4.2 DYUV
### 4.4.2.1 Representation in Memory

Each pixel pair (pixels i & i+1) is coded as two bytes as follows: 

$$ \delta U_i (3-0) \delta Y_i (3-0) \delta V_i (3-0) \delta Y_(i+1)$$
15       12 11        8 7         4 3     0

where i must be even. The pixel pair starts at an even X coordinate and at an even
memory address.

δY = 4 bit luminance difference code.

δU, δV = 4 bit chrominance difference codes

### 4.4.2.2 Decoding Model
The process of decoding DYUV coded images is as shown in Figure V.30. The composite data in memory is separated into its delta pcm coded Y, U and V components. Each of the three components is decoded by a DPCM decoder to give an eight bit pcm image. Due to the subsampling the U and V images are only half resolution horizontally. It is thus necessary to interpolate the missing values. The normal resolution Y image 
$$Y_c$$ 
is matrixed with the interpolated 
$$U_c$$ 
and 
$$V_c$$
images to give the eight bit digital RGB signals R'G'B'.

![](images/Figure%20V.30.png)

#### Delta decoding
For each component Y, U and V, the same decoding scheme is used. For each
picture element along a horizontal line the decoded value 
$$F_n$$
is obtained from the coded value 
$$c_n$$
by adding the decoded difference value 
$$ Qf^-1 ( c_n )$$
to the prediction value 
$$P_n$$
The prediction value 
$$P_n$$
is equal to the previous decoded value
$$F_n-1$$
except for the first element of the line, where the prediction value is set equal to the
separately defined absolute value 
$$P_abs$$

if n > 0
$$F_n = F_(n-1)+Qf^-1(c_n)$$ 


$$F_0 = P_(abs) + Qf^-1(c_0) $$

The notation used above is the same as that defined in V.3.4.1.

The same quantization table (Figure V.18) is used for defining the decoding function
$$Qf^-1()$$ 
as is used for encoding.
Addition is by modulo 256 arithmetic.

#### Interpolation of U and V

The values of U and V obtained from the delta decoder are at only half the horizontal resolution of the Y values, and so must be interpolated. Care must be taken to ensure that the inter-polation filter, which may perform simple linear interpolation, preserves the encoding phase relationships of V.3.4.1.2.

#### Matrixing of YUV to RGB
The Y value and the interpolated U and V values are matrixed to give RGB values.
The matrix equations are given below.

If the decoded values of YUV are

$$ Y_c U_c V_c$$

the values must first be de-normalised.

$$ U' = (U-c - 128) *1.733 $$
$$ V' = (V-c - 128) *1.371 $$
then

$$ B' = Y_c + (U-c - 128) *1.733 $$
$$ R' = Y_c + (V-c - 128) *1.371 $$
$$ G' = (Y_c - 0.299 * R' - 0.114 * B')/0.587$$

NOTE:
The decoded values R'G'B' will have nominal black levels of 16. Total excursions are in the range 0...255. 

### 4.4.2.3 DYUV Absolute Start Values

The initial values of Y, U & V for each line (
  $$P_(abs)$$
in the above equation for delta 
coding), are transmitted separately to the decoder and loaded via the display control program (V.4.5.1).

---
## 4.4.4 Absolute RGB

The 16 bit word representing each pixel is split into two halves, upper and lower, the
first in image store 0 and the second in image store 1.

Figure V.35 Memory Representation Absolute RGB

Upper: 
$$T_i R_i (4-0) G_i (4-3)$$

Lower: 
$$G_i (2-0) B_i (4-0)$$

R, G and B are 5 bit absolute values of Red, Green and Blue respectively. Each
component is decoded to the standard form of V.4.4.1.2 by multiplying by 8. A value
of 2 represents black level.
T is the Transparency bit (see V.5.7) 

---

## 4.4.5 CLUT
### 4.4.5.1 8-bit CLUT
The representation in memory is:

Figure V.36 Memory Representation 8-bit CLUT
$$C_i $$ 
7-0

C = 8 bit CLUT address.

### 4.4.5.2 7 bit CLUT
The representation in memory is:
Figure V.37 Memory Representation 7-bit CLUT
$$C_i (6-0)$$
7 - 6 - 0

C = 7 bit CLUT address.

### 4.4.5.3 4 bit CLUT

The basic unit is the pixel pair. The representation in memory is:
Figure V.38 Memory Representation 4-bit CLUT
$$C_i (3-0) C_(i+1) (3-0)$$
7       4   3         0

where C = 4 bit CLUT address and i must be even.
Each byte represents a pixel pair starting at an even X coordinate.

## 4.4.6 Runlength Coded CLUT

Run-coded images are stored in memory in compressed form, and are expanded to
CLUT codes by a runlength decoder associated with each image store (see Figure
V.26). There is no linear relationship between pixel number and memory address.
Lines of an image are of variable length in memory. The memory representations are given below.
### 4.4.6.1 Run-length coded 7-bit CLUT
Single pixels are encoded as one byte, and a run of pixels of the same color as two bytes.
Figure V.39 Memory Representation RL 7-bit CLUT
Single Pixel: 
0 `C_i` (6-0)
7 6       0
Pixel run:
1   `C_i` (6-0) `L_i` (7-0)
15  14     8  7      0
C = 7 bit CLUT address,
L = 8 bit run length = Number of pixels in run
where 2 <= L <= 255 and L = 1 is forbidden.
**L = 0 means 'Continue this run to the end of the line'.**
Every displayed line must finish with a zero-length run (L=0). This run must start no later than the last-pixel-but-one to be displayed, i.e. it must start no later than pixel 383 for a 625/compatible image, or pixel 359 for a 525 line image. **This means that the displayed line must have at least two identical pixels at its right hand end.**

### 4.4.6.2 Run-length coded 3-bit CLUT
The same basic memory representation is used as for 7-bit CLUT, but with a different interpretation of the pixel code byte. The 3-bit CLUT codes are converted to 4 bit CLUT codes by appending a zero msb before they address the CLUT.
Figure V.40 Memory Representation RL 3-bit CLUT
Pixel pair:
0   C_i(2-0)  1 C_i+1 (2-0)
7   6      4  3 2         0
Run of Pixel pair:
1   C_i(2-0)  0   C_i+1(2-0)  L_i(7-0)
15  14    12  11  10       8  7      0
i = even
C = 3 bit CLUT address,
L = 8 bit run length = Number of pixel pairs in run
where 2 <= L <= 255 and L = 1 is forbidden.
**L = 0 means 'Continue this run to the end of the line'.**
Every displayed line must finish with a zero-length run (L=0). This run must start no later than the last-pixel-but-one to be displayed, i.e. it must start no later than pixel 383 for a 625/compatible image, or pixel 359 for a 525 line image. **This means that the displayed line must have at least two identical pixels at its right hand end.**

### 4.4.7 CLUT Organization
The CLUT has 256 entries. In order to facilitate sharing of the CLUT between the two paths the CLUT is divided into four banks of 64 entries each, Bank 0 to Bank 3. The CLUT usage by the various planes is shown in Figure V.41.


B           | 8-bit CLUT | 7-bit CLUT| 4-bit CLUT | 3-bit CLUT 
------------|------------|-----------|------------|----------
63| 255        | 127       |            | 
Clut Bank 3 | .          | .         |            | 
0| .          | Path 1    |            | 
-| .          | Plane B or|-|
63| .          | Path 0    | 15         |-
Clut Bank 2 | .          | Plane A   |  Plane B   | 7 Plane B
0| Path 0     | 0         | 0          | 0
-| Plane A    |||-
63| .          | 127       |            |   
Clut Bank 1 | .          | .         |            |   
0| .          | .         |            |   
-| .          | Path 0    |-|
63| .          | Plane A   | 15         |-
Clut Bank 0 | .          | .         |  Plane A   | 7 Plane A
0| 0          | 0         | 0          | 0

### 4.4.8 Allowed Image Coding Combinations
The allowed image coding combinations are determined by:
(1) The bits/pixel for each plane. These are specified in the basic decoding model.
(2) The way the CLUT is shared between display paths.
(3) The data rate available in each path. This determines the allowed resolutions.
The basic set of allowed combinations, from (1) above, is:
Plane A: Off, DYUV, CLUT4, CLUT7, CLUT8, RL3, RL7.

Plane B: Off, DYUV, CLUT4, CLUT7, RL3, RL7, RGB555, QHY.
Note:
RL3 is a special case of CLUT 4, with the most significant bit set to 0 by the decoder before CLUT lookup
Restrictions
(1) RGB555 uses both image data streams, so it can only be used with plane A
turned off.
(2) CLUT8 uses the whole CLUT, so it may only be used with plane B set to DYUV
or turned off.
The same restriction applies to CLUT7 and RL7 used with dual 7-bit CLUTs.
(3) QHY is defined only for the extended case, and must be used with Normal
Resolution DYUV in plane A.

### 4.4.9 Allowed Image Coding Resolutions
The specified resolutions of each coding type are shown in Figure V.42.
Figure V.42 Allowed Image Code Resolutions

. |Normal |Double |High
--|-------|-------|----
CLUT4 |- |B |B
CLUT7 |B |- |E
CLUT8 |B |- |E
RGB555 |B |- |-
RL3 |- |B |B
RL7 |B |- |E
DYUV |B |- |E
QHY |- |- |E

B = Specified image coding for Base Case

E = Specified image coding for Extended Case

.  = Unspecified image coding

It should be noted that application-specific coded data can be transformed by
'software manipulation' to produce an image conforming to one of the above
specified coding types. This is allowed within this specification (see V.6.3.1).
Note also that, by use of the interlace (see V.5.14), the vertical resolution of all Normal and Double resolution image formats may be doubled. In the case of Double resolution formats, i.e. CLUT4 and RL3, the result is High resolution.

## 4.5 Display Control Facilities
Most display control is performed by means of a display control program (DCP)
associated with each path (see Figure V.27). UCM functions (see VII.2.3.4) are provided to access the DCP's.

### 4.5.1 Display Control Program
A detailed specification of the DCP is given in VII.2.3.2 and a short description is given here.

A display control program consists of a Field Control Table (FCT) and a Line Control Table (LCT). The FCT is a one-dimensional array of instructions, which are carried out before the start of each field. The LCT is a two dimensional array of instructions, each row of which is associated with a display line. Successive rows of the table correspond to successive display lines. The control actions corresponding to the entries in a row are carried out between the end of the previous line and the start of the line with which the row is associated.

The LCT control mechanism allows display parameters and visual effects to be
redefined on a line by line basis.

Each instruction consists of a one byte control code followed by three bytes of
parameters. The available instructions are shown in the tables given in Figures V.43, V.44 and V.45 and their respective parameters are detailed in the sections listed under 'more details'. **All the codes in these tables may be used in either the FCT or the LCT unless otherwise stated.** All codes not in these tables are reserved.

The Display Control Programs in the two paths operate in conjunction. The functions specific to each path are controlled only by that path. Most global control functions are only available in path 0, but some are available in both paths.

Display parameters loaded by a DCP instruction on a particular line are retained on
following display lines, until over-written by a subsequent instruction on a later line.
However, display parameters are **not** guaranteed to be retained from field to field.
They are un-defined at the start of a field, and must be loaded afresh for each field.
