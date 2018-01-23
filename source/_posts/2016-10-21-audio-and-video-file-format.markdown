---
layout: post
title: "音频和视频格式"
date: 2016-10-21 15:15:53 +0800
comments: true
categories: [Archive, Archives, iOS Development]
keywords: Audio, Video
description: audio and video format
---

## 音频格式

我们通常说的音频格式准确地讲应该是音频文件格式，它是计算机系统上用于存放数字音频数据的文件格式，也可以看作一个容器。

音频数据的比特分布我们称为音频编码格式，它可以非压缩编码或压缩编码。压缩编码又分为无损压缩和有损压缩。

编码器(codec)就是来编解码原始音频数据的。

声音源 --ADC--> raw audio data --codec--> audio data(uncompressed/compressed) --packed--> audio file format(container format)

>An audio file format is a file format for storing digital audio data on a computer system. The bit layout of the audio data (excluding metadata) is called the audio coding format and can be uncompressed, or compressed to reduce the file size, often using lossy compression. The data can be a raw bitstream in an audio coding format, but it is usually embedded in a container format or an audio data format with defined storage layer.

> It is important to distinguish between the audio coding format, the container containing the raw audio data, and an audio codec. A codec performs the encoding and decoding of the raw audio data while this encoded data is (usually) stored in a container file. Although most audio file formats support only one type of audio coding data (created with an audio coder), a multimedia container format (as Matroska or AVI) may support multiple types of audio and video data.

>There are three major groups of audio file formats:
	•	Uncompressed audio formats, such as WAV, AIFF, AU or raw header-less PCM;
	•	Formats with lossless compression, such as FLAC, Monkey's Audio (filename extension .ape), WavPack (filename extension .wv), TTA, ATRAC Advanced Lossless, ALAC (filename extension .m4a), MPEG-4 SLS, MPEG-4 ALS, MPEG-4 DST, Windows Media Audio Lossless (WMA Lossless), and Shorten (SHN).
	•	Formats with lossy compression, such as Opus, MP3, Vorbis, Musepack, AAC, ATRAC and Windows Media Audio Lossy (WMA lossy).

<!--more-->

## 视频格式

视频文件格式是计算机系统上一种用来存放数字视频数据的文件格式。视频几乎都是以压缩格式的形式存储的以便减小文件大小。

视频文件格式也是一个容器，里面包含编码完的视频和音频数据，同样是使用编码器来完成编解码工作。

> A video file format is a type of file format for storing digital video data on a computer system. Video is almost always stored in compressed form to reduce the file size.

> A video file normally consists of a container format (e.g. Matroska) containing video data in a video coding format (e.g. VP9) alongside audio data in an audio coding format (e.g. Opus). The container format can also contain synchronization information, subtitles, and metadata such as title. A standardized (or in some cases de facto standard) video file type such as .webm is a profilespecified by a restriction on which container format and which video and audio compression formats are allowed.

> The coded video and audio inside a video file container (i.e. not headers, footers and metadata) is called the essence. A program (or hardware) which can decode video or audio is called a codec; playing or encoding a video file will sometimes require the user to install a codec library corresponding to the type of video and audio coding used in the file.

## iOS and Android supported audio & video codec formats

### iOS 

iOS supports many industry-standard and Apple-specific audio formats, including the following:

*	AAC  
*	Apple Lossless (ALAC)  
*	A-law
*	IMA/ADPCM (IMA4)
*	Linear PCM
*	µ-law
*	DVI/Intel IMA ADPCM
*	Microsoft GSM 6.10
*	AES3-2003
	
Preferred Audio Formats in iOS

* For uncompressed (highest quality) audio, use 16-bit, little endian, linear PCM audio data packaged in a CAF file.

* For compressed audio when playing one sound at a time, and when you don’t need to play audio simultaneously with the iPod application, use the AAC format packaged in a CAF or m4a file.
* For less memory usage when you need to play multiple sounds simultaneously, use IMA4 (IMA/ADPCM) compression. This reduces file size but entails minimal CPU impact during decompression. As with linear PCM data, package IMA4 data in a CAF file.
 
iOS supports many industry-standard video formats and compression standards, including the following:

* H.264 video, up to 1.5 Mbps, 640 by 480 pixels, 30 frames per second, Low-Complexity version of the H.264 Baseline Profile with AAC-LC audio up to 160 Kbps, 48 kHz, stereo audio in .m4v, .mp4, and .mov file formats
* H.264 video, up to 768 Kbps, 320 by 240 pixels, 30 frames per second, Baseline Profile up to Level 1.3 with AAC-LC audio up to 160 Kbps, 48 kHz, stereo audio in .m4v, .mp4, and .mov file formats
* MPEG-4 video, up to 2.5 Mbps, 640 by 480 pixels, 30 frames per second, Simple Profile with AAC-LC audio up to 160 Kbps, 48 kHz, stereo audio in .m4v, .mp4, and .mov file formats
* Numerous audio formats, including the ones listed in Audio Technologies

### Android

Audio

* AAC LC
* HE-AACv1 (AAC+)
* HE-AACv2 (enhanced AAC+)
* AAC ELD (enhanced low delay AAC)
* AMR-NB
* AMR-WB
* FLAC
* MP3
* MIDI
* Vorbis
* PCM/WAVE
* Opus

Video

* H.263
* H.264 AVC
* H.265 HEVC
* MPEG-4 SP
* VP8
* VP9

### iOS 和 Android 都支持的音频、视频格式

Audio

* AAC LC (Low-Complexity profile)
* Linear PCM

Video

* H.264 AVC
* MPEG-4 SP (Simple Profile)

Profile

To address various applications ranging from low-quality, low-resolution surveillance cameras to high definition TV broadcasting and DVDs, many video standards group features into profiles and levels. MPEG-4 Part 2 has approximately 21 profiles, including profiles called Simple, Advanced Simple, Main, Core, Advanced Coding Efficiency, Advanced Real Time Simple, etc. The most commonly deployed profiles are Advanced Simple and Simple, which is a subset of Advanced Simple.


##Reference:

[Audio file format](https://en.wikipedia.org/wiki/Audio_file_format)  
[Video file format](https://en.wikipedia.org/wiki/Video_file_format)  
iOS Technology Overview  
[Supported Media Formats](https://developer.android.com/guide/appendix/media-formats.html)  


