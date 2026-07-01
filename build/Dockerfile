FROM emscripten/emsdk:5.0.7 AS builder

SHELL ["/bin/bash", "-c"]

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
      autoconf automake libtool pkg-config make git xz-utils \
      python3-pip \
    && pip3 install --break-system-packages --no-cache-dir meson ninja \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN cp /usr/share/automake-*/config.sub  /usr/local/share/config.sub \
    && cp /usr/share/automake-*/config.guess /usr/local/share/config.guess \
    && /usr/local/share/config.sub wasm32-unknown-emscripten \
    && printf '%s\n' '#!/bin/bash' 'set -e' \
        'find . -name config.sub   -exec cp /usr/local/share/config.sub   {} +' \
        'find . -name config.guess -exec cp /usr/local/share/config.guess {} +' \
        > /usr/local/bin/fixconfig \
    && chmod +x /usr/local/bin/fixconfig

ENV PREFIX=/opt
ENV PKG_CONFIG_PATH=${PREFIX}/lib/pkgconfig
ENV EM_PKG_CONFIG_PATH=${PREFIX}/lib/pkgconfig

ENV CFLAGS="-O3 -msimd128 -pthread"
ENV CXXFLAGS="-O3 -msimd128 -pthread"
ENV LDFLAGS="-pthread"

WORKDIR /src
COPY meson-cross.txt /src/meson-cross.txt

ADD https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz zlib.tar.gz
RUN echo "9a93b2b7dfdac77ceba5a558a580e74667dd6fede4585b91eefb60f03b72df23  zlib.tar.gz" | sha256sum -c - \
    && tar -xzf zlib.tar.gz && rm zlib.tar.gz && cd zlib-1.3.1 \
    && emconfigure ./configure --prefix=${PREFIX} --static \
    && emmake make -j$(nproc) install

ADD https://downloads.xiph.org/releases/ogg/libogg-1.3.6.tar.gz libogg.tar.gz
RUN echo "83e6704730683d004d20e21b8f7f55dcb3383cdf84c0daedf30bde175f774638  libogg.tar.gz" | sha256sum -c - \
    && tar -xzf libogg.tar.gz && rm libogg.tar.gz && cd libogg-1.3.6 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${PREFIX} \
        --disable-shared --enable-static \
    && emmake make -j$(nproc) install

ADD https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.7.tar.gz libvorbis.tar.gz
RUN echo "0e982409a9c3fc82ee06e08205b1355e5c6aa4c36bca58146ef399621b0ce5ab  libvorbis.tar.gz" | sha256sum -c - \
    && tar -xzf libvorbis.tar.gz && rm libvorbis.tar.gz && cd libvorbis-1.3.7 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${PREFIX} \
        --disable-shared --enable-static --disable-oggtest --with-ogg=${PREFIX} \
    && emmake make -j$(nproc) install

ADD https://downloads.sourceforge.net/project/lame/lame/3.100/lame-3.100.tar.gz lame.tar.gz
RUN echo "ddfe36cab873794038ae2c1210557ad34857a4b6bdc515785d1da9e175b1da1e  lame.tar.gz" | sha256sum -c - \
    && tar -xzf lame.tar.gz && rm lame.tar.gz && cd lame-3.100 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${PREFIX} \
        --disable-shared --enable-static --disable-frontend --disable-gtktest \
    && emmake make -j$(nproc) install

ADD https://github.com/xiph/opus/releases/download/v1.5.2/opus-1.5.2.tar.gz opus.tar.gz
RUN echo "65c1d2f78b9f2fb20082c38cbe47c951ad5839345876e46941612ee87f9a7ce1  opus.tar.gz" | sha256sum -c - \
    && tar -xzf opus.tar.gz && rm opus.tar.gz && cd opus-1.5.2 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${PREFIX} \
        --disable-shared --enable-static --disable-doc --disable-extra-programs \
        --disable-intrinsics --disable-rtcd \
    && emmake make -j$(nproc) install

ADD https://code.videolan.org/videolan/x264/-/archive/b35605ace3ddf7c1a5d67a2eb553f034aef41d55/x264-b35605ace3ddf7c1a5d67a2eb553f034aef41d55.tar.gz x264.tar.gz
RUN echo "cd71a7515b0e9a012e1ac9b1f8415bebcaf6fc97d4db32286642ac4c0fbe24f9  x264.tar.gz" | sha256sum -c - \
    && tar -xzf x264.tar.gz && rm x264.tar.gz \
    && cd x264-b35605ace3ddf7c1a5d67a2eb553f034aef41d55 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-linux --prefix=${PREFIX} \
        --enable-static --disable-shared --disable-cli --disable-asm \
        --disable-opencl --enable-pic --enable-thread \
    && emmake make -j$(nproc) install

