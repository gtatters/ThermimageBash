Command Line Instructions for Converting FLIR Video and JPG files for import to ImageJ
================

These instructions are for converting certain flir file types using command line tools. Variants of these functions are also incorporated into the Thermimage package through the functions, convertflirJPG(), converflirVID(), and ffmpegcall(). If these functions are not working in R, try following the command line instructions here to diagnose your system.

### System Requirements

Exiftool: <https://www.sno.phy.queensu.ca/~phil/exiftool/>

Imagemagick: <https://www.imagemagick.org/script/index.php>

Perl: <https://www.perl.org/get.html>

### Download and extract sample files to SampleFLIR folder on desktop:

<https://github.com/gtatters/ThermImageJ/blob/master/SampleFLIR.zip>

``` bash
cd ~/IRconvert/SampleFLIR
ls
```

    ## CSQconverted.avi
    ## SEQconverted.avi
    ## SEQconvertedjpegls.avi
    ## SEQconvertedpng.avi
    ## SampleFLIR.avi
    ## SampleFLIR.csq
    ## SampleFLIR.jpg
    ## SampleFLIR.seq
    ## output

### Download and extract this perl script (split.pl) to a scripts folder in an IRconvert folder:

<https://github.com/gtatters/ThermimageBash/blob/master/Uploads/split.pl>

``` bash
cd ~/IRconvert/scripts
ls
```

    ## ConvertACQtoTXT
    ## ConvertCSQtoAVI
    ## ConvertFCFtoAVI
    ## ConvertFLIRJPG.sh
    ## ConvertFLIRJPGtoRAW
    ## ConvertFLIRJPGtoTIFF
    ## ConvertFLIRRAWtoTIFF
    ## ConvertSEQtoAVI
    ## ConverttoGrayscale
    ## ExtractAllFLIRJPGs
    ## IRFileImport.R
    ## ImageJRad2Temp.txt
    ## ImportRaw_FCFsettings.jpeg
    ## Process_Folder GUI mod.ijm
    ## Process_Folder.ijm
    ## Process_Folder_TIFF2Temp.ijm
    ## SEQtoVID_Rima
    ## exiftool commands.docx
    ## ffmpegscript
    ## imagejscript_rad2temp
    ## split.pl
    ## split_fff.pl
    ## split_jpegls.pl
    ## split_tiff.pl
    ## test.pl
    ## test2.pl
    ## tryopts.sh
    ## workflow.txt
    ## workflow_windows.txt

### Workflow to convert csq (1024x768) to avi file

1.  Break video into .fff files into a temp/ subfolder
2.  Extract times from each frame (this is optional but allows a quick check that the fff files are valid).
3.  Put raw thermal data from fff into one thermalvid.raw file in temp folder.
4.  Break thermalvid.raw video from .CSQ file into .jpegls files into temp folder.
5.  Convert all jpegls files into avi file.
    -----Use -codec png for compatibility, -codec jpegls for greater compression. -----Use -pix\_fmt gray16be for big endian export format, -pix\_fmt gray16le for little endian format. -----Use -f image2 -codec png to export a series of PNG files instead of an avi.
6.  Import avi into ImageJ using File-&gt;Import-&gt;Movie(ffmpeg) import routine. -----Import png files into ImageJ using File-&gt;Import-&gt;Image Sequence

