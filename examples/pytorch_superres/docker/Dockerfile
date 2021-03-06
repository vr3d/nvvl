FROM nvcr.io/nvidia/pytorch:18.03-py3

ARG FFMPEG_VERSION=3.4.2

# nvcuvid deps
RUN apt-get update --fix-missing && \
    apt-get install -y libx11-6 libxext6
ENV NVIDIA_DRIVER_CAPABILITIES=video,compute,utility

# minimal ffmpeg from source
RUN apt-get install -y yasm wget && \
    cd /tmp && wget -q http://ffmpeg.org/releases/ffmpeg-$FFMPEG_VERSION.tar.bz2 && \
    tar xf ffmpeg-$FFMPEG_VERSION.tar.bz2 && \
    rm ffmpeg-$FFMPEG_VERSION.tar.bz2 && \
    cd ffmpeg-$FFMPEG_VERSION && \
    ./configure \
      --prefix=/usr/local \
      --disable-static \
      --disable-all \
      --disable-autodetect \
      --disable-iconv \
      --enable-shared \
      --enable-avformat \
      --enable-avcodec \
      --enable-avfilter \
      --enable-protocol=file \
      --enable-demuxer=mov,matroska \
      --enable-bsf=h264_mp4toannexb,hevc_mp4toannexb && \
    make -j8 && make install && \
    cd /tmp && rm -rf ffmpeg-$FFMPEG_VERSION && \
    apt-get remove -y yasm

# install stub library since driver libs aren't available at image build time
# this is a temporary requirement that will go away in future cuda versions
# libnvcuvid.so was created using the make-stub.sh script
COPY libnvcuvid.so /usr/local/cuda/lib64/stubs

# install nvvl
RUN pip install --upgrade cmake && \
    apt-get install -y pkg-config && \
    cd /tmp && \
    wget --no-check-certificate https://gitlab-master.nvidia.com/ADLR/nvvl/repository/archive.tar.bz2 -O nvvl.tar.bz2 && \
    mkdir -p nvvl/build && \
    tar xf nvvl.tar.bz2 -C nvvl --strip-components 1 && \
    rm nvvl.tar.bz2 && \
    cd nvvl/build && \
    cmake .. && make && make install && \
    cd ../pytorch && \
    python3 setup.py install && \
    pip uninstall -y cmake && \
    apt-get remove -y pkg-config && \
    apt-get autoremove -y

RUN pip install scikit-image tensorflow tensorboard tensorboardX

WORKDIR /workspace/examples/pytorch_superres
