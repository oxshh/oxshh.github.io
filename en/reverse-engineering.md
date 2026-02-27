---
title: Reverse engineering
nav_order: 5
parent: English
---
# Reverse engineering
Notes on reverse engineering.

A basic Cimg(x86)
```
00401264    int32_t main(int32_t argc, char** argv, char** envp)

00401278        void* fsbase
00401278        int64_t rax = *(fsbase + 0x28)
00401291        char var_38
00401291        __builtin_memset(dest: &var_38, ch: 0, count: 0x18)
00401291        
00401293        if (argc s<= 1)
00401293            goto label_4012f9
00401293        
00401295        char* file = argv[1]
00401299        int64_t i = -1
004012a4        char* file_1 = file
004012a4        
004012a7        while (i != 0)
004012a7            bool cond:1_1 = 0 != *file_1
004012a7            file_1 = &file_1[1]
004012a7            i -= 1
004012a7            
004012a7            if (not(cond:1_1))
004012a7                break
004012a7        
004012b8        if (strcmp(&file[not.q(i) - 6], ".cimg") == 0)
004012df            dup2(zx.q(open(file, oflag: 0)), 0)
004012f9        label_4012f9:
004012f9            read_exact(0, &var_38, 0x18, "ERROR: Failed to read header!", 0xffffffff)
00401317            char var_37
00401317            char var_36
00401317            char var_35
00401317            int32_t var_34
00401317            
00401317            if (var_38 != 0x63 || var_37 != 0x49 || var_36 != 0x4d || var_35 != 0x47)
00401320                puts(str: "ERROR: Invalid magic number!")
00401317            else
00401339                int64_t var_30
00401339                
00401339                if (var_34 != 2)
00401320                    puts(str: "ERROR: Unsupported version!")
00401339                else
00401348                    int64_t var_28
00401348                    
00401348                    if (var_30 != 0x4b)
00401320                        puts(str: "ERROR: Incorrect width!")
00401348                    else if (var_28 != 0x16)
00401320                        puts(str: "ERROR: Incorrect height!")
00401357                    else
0040135e                        int64_t rax_5 = malloc(bytes: 0x19c8)
0040135e                        
00401370                        if (rax_5 == 0)
00401320                            puts(str: "
00401320                                ERROR: Failed to allocate memory for the image data!")
00401370                        else
00401387                            read_exact(0, rax_5, 0x19c8, "ERROR: Failed to read data!", 
00401387                                0xffffffff)
00401397                            int64_t rax_6 = 0
004013ad                            char rcx_2
004013ad                            
004013ad                            do
0040139c                                if (var_30 * var_28 == rax_6)
004013d4                                    display(&var_38, rax_5)
004013de                                    int64_t i_1 = 0
004013e8                                    char rdx_2 = 1
004013e8                                    
004013f0                                    for (; var_30 * var_28 != i_1; i_1 += 1)
004013fd                                        if (*(rax_5 + (i_1 << 2)) != 0x8c
004013fd                                                || *(rax_5 + (i_1 << 2) + 1) != 0x1d)
00401409                                            rdx_2 = 0
004013fd                                        else if (*(rax_5 + (i_1 << 2) + 2) != 0x40)
00401404                                            rdx_2 = 0
00401404                                    
00401410                                    int64_t i_2 = 0
00401412                                    int32_t rsi_4 = 0
00401412                                    
00401417                                    for (; i_1 != i_2; i_2 += 1)
0040141e                                        if (*(rax_5 + (i_2 << 2) + 3) != 0x20)
00401420                                            rsi_4 += 1
00401420                                    
00401432                                    if (rsi_4 == 0x672 && (rdx_2 & 1) != 0)
00401436                                        win()
00401436                                    
00401449                                    if (rax == *(fsbase + 0x28))
0040145a                                        return 0
0040145a                                    
0040144b                                    __stack_chk_fail()
0040144b                                    noreturn
0040144b                                
0040139e                                rcx_2 = *(rax_5 + (rax_6 << 2) + 3)
004013a3                                rax_6 += 1
004013ad                            while (rcx_2 - 0x20 u<= 0x5e)
004013c4                            __fprintf_chk(fp: stderr, flag: 1, 
004013c4                                format: "
004013c4                                    ERROR: Invalid character 0x%x in the image data!\n")
004012b8        else
004012c8            __printf_chk(flag: 1, format: "ERROR: Invalid file extension!")
004012c8        
00401328        exit(status: 0xffffffff)
00401328        noreturn
```

### 1) Analyze the disassembled code

Read the checks:

- Magic: `"cIMG"`

- Version: `2`

- Width: `75`

- Height: `22`

- Pixel count: `75 * 22 = 1650`

- Pixel size: `4 bytes`

- Total data size: `1650 * 4 = 6600 bytes`

### 2) Reconstruct the pixel structure

From the checks:

- Byte0 = `0x8c`

- Byte1 = `0x1d`

- Byte2 = `0x40`

- Byte3 ∈ `[0x20; 0x7e]` (printable ASCII)

- Byte3 ≠ `0x20` (otherwise the condition `rsi_4 == 1650` will not be met)

That is, the pixel must be:

Code

```
8c 1d 40 XX where XX is any printable character except 0x20
```

We chose `0x2e` (`.`).

### 3) Check the final condition of the win() call

Code

```
if (rsi_4 == 0x672 && (rdx_2 & 1) != 0)
win()
```

- `0x672 = 1650` is the number of pixels.

- `rsi_4` is the number of pixels for which the 4th byte != 0x20.

- `rdx_2` — a flag that the first three bytes of all pixels are correct.

To get to `win()`:

- ALL pixels must have correct first three bytes.

- ALL pixels must have a 4th byte ≠ 0x20.

### 4) Building a correct file

We build:

- Header (24 bytes)

- 1650 pixels of 4 bytes each

And write to file.


script
```
import struct
from pwn import *

# Build the header (24 bytes total)
magic = b"cIMG"                    # 4 bytes
version = struct.pack("<I", 2)     # 4 bytes
width = struct.pack("<Q", 75)      # 8 bytes 
height = struct.pack("<Q", 22)     # 8 bytes 

header = magic + version + width + height

# Build the pixel data (75 * 22 * 4 = 6600 bytes)
pixel = b"\x8c\x1d\x40."   
pixel_data = pixel * (75 * 22)  

# Full file content
cimg_data = header + pixel_data

# Write to disk
filename = "/home/hacker/solution.cimg"
with open(filename, "wb") as f:
    f.write(cimg_data)

print(f"Wrote {len(cimg_data)} bytes: {cimg_data} to file: '{filename}'")
```