``` bash
cd ~/IRconvert
perl -f ~/IRconvert/scripts/split.pl -i ~/IRconvert/SampleFLIR/SampleFLIR.csq -o temp -b frame -p fff -x fff
ls temp
rm temp/frame00008.fff # remove 8th frame - due to file corruption
echo

exiftool -DateTimeOriginal temp/*.fff 
exiftool -b -RawThermalImage temp/*.fff > temp/thermalvid.raw
ls temp/*.raw
echo

perl ~/IRconvert/scripts/split.pl -i temp/thermalvid.raw -o temp -b frame -p jpegls -x jpegls
ls temp/*.jpegls
echo

cd ~/IRconvert/SampleFLIR
ffmpeg -f image2 -vcodec jpegls -r 30 -s 1024x768 -i ~/IRconvert/temp/frame%05d.jpegls -pix_fmt gray16be -vcodec jpegls -s 1024x768 CSQconverted.avi -y
echo

ffmpeg -f image2 -vcodec jpegls -r 30 -s 1024x768 -i ~/IRconvert/temp/frame%05d.jpegls -f image2 -pix_fmt gray16be -vcodec png -s 1024x768 output/frame%05d.png -y

ls output/*.avi
ls output/*.png
cd ~/IRconvert
rm -r temp
```

    ## frame00001.fff
    ## frame00002.fff
    ## frame00003.fff
    ## frame00004.fff
    ## frame00005.fff
    ## frame00006.fff
    ## frame00007.fff
    ## frame00008.fff
    ## 
    ## ======== temp/frame00001.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.583-07:00
    ## ======== temp/frame00002.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.617-07:00
    ## ======== temp/frame00003.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.650-07:00
    ## ======== temp/frame00004.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.683-07:00
    ## ======== temp/frame00005.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.717-07:00
    ## ======== temp/frame00006.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.750-07:00
    ## ======== temp/frame00007.fff
    ## Date/Time Original              : 2017:05:19 12:45:33.783-07:00
    ##     7 image files read
    ## temp/thermalvid.raw
    ## 
    ## temp/frame00001.jpegls
    ## temp/frame00002.jpegls
    ## temp/frame00003.jpegls
    ## temp/frame00004.jpegls
    ## temp/frame00005.jpegls
    ## temp/frame00006.jpegls
    ## temp/frame00007.jpegls
    ## 
    ## ffmpeg version 4.1 Copyright (c) 2000-2018 the FFmpeg developers
    ##   built with Apple LLVM version 10.0.0 (clang-1000.11.45.5)
    ##   configuration: --prefix=/usr/local/Cellar/ffmpeg/4.1_1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-libsnappy --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-opencl --enable-videotoolbox
    ##   libavutil      56. 22.100 / 56. 22.100
    ##   libavcodec     58. 35.100 / 58. 35.100
    ##   libavformat    58. 20.100 / 58. 20.100
    ##   libavdevice    58.  5.100 / 58.  5.100
    ##   libavfilter     7. 40.101 /  7. 40.101
    ##   libavresample   4.  0.  0 /  4.  0.  0
    ##   libswscale      5.  3.100 /  5.  3.100
    ##   libswresample   3.  3.100 /  3.  3.100
    ##   libpostproc    55.  3.100 / 55.  3.100
    ## Input #0, image2, from '/Users/GlennTattersall/IRconvert/temp/frame%05d.jpegls':
    ##   Duration: 00:00:00.23, start: 0.000000, bitrate: N/A
    ##     Stream #0:0: Video: jpegls, gray16le(bt470bg/unknown/unknown), 1024x768, lossless, 30 fps, 30 tbr, 30 tbn, 30 tbc
    ## Stream mapping:
    ##   Stream #0:0 -> #0:0 (jpegls (native) -> jpegls (native))
    ## Press [q] to stop, [?] for help
    ## Incompatible pixel format 'gray16be' for codec 'jpegls', auto-selecting format 'gray16le'
    ## Output #0, avi, to 'CSQconverted.avi':
    ##   Metadata:
    ##     ISFT            : Lavf58.20.100
    ##     Stream #0:0: Video: jpegls (MJLS / 0x534C4A4D), gray16le, 1024x768, q=2-31, 200 kb/s, 30 fps, 30 tbn, 30 tbc
    ##     Metadata:
    ##       encoder         : Lavc58.35.100 jpegls
    ## frame=    7 fps=0.0 q=-0.0 Lsize=    3656kB time=00:00:00.23 bitrate=128339.8kbits/s speed=1.08x    
    ## video:3650kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.156768%
    ## 
    ## ffmpeg version 4.1 Copyright (c) 2000-2018 the FFmpeg developers
    ##   built with Apple LLVM version 10.0.0 (clang-1000.11.45.5)
    ##   configuration: --prefix=/usr/local/Cellar/ffmpeg/4.1_1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-libsnappy --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-opencl --enable-videotoolbox
    ##   libavutil      56. 22.100 / 56. 22.100
    ##   libavcodec     58. 35.100 / 58. 35.100
    ##   libavformat    58. 20.100 / 58. 20.100
    ##   libavdevice    58.  5.100 / 58.  5.100
    ##   libavfilter     7. 40.101 /  7. 40.101
    ##   libavresample   4.  0.  0 /  4.  0.  0
    ##   libswscale      5.  3.100 /  5.  3.100
    ##   libswresample   3.  3.100 /  3.  3.100
    ##   libpostproc    55.  3.100 / 55.  3.100
    ## Input #0, image2, from '/Users/GlennTattersall/IRconvert/temp/frame%05d.jpegls':
    ##   Duration: 00:00:00.23, start: 0.000000, bitrate: N/A
    ##     Stream #0:0: Video: jpegls, gray16le(bt470bg/unknown/unknown), 1024x768, lossless, 30 fps, 30 tbr, 30 tbn, 30 tbc
    ## Stream mapping:
    ##   Stream #0:0 -> #0:0 (jpegls (native) -> png (native))
    ## Press [q] to stop, [?] for help
    ## Output #0, image2, to 'output/frame%05d.png':
    ##   Metadata:
    ##     encoder         : Lavf58.20.100
    ##     Stream #0:0: Video: png, gray16be, 1024x768, q=2-31, 200 kb/s, 30 fps, 30 tbn, 30 tbc
    ##     Metadata:
    ##       encoder         : Lavc58.35.100 png
    ## frame=    7 fps=0.0 q=-0.0 Lsize=N/A time=00:00:00.23 bitrate=N/A speed=0.818x    
    ## video:4758kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: unknown
    ## output/SampleFLIR.csq.avi
    ## output/JPGconverted.png
    ## output/SampleFLIR.png
    ## output/frame00001.png
    ## output/frame00002.png
    ## output/frame00003.png
    ## output/frame00004.png
    ## output/frame00005.png
    ## output/frame00006.png
    ## output/frame00007.png

