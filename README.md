Based on: lebougui/hls-creator. Playing with the script, fixing a mistake in the  (bandwidth should be in bytes), debugging, extending.

-r 960x540,640x360,320x180

Add resolutions for the streams (originally all is encoded in source resolution)

hls-creator
===========

This script has been forked from https://github.com/bentasker/HLS-Stream-Creator project.
Renamed script from "HLS-Stream-Creator.sh" to "hls.sh".
Some issues have been fixed + new params.


hls.sh is a simple BASH Script designed to take a media file, segment it and create an M3U8 playlist for serving using HLS.
There are numerous tools out there which are far better suited to the task, and offer many more options. This project only exists because I was asked to look
into HTTP Live Streaming in depth, so after reading the [IETF Draft](http://tools.ietf.org/html/draft-pantos-http-live-streaming-11 "HLS on IETF") I figured I'd start with the basics by creating a script to encode arbitrary video into a VOD style HLS feed.


I-FRAME Creator(iframe.sh) is used to generate I-Frame Playlist from existing HLS Playlist.

Pre-requisites
-------------
ffmeg must be installed.


Usage
------

Usage is incredibly simple

```
./hls.sh -[lf] [-c segmentcount] -i [inputfile] -s [segmentlength(seconds)] -o [outputdir] -b [bitrates]


Deprecated Legacy usage:
	hls.sh inputfile segmentlength(seconds) [outputdir='./output']

```

So to split a video file called *example.avi* into segments of 10 seconds, we'd run

```
./hls.sh -i example.avi -s 10
```
Todor: In order to split it in about 10 s, there should be keyframes around that range.
Usually that's done by transcoding with forcing keyframes each 2 seconds (or whatever), for 30 fps e.g.:

-g 60 -sc_threshold 0

Otherwise the lenght of the segments may vary a lot.

**Arguments**

```
    Mandatory Arguments:

	-i [file]	Input file or UR
	-s [s]  	Segment length (seconds)
	-b [birates]    Output video Bitrates in bits (not kbits), comma seperated list (no spaces) for adaptive streams: 1300000,730000, ... [TO DO: convert to kbits, take in kbits*1000]
	-r [resolutions] Per each -b param eter: 960x540,640x360, ...


    Optional Arguments:

	-o [directory]	Output directory (default: ./output)
	-c [count]	Number of segments to include in playlist (live streams only) - 0 is no limit
	 //was -b [bitrates]	Output video Bitrates in kb/s (comma seperated list for adaptive streams)
	-p [name]	Playlist filename prefix
	-t [name]	Segment filename prefix
	-l		Input is a live stream
	-m		Create HLS Segments only with Video Stream
	-f		Foreground encoding only (adaptive non-live streams only)
	-S		Name of a subdirectory to put segments into
	
```


Adaptive Streams
------------------

As of [HLS-6](http://projects.bentasker.co.uk/jira_projects/browse/HLS-6.html) the script can now generate adaptive streams with a top-level variant playlist for both VoD and Linear input streams.

In order to create seperate bitrate streams, pass a comma seperated list in with the *-b* option

```
./hls.sh -i example.avi -s 10 -b 28000,64000,128000,256000, 2000000
```

By default, transcoding for each bitrate will be forked into the background - if you wish to process the bitrates sequentially, pass the *-f* option

```
./hls.sh -i example.avi -s 10 -b 365000,2000000,6000000 -f
```

In either case, in accordance with the HLS spec, the audio bitrate will remain unchanged


Output
-------

As of version 1, the HLS resources will be output to the directory *output*. These will consist of video segments encoded in H.264 with AAC audio and an m3u8 file in the format

>\#EXTM3U  
>\#EXT-X-MEDIA-SEQUENCE:0  
>\#EXT-X-VERSION:3  
>\#EXT-X-TARGETDURATION:10  
>\#EXTINF:10, no desc  
>example_00001.ts  
>\#EXTINF:10, no desc  
>example_00002.ts  
>\#EXTINF:10, no desc  
>example_00003.ts  
>\#EXTINF:5, no desc  
>example_00004.ts  
>\#EXT-X-ENDLIST


The alternative audio media playlists with audio segments are created.
The appropriate EXT-X-MEDIA entries are added into TLM Playlist and a new attribute "AUDIO=" to each video playlist entry. like :

>\#EXT-X-STREAM-INF:BANDWIDTH=41014,AUDIO="aac"  
>test/41014/test_41014.m3u8  
>  
>  
>\#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="aac",LANGUAGE="eng",NAME="ENG",AUTOSELECT=YES,DEFAULT=NO,URI="test_Alt_Audio_Eng_AAC/test_Alt_Audio_Eng_AAC.m3u8"  



Using a Specific FFMPEG binary
-------------------------------

There may be occasions where you don't want to use the *ffmpeg* that appears in PATH. At the top of the script, the ffmpeg command is defined, so change this to suit your needs

```
FFMPEG='/path/to/different/ffmpeg'
```


Additional Environment Variables
-------------------------------

There are few environment variables which can control the ffmpeg behaviour.

* `VIDEO_CODEC` - The encoder which will be used by ffmpeg for video streams. Examples: _libx264_, _nvenc_
* `AUDIO_CODEC` - Encoder for the audio streams. Examples: _aac_, _libfdk_acc_, _mp3_, _libfaac_
* `NUMTHREADS` - A number which will be passed to the `-threads` argument of ffmpeg. Newer ffmpegs with modern libx264 encoders will use the optimal number of threads by default.
* `FFMPEG_FLAGS` - Additional flags for ffmpeg. They will be passed without any modification.

Example usage:

```
export VIDEO_CODEC="nvenc"
export FFMPEG_FLAGS="-pix_fmt yuv420p -profile:v"
./hls.sh example.avi 10
```


I-FRAME Creator
==================

Usage
------

Usage is incredibly simple


```
Usage : ./iframe.sh -h -l <master playlist location> -t <iframe type >

Retrieve all playlists file from a folder and generate the two iframes playlists :
- byte range based ones
- transport streams based ones.

OPTIONS :
        -h              displays this help
        -l              master playlist location
        -t              byterange or default

        Examples : 
        ./iframe.sh -l /cdn/video_TLM.m3u8 -t default (Not complete yet)
        ./iframe.sh -l /cdn/video_TLM.m3u8 -t byterange
	
Todor: Or rather ./iframe.sh -l cdn/video_TLM.m3u8 -t byterange
       (No heading slash)

Version       : ./iframe.sh 1.0 (2016/03/14) 
Maintainer(s) : Lebougi 
```

TODO
------

I-FRAME Creator shall be called by hls.sh to generate I-FRAME Playlist when encoding new HLS

