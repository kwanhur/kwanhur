---
title: build-imagemagick-libwebp-with-jpeg-turbo-from-source
date: 2018-04-09 13:46:26
tags:
    - lua
    - webp
    - imagemagick
---

对于喜欢折腾的人，安装所需系统库都是手工源码编译出来使用，这或许有种可控的感觉。

相比直接使用软件库已编译好的，源码编译可更加自由选择或指定版本。

近期在CentOS-7.2系统上，升级webp图片依赖系统库时，整理了个一键编译ImageMagick脚本，如下：

<!--more-->

```bash
$ head  -n 1 /etc/redhat-release 

CentOS Linux release 7.2.1511 (Core) 

$ uname -r
3.10.0-327.el7.x86_64

mkdir -p /export/svr /export/lib
DIR=/export/svr
PREFIX=/export/lib

JPEG_VER=1.5.2
PNG_VER=1.6.34
GIF_VER=5.1.4
WEBP_VER=0.6.1
IMAGICK_VER=7.0.7-26

if [ ! -e $PREFIX/libjpeg-turbo-$JPEG_VER.tar.gz ];then
  wget https://jaist.dl.sourceforge.net/project/libjpeg-turbo/$JPEG_VER/libjpeg-turbo-$JPEG_VER.tar.gz -O $PREFIX/libjpeg-turbo-$JPEG_VER.tar.gz
fi

if [ ! -e $PREFIX/libpng-$PNG_VER.tar.gz ];then
  wget https://jaist.dl.sourceforge.net/project/libpng/libpng16/$PNG_VER/libpng-$PNG_VER.tar.gz -O $PREFIX/libpng-$PNG_VER.tar.gz
fi

if [ ! -e $PREFIX/giflib-$GIF_VER.tar.bz2 ];then
  wget https://jaist.dl.sourceforge.net/project/giflib/giflib-$GIF_VER.tar.bz2 -O $PREFIX/giflib-$GIF_VER.tar.bz2
fi

if [ ! -e $PREFIX/libwebp-$WEBP_VER.tar.gz ];then
  wget https://codeload.github.com/webmproject/libwebp/tar.gz/v$WEBP_VER -O $PREFIX/libwebp-$WEBP_VER.tar.gz
fi

if [ ! -e $PREFIX/ImageMagick-$IMAGICK_VER.tar.gz ];then
  wget https://codeload.github.com/ImageMagick/ImageMagick/tar.gz/$IMAGICK_VER -O $PREFIX/ImageMagick-$IMAGICK_VER.tar.gz
fi

cd $PREFIX
rm -fr libjpeg-turbo-$JPEG_VER libpng-$PNG_VER giflib-$GIF_VER libwebp-$WEBP_VER ImageMagick-$IMAGICK_VER

#base package
yum install -y -q libtool libtool-ltdl libtool-ltdl-devel gcc make autoconf automake readline-devel nasm

#libjpeg-turbo
unlink $PREFIX/libjpeg-turbo
rm -rf $PREFIX/libjpeg-turbo-$JPEG_VER
cd ${DIR}/libjpeg-turbo-$JPEG_VER
make clean >/dev/null
./configure --prefix=$PREFIX/libjpeg-turbo-$JPEG_VER && make -j24 && make -j24 install
ln -sfnv $PREFIX/libjpeg-turbo-$JPEG_VER $PREFIX/libjpeg-turbo

test $? -eq 0 && echo "install_status libjpeg-turbo OK"||echo "install_status libjpeg-turbo failed"
sudo cp -a $PREFIX/libjpeg-turbo/lib/pkgconfig/*.pc /usr/lib64/pkgconfig/
sudo echo "$PREFIX/libjpeg-turbo/lib" > /etc/ld.so.conf.d/jpeg-turbo.conf

#libpng
unlink $PREFIX/libpng
rm -rf $PREFIX/libpng-$PNG_VER
cd ${DIR}/libpng-$PNG_VER
make clean >/dev/null
./configure --prefix=$PREFIX/libpng-$PNG_VER && make -j24 && make -j24 install
ln -sfnv $PREFIX/libpng-$PNG_VER $PREFIX/libpng

test $? -eq 0 && echo "install_status libpng OK"||echo "install_status libpng failed"
sudo cp -a $PREFIX/libpng/lib/pkgconfig/*.pc /usr/lib64/pkgconfig/
sudo echo "$PREFIX/libpng/lib" > /etc/ld.so.conf.d/libpng.conf

#giflib
unlink $PREFIX/giflib
rm -fr $PREFIX/giflib-$GIF_VER
cd ${DIR}/giflib-$GIF_VER
make clean >/dev/null
./configure --prefix=$PREFIX/giflib-$GIF_VER && make -j24 && make install
ln -sfnv $PREFIX/giflib-$GIF_VER $PREFIX/giflib 

test $? -eq 0 && echo "install_status giflib OK" || echo "install_status giflib failed"
sudo echo "$PREFIX/giflib/lib" > /etc/ld.so.conf.d/giflib.conf

#libwebp
cd ${DIR}/libwebp-$WEBP_VER
make clean >/dev/null
unlink $PREFIX/libwebp
rm -rf $PREFIX/libwebp-$WEBP_VER
./configure --prefix=$PREFIX/libwebp-$WEBP_VER --enable-everything --disable-tiff --disable-wic \
--with-gifincludedir=$PREFIX/giflib/include --with-giflibdir=$PREFIX/giflib/lib \
--with-jpegincludedir=$PREFIX/libjpeg-turbo/include --with-jpeglibdir=$PREFIX/libjpeg-turbo/lib \
--with-pngincludedir=$PREFIX/libpng/include --with-pnglibdir=$PREFIX/libpng/lib && make -j24 && make install
ln -sfnv $PREFIX/libwebp-$WEBP_VER $PREFIX/libwebp

test $? -eq 0 && echo "install_status libwebp OK"||echo "install_status libwebp failed"
sudo cp -a $PREFIX/libwebp/lib/pkgconfig/*.pc /usr/lib64/pkgconfig/
sudo echo "$PREFIX/libwebp/lib" > /etc/ld.so.conf.d/libwebp.conf

#webp support yes
cd ${DIR}/ImageMagick-$IMAGICK_VER
unlink $PREFIX/imagemagick
rm -rf $PREFIX/imagemagick-$IMAGICK_VER
make clean >/dev/null 
export PKG_CONFIG=/usr/bin/pkg-config
export CPPFLAGS="-I$PREFIX/libjpeg-turbo/include"
export LDFLAGS="-L$PREFIX/libjpeg-turbo/lib"
./configure --prefix=$PREFIX/imagemagick-$IMAGICK_VER --enable-static --enable-shared  --disable-docs  \
--with-modules --with-webp=yes --disable-openmp --enable-delegate-build && make -j 24 && make install
ln -sfnv $PREFIX/imagemagick-$IMAGICK_VER $PREFIX/imagemagick

test $? -eq 0 && echo "install_status imagemagick OK"||echo "install_status imagemagick failed"
sudo cp -a $PREFIX/imagemagick/lib/pkgconfig/*.pc /usr/lib64/pkgconfig/
sudo echo "$PREFIX/imagemagick/lib" > /etc/ld.so.conf.d/imagemagick.conf

#pacakge imagemagick with dependent libraries
VERSION=7.0.7-26
LIB_PKG=/usr/lib64/pkgconfig
LD_CONF=/etc/ld.so.conf.d

tar -jPcf imagemagick-$VERSION.tar.bz2 $PREFIX/imagemagick $PREFIX/imagemagick-7.0.7-26 $PREFIX/giflib \
$PREFIX/giflib-$GIF_VER $PREFIX/libpng $PREFIX/libpng-$PNG_VER $PREFIX/libwebp $PREFIX/libwebp-$WEBP_VER \
$PREFIX/libjpeg-turbo $PREFIX/libjpeg-turbo-$JPEG_VER \
$LIB_PKG/ImageMagick-7.Q16HDRI.pc $LIB_PKG/ImageMagick.pc $LIB_PKG/Magick++-7.Q16HDRI.pc $LIB_PKG/MagickCore-7.Q16HDRI.pc \
$LIB_PKG/MagickCore.pc $LIB_PKG/Magick++.pc $LIB_PKG/MagickWand-7.Q16HDRI.pc $LIB_PKG/MagickWand.pc $LIB_PKG/libwebpdecoder.pc \
$LIB_PKG/libwebpdemux.pc $LIB_PKG/libwebpmux.pc $LIB_PKG/libwebp.pc $LIB_PKG/libjpeg.pc $LIB_PKG/libturbojpeg.pc \
$LD_CONF/jpeg-turbo.conf $LD_CONF/libwebp.conf $LD_CONF/imagemagick.conf $LD_CONF/giflib.conf $LD_CONF/libpng.conf
```

通过该系统库，完美适配 [lua-resty-imagick](https://github.com/kwanhur/lua-resty-imagick) :)