Which produces the following output:

<https://github.com/gtatters/ThermimageBash/blob/master/Uploads/CSQconverted.avi?raw=true>

The above avi should open up in VLC player, but may or may not play properly. In ImageJ, with the ffmpeg plugin installed, the jpegls compression should work.

<img src='Uploads/frame00001.png' align="centre">

The above PNG file is a sample image of the 16 bit grayscale image. Although it looks washed out, it can be imported into ImageJ and the Brightness/Contrast changed for optimal viewing.

### Workflow to convert seq (640x480) to avi file

1.  Break video into .fff files into temp/ subfolder.
2.  Extract times from each frame (this is optional but allows a quick check that the fff files are valid).
3.  Put raw thermal data from fff into one thermalvid.raw file in temp folder.
4.  Break thermalvid.raw video from .CSQ file into .tiff files into temp folder.
5.  Convert all tiff files into avi file.
6.  Convert all jpegls files into avi file.
    ----- Use -codec png for compatibility, -codec jpegls for greater compression. ------Use -pix\_fmt gray16be for big endian export format, -pix\_fmt gray16le for little endian format.
7.  Import avi into ImageJ using File-&gt;Import-&gt;Movie(ffmpeg) import routine.

``` bash
cd ~/IRconvert
perl -f ~/IRconvert/scripts/split.pl -i ~/IRconvert/SampleFLIR/SampleFLIR.seq -o temp -b frame -p fff -x fff
ls temp
echo

exiftool -DateTimeOriginal temp/*.fff 
exiftool -b -RawThermalImage temp/*.fff > temp/thermalvid.raw
ls temp/*.raw
echo

perl ~/IRconvert/scripts/split.pl -i temp/thermalvid.raw -o temp -b frame -p tiff -x tiff
ls temp/*tiff
echo

cd ~/IRconvert/SampleFLIR
ffmpeg -f image2 -vcodec tiff -r 30 -s 640x480 -i ~/IRconvert/temp/frame%05d.tiff -pix_fmt gray16be -vcodec jpegls -s 640x480 SEQconvertedjpegls.avi -y

ffmpeg -f image2 -vcodec tiff -r 30 -s 640x480 -i ~/IRconvert/temp/frame%05d.tiff -pix_fmt gray16be -vcodec png -s 640x480 SEQconvertedpng.avi -y
echo

ls output/*.avi
cd ~/IRconvert
rm -r temp
```

    ## frame00001.fff
    ## frame00002.fff
    ## frame00003.fff
    ## frame00004.fff
    ## frame00005.fff
    ## frame00006.fff
    ## frame00007.fff
    ## frame00008.fff
    ## frame00009.fff
    ## frame00010.fff
    ## frame00011.fff
    ## frame00012.fff
    ## frame00013.fff
    ## frame00014.fff
    ## frame00015.fff
    ## frame00016.fff
    ## frame00017.fff
    ## frame00018.fff
    ## frame00019.fff
    ## frame00020.fff
    ## frame00021.fff
    ## frame00022.fff
    ## frame00023.fff
    ## frame00024.fff
    ## frame00025.fff
    ## frame00026.fff
    ## frame00027.fff
    ## frame00028.fff
    ## 
    ## ======== temp/frame00001.fff
    ## Date/Time Original              : 2017:09:27 12:17:50.423+08:00
    ## ======== temp/frame00002.fff
    ## Date/Time Original              : 2017:09:27 12:17:51.451+08:00
    ## ======== temp/frame00003.fff
    ## Date/Time Original              : 2017:09:27 12:17:52.479+08:00
    ## ======== temp/frame00004.fff
    ## Date/Time Original              : 2017:09:27 12:17:53.440+08:00
    ## ======== temp/frame00005.fff
    ## Date/Time Original              : 2017:09:27 12:17:54.468+08:00
    ## ======== temp/frame00006.fff
    ## Date/Time Original              : 2017:09:27 12:17:55.496+08:00
    ## ======== temp/frame00007.fff
    ## Date/Time Original              : 2017:09:27 12:17:56.424+08:00
    ## ======== temp/frame00008.fff
    ## Date/Time Original              : 2017:09:27 12:17:57.452+08:00
    ## ======== temp/frame00009.fff
    ## Date/Time Original              : 2017:09:27 12:17:58.480+08:00
    ## ======== temp/frame00010.fff
    ## Date/Time Original              : 2017:09:27 12:17:59.508+08:00
    ## ======== temp/frame00011.fff
    ## Date/Time Original              : 2017:09:27 12:18:00.438+08:00
    ## ======== temp/frame00012.fff
    ## Date/Time Original              : 2017:09:27 12:18:01.466+08:00
    ## ======== temp/frame00013.fff
    ## Date/Time Original              : 2017:09:27 12:18:02.493+08:00
    ## ======== temp/frame00014.fff
    ## Date/Time Original              : 2017:09:27 12:18:03.422+08:00
    ## ======== temp/frame00015.fff
    ## Date/Time Original              : 2017:09:27 12:18:04.450+08:00
    ## ======== temp/frame00016.fff
    ## Date/Time Original              : 2017:09:27 12:18:05.478+08:00
    ## ======== temp/frame00017.fff
    ## Date/Time Original              : 2017:09:27 12:18:06.406+08:00
    ## ======== temp/frame00018.fff
    ## Date/Time Original              : 2017:09:27 12:18:07.434+08:00
    ## ======== temp/frame00019.fff
    ## Date/Time Original              : 2017:09:27 12:18:08.462+08:00
    ## ======== temp/frame00020.fff
    ## Date/Time Original              : 2017:09:27 12:18:09.490+08:00
    ## ======== temp/frame00021.fff
    ## Date/Time Original              : 2017:09:27 12:18:10.418+08:00
    ## ======== temp/frame00022.fff
    ## Date/Time Original              : 2017:09:27 12:18:11.446+08:00
    ## ======== temp/frame00023.fff
    ## Date/Time Original              : 2017:09:27 12:18:12.475+08:00
    ## ======== temp/frame00024.fff
    ## Date/Time Original              : 2017:09:27 12:18:13.502+08:00
    ## ======== temp/frame00025.fff
    ## Date/Time Original              : 2017:09:27 12:18:14.430+08:00
    ## ======== temp/frame00026.fff
    ## Date/Time Original              : 2017:09:27 12:18:15.458+08:00
    ## ======== temp/frame00027.fff
    ## Date/Time Original              : 2017:09:27 12:18:16.486+08:00
    ## ======== temp/frame00028.fff
    ## Date/Time Original              : 2017:09:27 12:18:17.414+08:00
    ##    28 image files read
    ## temp/thermalvid.raw
    ## 
    ## temp/frame00001.tiff
    ## temp/frame00002.tiff
    ## temp/frame00003.tiff
    ## temp/frame00004.tiff
    ## temp/frame00005.tiff
    ## temp/frame00006.tiff
    ## temp/frame00007.tiff
    ## temp/frame00008.tiff
    ## temp/frame00009.tiff
    ## temp/frame00010.tiff
    ## temp/frame00011.tiff
    ## temp/frame00012.tiff
    ## temp/frame00013.tiff
    ## temp/frame00014.tiff
    ## temp/frame00015.tiff
    ## temp/frame00016.tiff
    ## temp/frame00017.tiff
    ## temp/frame00018.tiff
    ## temp/frame00019.tiff
    ## temp/frame00020.tiff
    ## temp/frame00021.tiff
    ## temp/frame00022.tiff
    ## temp/frame00023.tiff
    ## temp/frame00024.tiff
    ## temp/frame00025.tiff
    ## temp/frame00026.tiff
    ## temp/frame00027.tiff
    ## temp/frame00028.tiff
    ## 
    ## ffmpeg version 4.1 Copyright (c) 2000-2018 the FFmpeg developers
    ##   built with Apple LLVM version 10.0.0 (clang-1000.11.45.5)
    ##   configuration: --prefix=/usr/local/Cellar/ffmpeg/4.1_1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-libsnappy --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-opencl --enable-videotoolbox
    ##   libavutil      56. 22.100 / 56. 22.100
    ##   libavcodec     58. 35.100 / 58. 35.100
    ##   libavformat    58. 20.100 / 58. 20.100
    ##   libavdevice    58.  5.100 / 58.  5.100
    ##   libavfilter     7. 40.101 /  7. 40.101
    ##   libavresample   4.  0.  0 /  4.  0.  0
    ##   libswscale      5.  3.100 /  5.  3.100
    ##   libswresample   3.  3.100 /  3.  3.100
    ##   libpostproc    55.  3.100 / 55.  3.100
    ## Input #0, image2, from '/Users/GlennTattersall/IRconvert/temp/frame%05d.tiff':
    ##   Duration: 00:00:00.93, start: 0.000000, bitrate: N/A
    ##     Stream #0:0: Video: tiff, gray16le, 640x480 [SAR 1:1 DAR 4:3], 30 fps, 30 tbr, 30 tbn, 30 tbc
    ## Stream mapping:
    ##   Stream #0:0 -> #0:0 (tiff (native) -> jpegls (native))
    ## Press [q] to stop, [?] for help
    ## Incompatible pixel format 'gray16be' for codec 'jpegls', auto-selecting format 'gray16le'
    ## Output #0, avi, to 'SEQconvertedjpegls.avi':
    ##   Metadata:
    ##     ISFT            : Lavf58.20.100
    ##     Stream #0:0: Video: jpegls (MJLS / 0x534C4A4D), gray16le, 640x480 [SAR 1:1 DAR 4:3], q=2-31, 200 kb/s, 30 fps, 30 tbn, 30 tbc
    ##     Metadata:
    ##       encoder         : Lavc58.35.100 jpegls
    ## frame=   28 fps=0.0 q=-0.0 Lsize=    5763kB time=00:00:00.93 bitrate=50583.9kbits/s speed=10.2x    
    ## video:5757kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.109466%
    ## ffmpeg version 4.1 Copyright (c) 2000-2018 the FFmpeg developers
    ##   built with Apple LLVM version 10.0.0 (clang-1000.11.45.5)
    ##   configuration: --prefix=/usr/local/Cellar/ffmpeg/4.1_1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-libsnappy --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-opencl --enable-videotoolbox
    ##   libavutil      56. 22.100 / 56. 22.100
    ##   libavcodec     58. 35.100 / 58. 35.100
    ##   libavformat    58. 20.100 / 58. 20.100
    ##   libavdevice    58.  5.100 / 58.  5.100
    ##   libavfilter     7. 40.101 /  7. 40.101
    ##   libavresample   4.  0.  0 /  4.  0.  0
    ##   libswscale      5.  3.100 /  5.  3.100
    ##   libswresample   3.  3.100 /  3.  3.100
    ##   libpostproc    55.  3.100 / 55.  3.100
    ## Input #0, image2, from '/Users/GlennTattersall/IRconvert/temp/frame%05d.tiff':
    ##   Duration: 00:00:00.93, start: 0.000000, bitrate: N/A
    ##     Stream #0:0: Video: tiff, gray16le, 640x480 [SAR 1:1 DAR 4:3], 30 fps, 30 tbr, 30 tbn, 30 tbc
    ## Stream mapping:
    ##   Stream #0:0 -> #0:0 (tiff (native) -> png (native))
    ## Press [q] to stop, [?] for help
    ## Output #0, avi, to 'SEQconvertedpng.avi':
    ##   Metadata:
    ##     ISFT            : Lavf58.20.100
    ##     Stream #0:0: Video: png (MPNG / 0x474E504D), gray16be, 640x480 [SAR 1:1 DAR 4:3], q=2-31, 200 kb/s, 30 fps, 30 tbn, 30 tbc
    ##     Metadata:
    ##       encoder         : Lavc58.35.100 png
    ## frame=   28 fps=0.0 q=-0.0 Lsize=   10032kB time=00:00:00.93 bitrate=88049.1kbits/s speed=3.94x    
    ## video:10025kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.062800%
    ## 
    ## output/SampleFLIR.csq.avi