ADD https://download.videolan.org/pub/videolan/x265/x265_4.2.tar.gz x265.tar.gz
RUN echo "40b1ea0453e0309f0eba934e0ddf533f8f6295966679e8894e8f1c1c8d5e1210  x265.tar.gz" | sha256sum -c - \
    && tar -xzf x265.tar.gz && rm x265.tar.gz && cd x265_4.2 \
    && sed -i 's/^set(X86_ALIASES .*/set(X86_ALIASES)/' source/CMakeLists.txt \
    && grep -q '^set(X86_ALIASES)$' source/CMakeLists.txt \
       || { echo "FATAL: x265 X86_ALIASES patch did not apply"; exit 1; } \
    && emcmake cmake -S source -B build -DCMAKE_INSTALL_PREFIX=${PREFIX} \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_FLAGS="${CFLAGS}" -DCMAKE_CXX_FLAGS="${CXXFLAGS}" \
        -DENABLE_SHARED=OFF -DENABLE_CLI=OFF -DENABLE_ASSEMBLY=OFF \
        -DENABLE_LIBNUMA=OFF -DENABLE_PIC=ON \
    && cmake --build build -j$(nproc) --target install

ADD https://github.com/webmproject/libvpx/archive/refs/tags/v1.16.0.tar.gz libvpx.tar.gz
RUN echo "7a479a3c66b9f5d5542a4c6a1b7d3768a983b1e5c14c60a9396edc9b649e015c  libvpx.tar.gz" | sha256sum -c - \
    && tar -xzf libvpx.tar.gz && rm libvpx.tar.gz && cd libvpx-1.16.0 \
    && emconfigure ./configure --target=generic-gnu --prefix=${PREFIX} \
        --enable-static --disable-shared --enable-pic --enable-multithread \
        --disable-examples --disable-tools --disable-docs --disable-unit-tests \
        --disable-runtime-cpu-detect \
        --extra-cflags="${CFLAGS}" \
    && emmake make -j$(nproc) install

ADD https://download.videolan.org/pub/videolan/dav1d/1.5.3/dav1d-1.5.3.tar.xz dav1d.tar.xz
RUN echo "732010aa5ef461fa93355ed2c6c5fedb48ddc4b74e697eaabe8907eaeb943011  dav1d.tar.xz" | sha256sum -c - \
    && tar -xJf dav1d.tar.xz && rm dav1d.tar.xz && cd dav1d-1.5.3 \
    && meson setup build --cross-file /src/meson-cross.txt --prefix=${PREFIX} \
        --buildtype=release --default-library=static \
        -Denable_tools=false -Denable_tests=false -Denable_asm=false \
    && ninja -C build install

RUN git clone --depth 1 --branch v3.14.0 https://aomedia.googlesource.com/aom aom \
    && [ "$(git -C aom rev-parse HEAD)" = "047d8cf6168feafe1300eb6902000dd1a03d5549" ] \
       || { echo "FATAL: aom tag v3.14.0 commit mismatch"; exit 1; } \
    && cd aom \
    && emcmake cmake -B build -DCMAKE_INSTALL_PREFIX=${PREFIX} -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_FLAGS="${CFLAGS}" -DCMAKE_CXX_FLAGS="${CXXFLAGS}" \
        -DAOM_TARGET_CPU=generic -DENABLE_NASM=0 -DCONFIG_RUNTIME_CPU_DETECT=0 \
        -DCONFIG_MULTITHREAD=1 \
        -DCONFIG_AV1_DECODER=0 -DCONFIG_AV1_ENCODER=1 \
        -DENABLE_TESTS=0 -DENABLE_TOOLS=0 -DENABLE_EXAMPLES=0 -DENABLE_DOCS=0 \
    && cmake --build build -j$(nproc) --target install

