ffmpeg-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of ffmpeg (license: GPL-2.0-or-later) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  zlib.tar.gz
    https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
    sha256 9a93b2b7dfdac77ceba5a558a580e74667dd6fede4585b91eefb60f03b72df23
  libogg.tar.gz
    https://downloads.xiph.org/releases/ogg/libogg-1.3.6.tar.gz
    sha256 83e6704730683d004d20e21b8f7f55dcb3383cdf84c0daedf30bde175f774638
  libvorbis.tar.gz
    https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.7.tar.gz
    sha256 0e982409a9c3fc82ee06e08205b1355e5c6aa4c36bca58146ef399621b0ce5ab
  lame.tar.gz
    https://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz
    sha256 ddfe36cab873794038ae2c1210557ad34857a4b6bdc515785d1da9e175b1da1e
  opus.tar.gz
    https://github.com/xiph/opus/releases/download/v1.5.2/opus-1.5.2.tar.gz
    sha256 65c1d2f78b9f2fb20082c38cbe47c951ad5839345876e46941612ee87f9a7ce1
  x264.tar.gz
    https://code.videolan.org/videolan/x264/-/archive/b35605ace3ddf7c1a5d67a2eb553f034aef41d55/x264-b35605ace3ddf7c1a5d67a2eb553f034aef41d55.tar.gz
    sha256 cd71a7515b0e9a012e1ac9b1f8415bebcaf6fc97d4db32286642ac4c0fbe24f9
  x265.tar.gz
    https://download.videolan.org/pub/videolan/x265/x265_4.2.tar.gz
    sha256 40b1ea0453e0309f0eba934e0ddf533f8f6295966679e8894e8f1c1c8d5e1210
  libvpx.tar.gz
    https://github.com/webmproject/libvpx/archive/refs/tags/v1.16.0.tar.gz
    sha256 7a479a3c66b9f5d5542a4c6a1b7d3768a983b1e5c14c60a9396edc9b649e015c
  dav1d.tar.xz
    https://download.videolan.org/pub/videolan/dav1d/1.5.3/dav1d-1.5.3.tar.xz
    sha256 732010aa5ef461fa93355ed2c6c5fedb48ddc4b74e697eaabe8907eaeb943011
  fribidi.tar.xz
    https://github.com/fribidi/fribidi/releases/download/v1.0.16/fribidi-1.0.16.tar.xz
    sha256 1b1cde5b235d40479e91be2f0e88a309e3214c8ab470ec8a2744d82a5a9ea05c
  freetype.tar.xz
    https://download.savannah.gnu.org/releases/freetype/freetype-2.14.3.tar.xz
    sha256 36bc4f1cc413335368ee656c42afca65c5a3987e8768cc28cf11ba775e785a5f
  harfbuzz.tar.xz
    https://github.com/harfbuzz/harfbuzz/releases/download/14.2.0/harfbuzz-14.2.0.tar.xz
    sha256 94017020f96d025bb66ae91574e4cf334bcad23e8175a8a40565b3721bc2eaff
  libass.tar.gz
    https://github.com/libass/libass/releases/download/0.17.4/libass-0.17.4.tar.gz
    sha256 a886b3b80867f437bc55cff3280a652bfa0d37b43d2aff39ddf3c4f288b8c5a8
  ffmpeg.tar.xz
    https://ffmpeg.org/releases/ffmpeg-8.1.1.tar.xz
    sha256 b6863adde98898f42602017462871b5f6333e65aec803fdd7a6308639c52edf3
