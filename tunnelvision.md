# PicoCTF platform: Tunn3l_v1s10n

This challenge involves fixing a corrupted image file.

## Solution

Downloading the image from the challenge and opening it showed error: "BMP image has unsupported header size".  
At first I used `file` on the image and it returned:  
```
tunn3l_v1s10n: data
```
Usually data type for an image, means that maybe the image has been tampered with or is corrupted.  
Then I ran exiftool (tool for reading and editing files metadata), I noticed a few strange things:  
BMP version is unknown (BMP has multiple header versions), image height was somewhat small.  
Following this I decided to look at the bytes of the file with hexeditor tool:
```bash
exiftool tunn3l_v1s10n 
ExifTool Version Number         : 13.25
File Name                       : tunn3l_v1s10n
Directory                       : .
File Size                       : 2.9 MB
File Modification Date/Time     : 2026:05:10 12:26:33-04:00
File Access Date/Time           : 2026:05:13 10:58:19-04:00
File Inode Change Date/Time     : 2026:05:10 12:26:57-04:00
File Permissions                : -rw-rw-rw-
File Type                       : BMP
File Type Extension             : bmp
MIME Type                       : image/bmp
BMP Version                     : Unknown (53434)
Image Width                     : 1134
Image Height                    : 306
Planes                          : 1
Bit Depth                       : 24
Compression                     : None
Image Length                    : 2893400
Pixels Per Meter X              : 5669
Pixels Per Meter Y              : 5669
Num Colors                      : Use BitDepth
Num Important Colors            : All
Red Mask                        : 0x27171a23
Green Mask                      : 0x20291b1e
Blue Mask                       : 0x1e212a1d
Alpha Mask                      : 0x311a1d26
Color Space                     : Unknown (,5%()
Rendering Intent                : Unknown (826103054)
Image Size                      : 1134x306
Megapixels                      : 0.347
```

Opening the file with hexeditor we can see at the top the header:
```text
00000000  42 4D 8E 26  2C 00 00 00   00 00 BA D0  00 00 BA D0 BM.&,...........
00000010  00 00 6E 04  00 00 32 01   00 00 01 00  18 00 00 00 ..n...2.........
00000020  00 00 58 26  2C 00 25 16   00 00 25 16  00 00 00 00 ..X&,.%...%.....
00000030  00 00 00 00  00 00 23 1A   17 27 1E 1B  29 20 1D 2A ......#..'..) .*
```

I decided to google for BMP file information, looked it up on wikipedia and found this detailed BMP format explanation:
https://www.ece.ualberta.ca/~elliott/ee552/studentAppNotes/2003_w/misc/bmp_file_format/bmp_file_format.htm
## First Fix - DataOffset
```text
* BMP fields are little-endian so to read the bytes we need to reverse them.
BMP Header: 	14 bytes
	Signature: 	2 bytes 	| 	42 4D (BM)
	FileSize: 	4 bytes		|	8E 26  2C 00 (little endian)
		Convert to decimal 0x002C268E = 2893454 bytes ~ 2.9 MB
    Reserved:   4 bytes     |   00 00  00 00 (application specific)
    DataOffset: 4 bytes     |   BA D0  00 00 -> FIX -> 36 00 00 00 (from beginning of file to beginning of bitmap data)
        Convert to decimal 0x00D0BA = 53434 bytes
        This should be the offset where bitmap data starts which is 54 bytes == 0x36 (40(BITMAPINFOHEADER) + 14(BMP file header))
        The bytes here are incorrect and we need to fix them to the correct offset 0x36.
```

## Second Fix - Info Header Size
```text
Info Header:    40 bytes
    Size:       4 bytes     |   BA D0 00 00 -> FIX -> 28 00 00 00
        The size of the InfoHeader should be 40 bytes = 0x28, we need to fix these incorrect bytes as before.
    Width:      4 bytes     |   6E 04  00 00 -> 0x0000046E = 1134
    Height:     4 bytes     |   32 01  00 00 -> 0x00000132 = 306
    ......
```

After fixing the BMP Header, the image rendered correctly but only showed the message notaflag{sorry}.

## Change Image Dimensions
Trying to find the flag, I looked again at the output of exiftool on the file, and the image width and height caught my eye.  
The image dimensions were suspiciously small compared to the file size, suggesting maybe there is more image data hiding.  
At first I changed image height to 0x00000332 = 818 pixels, opening the image I noticed the cropped flag text at the top of the image, by adding a bit more height 0x00000432 = 1074 pixels I was able to see the full flag text.  

## Final Thoughts
The image was cropped with incorrect height metadata, hiding the flag in unused pixel. This challenge involved editing file bytes and experimenting with metadata values. it was clever exercise in file format manipulation and showed the importance of being cautious when handling downloaded files.