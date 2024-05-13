# Cross-compiling pkg-config for Windows(static build)

## Installing build tools

```bash
sudo apt install git autoconf automake libtool ninja-build mingw-w64 mingw-w64-tools

sudo update-alternatives --set x86_64-w64-mingw32-g++ /usr/bin/x86_64-w64-mingw32-g++-posix
sudo update-alternatives --set x86_64-w64-mingw32-gcc /usr/bin/x86_64-w64-mingw32-gcc-posix
sudo update-alternatives --set i686-w64-mingw32-g++ /usr/bin/i686-w64-mingw32-g++-posix
sudo update-alternatives --set i686-w64-mingw32-gcc /usr/bin/i686-w64-mingw32-gcc-posix
```

## Installing latest meson

`meson` installed with `apt` needs to be removed. After installing `meson` with `pip`, set the PATH. Success if meson `version` is 1.4.0 or higher.

```bash
sudo apt purge meson

sudo apt install python3-pip
python3 -m pip install meson

export PATH=$PATH:~/.local/bin

# Uncomment the following lines if necessary.
# echo -e '\nexport PATH=$PATH:~/.local/bin' >> ~/.bash_profile
# source ~/.bash_profile

meson --version
```

## Checking out this repository(pkg-config-for-windows)

Since `cross_file.txt` contains.

```bash
git clone https://github.com/sincos2854/pkg-config-for-windows.git
```

## Checking out GLib and pkg-config

```bash
cd ./pkg-config-for-windows
git clone -b 2.80.2 --depth 1 https://gitlab.gnome.org/GNOME/glib.git
git clone -b pkg-config-0.29.2 --depth 1 https://gitlab.freedesktop.org/pkg-config/pkg-config.git
```

## Building Glib

```bash
export GLIB_OUT_DIR=$(pwd)/glib_out
export PKG_OUT_DIR=$(pwd)/pkg_out

cd ./glib

meson setup build \
--cross-file=../cross_file.txt \
--prefix=$GLIB_OUT_DIR \
--buildtype=release \
--strip \
--default-library=static \
-Db_vscrt=static_from_buildtype \
-Dtests=false

meson compile -C build
meson install -C build
```

## Building pkg-config

```bash
cd ../pkg-config

./autogen.sh \
--host=x86_64-w64-mingw32 \
--prefix=$PKG_OUT_DIR \
--enable-static \
PKG_CONFIG_LIBDIR=$GLIB_OUT_DIR/lib/pkgconfig \
LIBS="-lole32 -lshell32 -luser32 -lws2_32 -luuid" \
LDFLAGS="-static -s"

make
make install

cd ../
```

## Copying pkg-config.exe

```bash
mkdir windows
cp $PKG_OUT_DIR/bin/pkg-config.exe ./windows/
```

For static build, `libglib-2.0-0.dll` and `libintl-8.dll` are not required.