ADD https://github.com/fribidi/fribidi/releases/download/v1.0.16/fribidi-1.0.16.tar.xz fribidi.tar.xz
RUN echo "1b1cde5b235d40479e91be2f0e88a309e3214c8ab470ec8a2744d82a5a9ea05c  fribidi.tar.xz" | sha256sum -c - \
    && tar -xJf fribidi.tar.xz && rm fribidi.tar.xz && cd fribidi-1.0.16 \
    && meson setup build --cross-file /src/meson-cross.txt --prefix=${PREFIX} \
        --buildtype=release --default-library=static \
        -Ddocs=false -Dtests=false -Dbin=false \
    && ninja -C build install

ADD https://download.savannah.gnu.org/releases/freetype/freetype-2.14.3.tar.xz freetype.tar.xz
RUN echo "36bc4f1cc413335368ee656c42afca65c5a3987e8768cc28cf11ba775e785a5f  freetype.tar.xz" | sha256sum -c - \
    && tar -xJf freetype.tar.xz && rm freetype.tar.xz && cd freetype-2.14.3 \
    && emcmake cmake -B build -DCMAKE_INSTALL_PREFIX=${PREFIX} -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_C_FLAGS="${CFLAGS}" \
        -DBUILD_SHARED_LIBS=OFF \
        -DFT_DISABLE_HARFBUZZ=ON -DFT_DISABLE_BROTLI=ON -DFT_DISABLE_BZIP2=ON \
        -DFT_DISABLE_ZLIB=ON -DFT_DISABLE_PNG=ON \
    && cmake --build build -j$(nproc) --target install

ADD https://github.com/harfbuzz/harfbuzz/releases/download/14.2.0/harfbuzz-14.2.0.tar.xz harfbuzz.tar.xz
RUN echo "94017020f96d025bb66ae91574e4cf334bcad23e8175a8a40565b3721bc2eaff  harfbuzz.tar.xz" | sha256sum -c - \
    && tar -xJf harfbuzz.tar.xz && rm harfbuzz.tar.xz && cd harfbuzz-14.2.0 \
    && meson setup build --cross-file /src/meson-cross.txt --prefix=${PREFIX} \
        --buildtype=release --default-library=static \
        -Dtests=disabled -Ddocs=disabled -Dutilities=disabled -Dbenchmark=disabled \
        -Dicu=disabled -Dglib=disabled -Dgobject=disabled -Dcairo=disabled \
        -Dfreetype=enabled -Dchafa=disabled \
    && ninja -C build install

ADD https://github.com/libass/libass/releases/download/0.17.4/libass-0.17.4.tar.gz libass.tar.gz
RUN echo "a886b3b80867f437bc55cff3280a652bfa0d37b43d2aff39ddf3c4f288b8c5a8  libass.tar.gz" | sha256sum -c - \
    && tar -xzf libass.tar.gz && rm libass.tar.gz && cd libass-0.17.4 && fixconfig \
    && emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${PREFIX} \
        --disable-shared --enable-static --disable-fontconfig --disable-directwrite \
        --disable-coretext --disable-require-system-font-provider --disable-asm \
    && emmake make -j$(nproc) install

ARG VERSION=8.1.1
ENV VERSION=${VERSION}
ADD https://ffmpeg.org/releases/ffmpeg-${VERSION}.tar.xz ffmpeg.tar.xz
RUN echo "b6863adde98898f42602017462871b5f6333e65aec803fdd7a6308639c52edf3  ffmpeg.tar.xz" | sha256sum -c - \
    && tar -xJf ffmpeg.tar.xz && rm ffmpeg.tar.xz

WORKDIR /src/ffmpeg-${VERSION}
COPY js ./js

ENV FFMPEG_LINK_MODULE="\
    /tmp/wasm-stubs.o \
    -O3 \
    -pthread \
    -sPROXY_TO_PTHREAD \
    -sPTHREAD_POOL_SIZE=8 \
    --post-js js/post.js \
    -sWASM_BIGINT=1 \
    -sALLOW_MEMORY_GROWTH=1 \
    -sINITIAL_MEMORY=268435456 \
    -sMAXIMUM_MEMORY=2147483648 \
    -sSTACK_SIZE=5242880 \
    -sEXIT_RUNTIME=1 \
    -sFORCE_FILESYSTEM=1 \
    -sERROR_ON_UNDEFINED_SYMBOLS=0 \
    -sEXPORTED_RUNTIME_METHODS=FS,NODEFS,WORKERFS,ENV,callMain \
    -sINCOMING_MODULE_JS_API=arguments,preRun,postRun,print,printErr,onExit,onAbort,wasmBinary,locateFile,instantiateWasm,mainScriptUrlOrBlob \
    -sMODULARIZE=1 \
    -sEXPORT_ES6=1 \
    -lnodefs.js -lworkerfs.js \
"

