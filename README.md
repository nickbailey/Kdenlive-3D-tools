# Kdenlive-3D-tools

## What's this?

Some consumer camcorders come with the lenses or adaptors to permit 3D stereography.
The result is stored as an MPEG transport stream ("MTS" file) and that makes it hard
to edit with non-proprietary video editors such as [Kdenlive](https://kdenlive.org).

Such files store two video streams. The first is a conventional AVC stream
which records the left eye. The right eye information from my camcorder is recorded as an
[AVCHD](https://en.wikipedia.org/wiki/AVCHD) dependent stream. My understanding is this
contains only differences between the right and left eye vide; there are no i-frames.

This structure retains compatibility with existing video applications because they
only recognise the left-eye information and treat the file as a regular 2D one.

3D is, at best, niche. It is therefore unlikely there will be any core work in Kdenlive
to support 3D files presented in this format. However, stereo videography is still
being done with two cameras, and it would be nice to edit the results Kdenlive.

The plan is to enable Kdenilve to be able to produce a final 3D cut using the following
workflow:

1. Extract the video and audio streams from the AVCHD container.
1. Create a right-eye standalone video stream for use later.
1. Edit the MTS sources using the left-eye stream as normal.
1. Export a script which does the editing.
1. Run the tools here to perform the render. This will invoke the following processes:
    1. Run kdenlive's script against the source files producing a left-eye output file.
    1. Adjust the environment so that the edit operation
       will operate on the right-eye video files.
    1. Combine the output from the left and right eye sources to a form
       displayable on a 3D TV or projector (not necessarily AVCHD; possibly
       side-by-side or top-and-bottom frames)
1. declare success.

It is recognised that this is not an efficient way of doing things, but it should at least
give back some creative control of 3D content to those who are no longer able to run
the supplied "editing suites" that came with the original cameras.

## Project status

Just starting up and targetting Linux.

I've managed to extract the right eye as an MPV which plays stand-alone,
but not using a fully open-source tool chain. There's a windows executable I'm using to decode the dependent
stream in to a stand-alone file once the streams have been extracated, which runs in
[Wine](https://wiki.winehq.org/).

## Stage-by-stage from the Command Line
### Demultiplexing the MTS files

Either install your distribution's `tsmuxer` package (for Debian, version 2.6.11 is made available
via [deb-multimedia](http://www.deb-multimedia.org/) or you may prefer to build it from the
[github source repository](https://github.com/justdan96/tsMuxer).

Establish the tracks in the MTS file:
```
$ tsMuxeR file.MTS
Network Optix tsMuxeR.  Version 2.6.11. www.networkoptix.com
Track ID:    4114
Stream type: MVC
Stream ID:   V_MPEG4/ISO/MVC
Stream info: H.264/MVC Views: 2 Profile: High@4.0  Resolution: 1920:1080i  Frame rate: 25  3d-pg-planes: 0
Stream lang: 

Track ID:    4113
Stream type: H.264
Stream ID:   V_MPEG4/ISO/AVC
Stream info: Profile: High@4.0  Resolution: 1920:1080i  Frame rate: 25
Stream lang: 

Track ID:    4352
Stream type: AC3
Stream ID:   A_AC3
Stream info: Bitrate: 256Kbps Sample Rate: 48KHz Channels: 2
Stream lang: 

Track ID:    4608
Stream type: PGS
Stream ID:   S_HDMV/PGS
Stream info: Presentation Graphic Stream #0 Resolution: 1920:1080 Frame rate: 25
Stream lang: 

Duration: 00:00:25.305
```

Create a file with the extension `.meta`. For this example, let's use /tmp/3d/demux.meta.
Looking at above track information, the following will extract all of the tracks
including the subtitle one.
```
MUXOPT --no-pcr-on-video-pid --new-audio-pes --demux --vbr  --vbv-len=500
V_MPEG4/ISO/MVC, "/home/nick/Packages/FRIM/x64/olesya1.MTS", insertSEI, contSPS, track=4114
V_MPEG4/ISO/AVC, "/home/nick/Packages/FRIM/x64/olesya1.MTS", insertSEI, contSPS, track=4113
A_AC3, "/home/nick/Packages/FRIM/x64/olesya1.MTS", track=4352
S_HDMV/PGS, "/home/nick/Packages/FRIM/x64/olesya1.MTS", fps=25, track=4608
```
Run `tsmuxer` specifying the output directory to extract each stream into its
own file.

```
nick@polonius:/tmp/3d$ tsMuxeR demux.meta .
Network Optix tsMuxeR.  Version 2.6.11. www.networkoptix.com
Decoding H264 stream (track 1): H.264/MVC Views: 2 Profile: High@4.0  Resolution: 1920:1080i  Frame rate: 25
MVC muxing fps is not set. Get fps from stream. Value: 25
B-pyramid level 2 detected. Shift DTS to 3 frames
Decoding H264 stream (track 2): Profile: High@4.0  Resolution: 1920:1080i  Frame rate: 25
H.264 muxing fps is not set. Get fps from stream. Value: 25
B-pyramid level 2 detected. Shift DTS to 3 frames
0.3% complete
Decoding AC3 stream (track 3): Bitrate: 256Kbps Sample Rate: 48KHz Channels: 2
Decoding PGS stream (track 4):  Resolution: 1920:1080  Frame rate: 25
96.9% complete
Processed 624 video frames
Processed 624 video frames
100.0% complete
Flushing write buffer
Demux complete.
Demuxing time: 3 sec
nick@polonius:/tmp/3d$ ls -l
total 73356
-rw-r--r-- 1 nick nick      394 Jun  5 17:41 demux.meta
-rw-r----- 1 nick nick 37141757 Jun  5 17:43 olesya1.track_4113.264
-rw-r----- 1 nick nick 36998546 Jun  5 17:43 olesya1.track_4114.mvc
-rw-r----- 1 nick nick   798720 Jun  5 17:43 olesya1.track_4352.ac3
-rw-r----- 1 nick nick   162552 Jun  5 17:43 olesya1.track_4608.sup
```

