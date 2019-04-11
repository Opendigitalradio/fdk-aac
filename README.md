A patched version of fdk-aac with DAB+ support
==============================================

This is a modified version of fdk-aac that supports the AOTs
required for DAB+ encoding.

This library is a dependency for https://github.com/Opendigitalradio/ODR-AudioEnc/

See http://www.opendigitalradio.org for more

Installation
============

Make sure you have installed git, otherwise install it with `sudo apt-get install git`

    $ git clone https://github.com/Opendigitalradio/fdk-aac.git
    $ cd fdk-aac/
    $ ./bootstrap
    $ ./configure 
    $ make
    $ sudo make install