Which produces the following output:

<https://github.com/gtatters/ThermimageBash/blob/master/Uploads/SEQconvertedjpegls.avi?raw=true>

<https://github.com/gtatters/ThermimageBash/blob/master/Uploads/SEQconvertedpng.avi?raw=true>

Note: the above avi should open up in VLC player, but may or may not play properly. In ImageJ, with the ffmpeg plugin installed, the jpegls compression should work.

### Workflow to convert FLIR jpg (640x480) to png file

1.  Use exiftool to extract RawThermalImage from the FLIR jpg.
2.  Pass the raw thermal image data to imagemagick's convert function to convert to 16 bi grayscale with little endian
3.  Convert to PNG (PNG is lossless, compressed, and easiest). --- Save to different filetype (tiff, bmp, or jpg) as needed (not recommended for further analysis).
4.  Use exiftool to extract calibration constants from file (for use in converting raw values)

``` bash
cd ~/IRconvert/SampleFLIR
exiftool ~/IRconvert/SampleFLIR/SampleFLIR.jpg -b -RawThermalImage | convert - gray:- | convert -depth 16 -endian lsb -size 640x480 gray:- output/JPGconverted.png

exiftool ~/IRconvert/SampleFLIR/SampleFLIR.jpg -*Planck*
```

    ## Planck R1                       : 21106.77
    ## Planck B                        : 1501
    ## Planck F                        : 1
    ## Planck O                        : -7340
    ## Planck R2                       : 0.012545258

<img src='Uploads/JPGconverted.png' align="centre">

### Workflow to convert FLIR jpg multi-burst (with ultramax) to png file

Note: this section is a work in progress. Code below is not yet functional, but saved here for reference.

Extract the multiple raw thermal image burts and export as .hex exiftool -config config.txt -a -b -CompressedBurst -v -W "Image/%.2c.hex" IR\_2017-02-10\_0003.jpg

Extract the just the first of the multiple raw thermal image burts and export as .hex exiftool -config config.txt -b -CompressedBurst -v -W "%.2c.hex" IR\_2017-02-10\_0003.jpg

Convert these .hex files to png ffmpeg -f image2 -vcodec jpegls -i "%02d.hex" -f image2 -vcodec png burst%02d.png

ffmpeg -f image2 -vcodec jpegls -i "./Image/%02d.hex" -f image2 -vcodec png PNG/burst%02d.png

Then try using fairSIM from github - a plug-in for ImageJ that produces the superresolution image

### References

1.  <https://www.sno.phy.queensu.ca/~phil/exiftool/>

2.  <https://www.imagemagick.org/script/index.php>

3.  <https://www.eevblog.com/forum/thermal-imaging/csq-file-format/>
