# SPR Format Specification

## Overview

SPR files are a compiled (binary) form of a sprite sheet, used to store all images used to represent an ingame entity in a single file. As such, they're essentially a texture atlas and can be represented similarly after the individual images (frames) have been extracted.

They encode the following information:

* Pixel data (for one or several images)
* Frame definitions
* Palette information

## Prerequisites

In order to understand the SPR file format, some familiarity with the following concepts is required:

* [RGB](https://en.wikipedia.org/wiki/RGB_color_model) and [alpha channels](https://en.wikipedia.org/wiki/Alpha_compositing): A standard way to define colors and transparency
* Sprite sheets (see [Texture Atlas](https://en.wikipedia.org/wiki/Texture_atlas)): A straightforward method to combine textures in a reversible way
* [Palettes](https://en.wikipedia.org/wiki/Palette_(computing)), [bitmaps](https://en.wikipedia.org/wiki/BMP_file_format), and [indexed colors](https://en.wikipedia.org/wiki/Indexed_color): By using a lookup table, less space is used to store pixels
* [Run-length encoding](https://en.wikipedia.org/wiki/Run-length_encoding): This basic lossless compression algorithm serves to reduce the image's file size

## Frames

Each frame simply defines the pixel data, width, and height of a bitmap. The way these are defined differs, but at the end of the day the result is always an image that can be used to render a sprite in the game.

**Indexed** frames use the color palette that is appended at the end of the file, with each pixel index referring to one of the 256 colors instead of the a RGB(A) value that represents the pixel color (and opacity). Old versions appear to use a predefined "system palette" instead, though more information is needed on the subject.

**RGB** frames instead define the pixels as RGBA values. Transparency is only supported starting from version 2.0, with previous versions ignoring the alpha value. These frames always appear after the indexed color images (if any exist) and it is common to see very few to none of those, which is probably due to them taking up more space and only being required for transparency effects.

## Palettes

All palettes contain 256 colors and are defined as RGBA values, i.e., 4 bytes per color for a total of 1024 bytes that are located at the end of the file. They're referred by the indexed-color pixels and serve as a Color Lookup Table (CLUT) when generating the actual images. The first color (at index 0) is set as the transparency color, i.e. it won't be shown in the game.

It's not quite clear what *exact* palette SPR files of version 1.0 use, since they don't include the actual palette colors and are said to use the "system palette" instead. From my limited research, this most likely to refers to the [Windows 256 color standard palette](Images/WindowsSystemPalette.png) that was commonly used at the time of RO's development.

Disclaimer: Since I haven't seen this version used anywhere, I can't guarantee that those colors are indeed accurate.

## Compression of similar (transparent) pixels

In the later versions, indexed-color pixels that represent the invisible background are compressed using Run-Length Encoding (see above). This doesn't apply to the RGBA frames, however, which are not compressed.

This simply means that all "runs" of multiple (more than two) zero bytes in the indexed-color frames are replaced with ``00 ??`` where ``??`` is the number of zeroes replaced (two zero bytes result in ``00``, i.e., a single zero byte). Decompressing then simply removes ``??`` and replaces it with the respective number of zero bytes again.

Since it is almost guaranteed that many "runs" of the same color will appear in the transparent background and rarely, if ever, in the actual sprites, applying RLE to the remaining data wouldn't be helpful in reducing the file size (in fact, it would make it larger as each one-byte indexed-color pixel would be replaced by *two* bytes).

> When a 0x00 byte is parsed, the next byte indicates the number of 0x00 bytes it decompresses to. (0x00 0x00 decompresses to a single 0x00 byte)

## Layout

The file structure differs slightly with each file version, as newer versions added more features. Please consult the tables below for a detailed specification.

It appears that the header encodes the version in reverse, i.e. ``53 50 01 02`` reads as ``SP 1.2`` but it's actually version 2.1 and not 1.2.

### Version 1.0

In its oldest and most primitive form, a SPR file includes only the very basics required to display an image and the definition for the spritesheet's frames.

|        Field        | Offset |   Size   |  Type  |                   Description and notes                    |
| :-----------------: | :----: | :------: | :----: | :--------------------------------------------------------: |
|       Header        |   0    | 2 bytes  |  char  |              "SP" as an ASCII-encoded string               |
|       Version       |   2    | 2 bytes  | binary |            Versioning information (Minor.Major)            |
| Indexed Frame Count |   4    | 2 bytes  |  int   | The number of individual indexed-color images in the atlas |
|       Bitmaps       |   6+   | variable | struct |  The size depends on the frame count and number of pixels  |

I haven't seen this version myself, but it seems likely it would've been used in the early versions of the game; or perhaps by its predecessor Arcturus, which used the same engine.

### Version 1.1

Rather than using the "system palette" (see above), a 256-color palette consisting of RGBA values is added to the end of the SPR file itself.

|        Field        |  Offset  |   Size   |  Type  |                   Description and notes                    |
| :-----------------: | :------: | :------: | :----: | :--------------------------------------------------------: |
|       Header        |    0     | 2 bytes  |  char  |              "SP" as an ASCII-encoded string               |
|       Version       |    2     | 2 bytes  | binary |            Versioning information (Minor.Major)            |
| Indexed Frame Count |    4     | 2 bytes  |  int   | The number of individual indexed-color images in the atlas |
|   Indexed Bitmaps   |    6+    | variable | struct |  The size depends on the frame count and number of pixels  |
|       Palette       | EOF-1024 | variable | struct |     Listed in order ABGR, defines the CLUT (see above)     |

I have yet to see this version myself, so I can't confirm these details.

### Version 2.0

This version introduces images with transparency, which are included as separate RGBA frames defining each pixel and no longer refer to the palette.

|        Field        |  Offset  |   Size   |  Type  |                   Description and notes                    |
| :-----------------: | :------: | :------: | :----: | :--------------------------------------------------------: |
|       Header        |    0     | 2 bytes  |  char  |              "SP" as an ASCII-encoded string               |
|       Version       |    2     | 2 bytes  | binary |            Versioning information (Minor.Major)            |
| Indexed Frame Count |    4     | 2 bytes  |  int   | The number of individual indexed-color images in the atlas |
|  RGBA Frame Count   |    6     | 2 bytes  |  int   |     The number of individual RGBA images in the atlas      |
|   Indexed Bitmaps   |    8+    | variable | struct |  The size depends on the frame count and number of pixels  |
|    RGBA Bitmaps     |    8+    | variable | struct |  The size depends on the frame count and number of pixels  |
|       Palette       | EOF-1024 | variable | struct |     Listed in order ABGR, defines the CLUT (see above)     |

I have yet to see this version myself, so I can't confirm these details.

### Version 2.1

Similar to version 2.0, but now all background pixels of the indexed-color bitmaps (i.e., those using the palette color with index 0) are compressed using RLE.

|          Field           |  Offset  |   Size   |  Type  |                   Description and notes                    |
| :----------------------: | :------: | :------: | :----: | :--------------------------------------------------------: |
|          Header          |    0     | 2 bytes  |  char  |              "SP" as an ASCII-encoded string               |
|         Version          |    2     | 2 bytes  | binary |            Versioning information (Minor.Major)            |
|   Indexed Frame Count    |    4     | 2 bytes  |  int   | The number of individual indexed-color images in the atlas |
|     RGBA Frame Count     |    6     | 2 bytes  |  int   |     The number of individual RGBA images in the atlas      |
|    RLE-Encoded Indexed Bitmaps      |    8+    | variable | struct |  The size depends on the frame count and number of pixels  |
| RGBA Bitmaps |    8+    | variable | struct |  The size depends on the frame count and number of pixels  |
|         Palette          | EOF-1024 | variable | struct |     Listed in order ABGR, defines the CLUT (see above)     |

## Tools

There are a variety of tools available to parse SPR files and obtain the image data in its raw form:

* [Tokei's GRF Viewer](https://rathena.org/board/topic/77080-grf-grf-editor): Allows opening of SPR/ACT even from the GRF. More importantly, it has great visualization for sprites and animations
* [actOR2](https://ratemyserver.net/index.php?page=download_tool): Oldschool executable tool that allows editing SPR (and the associated ACT) files
* [SprViewer](https://ratemyserver.net/index.php?page=download_tool): Minimalist oldschool executable that extracts SPR data, but doesn't support ACT

After converting the binary data back to the original (bitmap) form, any image editing software can be used to modify them. If your goal is to create spritesheets, there's a few options that should work well enough:

* [TexturePacker](https://www.codeandweb.com/texturepacker): By far the most-feature rich and easy-to-use offline tool, but it's very limited in its free edition.
* There's also an online tool supporting the same format: https://www.codeandweb.com/free-sprite-sheet-packer