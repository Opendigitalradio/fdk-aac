fdk-aac-dabplus Package
=======================

This package contains several tools that use the standalone library
of the Fraunhofer FDK AAC code from Android, patched for
960-transform to do DAB+ broadcast encoding.

The *dabplus-enc-file-zmq* can encode from a file or pipe source,
and encode to a ZeroMQ output compatible with ODR-DabMux, to a
file or to stdout.

The *dabplus-enc-alsa-zmq* can encode from an ALSA soundcard,
and encode to a ZeroMQ output compatible with ODR-DabMux. It supports
experimental sound card clock drift compensation, that can compensate
for imprecise sound card clocks.

*dabplus-enc-file-zmq* and *dabplus-enc-alsa-zmq* include
support for DAB MOT Slideshow and DLS, written by [CSP](http://rd.csp.it).

To encode DLS and Slideshow data, the *mot-encoder* tool reads images
from a folder and DLS text from a file, and generates the PAD data
for the encoder.

For detailed usage, see the usage screen of the different tools.

More information is available on the
[Opendigitalradio wiki](http://opendigitalradio.org)

How to build
=============

Requirements:

* boost-thread and boost-system
* ImageMagick magickwand (for MOT slideshow)
* The alsa libraries (libasound2)
* Download and install libfec from https://github.com/Opendigitalradio/ka9q-fec
* Download and install ZeroMQ from http://download.zeromq.org/zeromq-4.0.3.tar.gz

This package:

    git clone https://github.com/mpbraendli/fdk-aac-dabplus.git
    cd fdk-aac-dabplus
    ./bootstrap
    ./configure
    make
    sudo make install

* See the possible scenarios below on how to use the tools
* use mot-encoder to encode images into MOT Slideshow


How to use
==========

We assume:

    ALSASRC="default"
    DST="tcp://yourserver:9000"
    BITRATE=64

Scenario 1
----------

Live Stream from ALSA sound card at 32kHz, with ZMQ output for ODR-DabMux:

    dabplus-enc-alsa-zmq -d $ALSASRC -c 2 -r 32000 -b $BITRATE -o $DST

To enable sound card drift compensation, add the option **-D**:

    dabplus-enc-alsa-zmq -d $ALSASRC -c 2 -r 32000 -b $BITRATE -o $DST -D

You might see **U** and **O<number>** appearing on the terminal. They correspond
to audio underruns and overruns that happen due to the different speeds at which
the audio is captured from the soundcard, and encoded into HE-AACv2.

High occurrence of these will lead to audible artifacts.

Scenario 2
----------

Play some local audio source from a file, with ZMQ output for ODR-DabMux. The problem with
playing a file is that dabplus-enc-file-zmq cannot directly be used, because ODR-DabMux
does not back-pressure the encoder, which will therefore encode much faster than realtime.

While this issue is sorted out, the following trick is a very flexible solution: use the
alsa virtual loop soundcard *snd-aloop* in the following way:

    modprobe snd-aloop

This creates a new audio card (usually 'hw:1' but have a look at /proc/asound/card to be sure) that
can then be used for the alsa encoder:

    ./dabplus-enc-alsa-zmq -d hw:1 -c 2 -r 32000 -b 64 -o

Then, you can use any media player that has an alsa output to play whatever source it supports:

    cd your/preferred/music
    mplayer -ao alsa:device=hw=1.1 -srate 32000 -shuffle *

Important: you must specify the correct sample rate on both "sides" of the virtual sound card.


Scenario 3
----------
Live Stream encoding and preparing for DAB muxer, with ZMQ output, at 32kHz, using sox.
This illustrates the fifo input of *dabplus-enc-file-zmq*.


    sox -t alsa $ALSASRC -b 16 -t raw - rate 32k channels 2 | \
    dabplus-enc-file-zmq -r 32000 \
    -i /dev/stdin -b $BITRATE -f raw -a -o $DST -p 53

The -p 53 sets the padlen, compatible with the default mot-encoder setting. mot-encoder needs
to be given the same value for this option.


Scenario 4
----------
Live Stream encoding and preparing for DAB muxer, with FIFO to odr-dabmux, 48kHz, using
arecord.

    arecord -t raw -f S16_LE -c 2 -r 48000 -D plughw:CARD=Loopback,DEV=0,SUBDEV=0 | \
    dabplus-enc-file-zmq -a -b 24 -f raw -c 2 -r 48000 -i /dev/stdin -o - | \
    mbuffer -q -m 10k -P 100 -s 360 > station1.fifo

Here we are using the ALSA plughw feature.

Scenario 5
----------
Wave file encoding, for non-realtime processing

    dabplus-enc-file-zmq -a -b 64 -i wave_file.wav -o station1.dabp


Usage of MOT Slideshow and DLS
==============================

MOT Slideshow is a new feature, which has been tested on several receivers and
using [XPADxpert](http://www.basicmaster.de/xpadxpert/), but is still a work
in progress.

*mot-encoder* reads images from
the specified folder, and generates the PAD data for the encoder. This is
communicated through a fifo to the encoder. It also reads DLS from a file, and
includes this information in the PAD.

*dabplus-enc-file-zmq* and *dabplus-enc-alsa-zmq* can insert the PAD data from
mot-encoder into the bitstream.
The mp2 encoder [toolame-dab](https://github.com/Opendigitalradio/toolame-dab)
can also read *mot-encoder* data.

This is an ongoing development. Make sure you use the same pad length option
for *mot-encoder* and the audio encoder. Only some pad lengths are supported,
please see *mot-encoder*'s help.

Known Limitations
-----------------

*mot-encoder* encodes slides in a 10 second interval, which is not linked
to the rate at which the encoder reads the PAD data.