RUN printf '%s\n' \
      '#include <errno.h>' \
      'int sched_getaffinity(int p, unsigned long sz, void *m) {' \
      '    (void)p; (void)sz; (void)m; errno = ENOSYS; return -1; }' \
      > /tmp/wasm-stubs.c \
    && emcc -O2 -pthread -c /tmp/wasm-stubs.c -o /tmp/wasm-stubs.o \
    && emconfigure ./configure \
        --prefix=${PREFIX} \
        --target-os=none --arch=wasm32 --cpu=generic \
        --enable-cross-compile \
        --cc=emcc --cxx=em++ --ld=em++ --ar=emar --ranlib=emranlib --nm=emnm \
        --objcc=emcc --dep-cc=emcc \
        --disable-asm --disable-x86asm --disable-inline-asm \
        --disable-stripping --disable-debug --disable-doc \
        --disable-autodetect --disable-network --disable-runtime-cpudetect \
        --disable-programs --enable-ffmpeg \
        --enable-pthreads \
        --disable-decoder=libaom_av1 \
        --enable-gpl \
        --enable-zlib --enable-iconv \
        --enable-libx264 --enable-libx265 --enable-libvpx --enable-libaom \
        --enable-libdav1d --enable-libmp3lame --enable-libopus --enable-libvorbis \
        --enable-libass --enable-libfreetype --enable-libharfbuzz --enable-libfribidi \
        --pkg-config-flags="--static" \
        --extra-cflags="-I${PREFIX}/include ${CFLAGS}" \
        --extra-cxxflags="-I${PREFIX}/include ${CXXFLAGS}" \
        --extra-ldflags="-L${PREFIX}/lib ${LDFLAGS}" \
        --extra-ldexeflags="${FFMPEG_LINK_MODULE}" \
       || { echo "=== ffmpeg configure FAILED — config.log tail ==="; \
            tail -n 90 ffbuild/config.log; exit 1; } \
    && echo "=== verifying external libraries were detected ===" \
    && for lib in LIBX264 LIBX265 LIBVPX LIBAOM LIBDAV1D LIBMP3LAME LIBOPUS \
                  LIBVORBIS LIBASS LIBFREETYPE LIBHARFBUZZ LIBFRIBIDI; do \
         grep -q "^CONFIG_${lib}=yes" ffbuild/config.mak \
           || { echo "FATAL: ffmpeg configure did not enable ${lib}"; exit 1; }; \
       done \
    && grep -q '^CONFIG_FFMPEG=yes' ffbuild/config.mak \
       || { echo "FATAL: the ffmpeg program was not enabled"; exit 1; } \
    && for h in HAVE_SYSCTL HAVE_GETHRTIME HAVE_SCHED_GETAFFINITY; do \
         sed -i "s/#define ${h} 1/#define ${h} 0/" config.h; \
       done \
    && ! grep -qE '#define (HAVE_SYSCTL|HAVE_GETHRTIME|HAVE_SCHED_GETAFFINITY) 1' config.h \
       || { echo "FATAL: a misdetected HAVE_* macro was not patched off"; exit 1; } \
    && echo "=== all external libraries detected — building ===" \
    && emmake make -j$(nproc) EXESUF=.mjs ffmpeg.mjs

RUN mkdir -p /dist \
    && cp ffmpeg_g.mjs  /dist/ffmpeg.mjs \
    && cp ffmpeg_g.wasm /dist/ffmpeg.wasm \
    && sed -i 's/ffmpeg_g\./ffmpeg./g' /dist/ffmpeg.mjs \
    && ! grep -q 'ffmpeg_g\.' /dist/ffmpeg.mjs \
       || { echo "FATAL: an ffmpeg_g. reference was left unpatched in ffmpeg.mjs"; exit 1; } \
    && grep -q 'ffmpeg\.wasm' /dist/ffmpeg.mjs \
       || { echo "FATAL: wasm filename reference not patched in ffmpeg.mjs"; exit 1; } \
    && [ "$(head -c4 /dist/ffmpeg.wasm | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
    && [ "$(stat -c%s /dist/ffmpeg.wasm)" -gt 1048576 ] \
    && ls -la /dist

FROM scratch AS exportor
COPY --from=builder /dist /dist
