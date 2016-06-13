UNIX BUILD NOTES
====================
這是一些關於如何在 Unix 上建立 Bitcoin Core 的筆記。

(for OpenBSD specific instructions, see [build-openbsd.md](build-openbsd.md))

Note
---------------------
總是用絕對路徑來配置和編譯比特幣和相依套件，舉例來說，當決定相依套件的路徑:

	../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX

在這裡 BDB_PREFIX 必須是一個絕對路徑 - 它被定義要使用 $(pwd) 來確定這個絕對路徑的使用。

To Build
---------------------

```bash
./autogen.sh
./configure
make
make install # optional
```

如果相依套件正確的話，這也會建立 bitcoin-qt。

Dependencies
---------------------

需要的相依套件:

 函式庫      | 目的             | 描述
 ------------|------------------|----------------------
 libssl      | Crypto           | Random Number Generation, 橢圓曲線密碼學
 libboost    | Utility          | 函式庫包含執行續、資料結構...等
 libevent    | Networking       | 作業系統獨立非同步網路連結

可選擇的相依套件:

 函式庫      | 目的             | 描述
 ------------|------------------|----------------------
 miniupnpc   | UPnP Support     | 跳防火牆輔助
 libdb4.8    | Berkeley DB      | 錢包儲存 (只有錢包啟動時需要)
 qt          | GUI              | 圖形用戶介面工具包 (只有圖形用戶介面啟動時需要)
 protobuf    | Payments in GUI  | 支付協定用資料交換格式 (只有圖形用戶介面啟動時需要)
 libqrencode | QR codes in GUI  | 可選擇的產生 QR codes (只有圖形用戶介面啟動時需要)
 univalue    | Utility          | JSON parsing 和編碼 (除非 --with-system-univalue 完成配置，否則會使用附帶版本)
 libzmq3     | ZMQ notification | 可選擇的，允許產生 ZMQ 通知 (需要 4.x 以上的 ZMQ 版本)

關於發布出的使用版本，可以看 [release-process.md](release-process.md) 中的 *Fetch and build inputs*.

Memory Requirements
--------------------

C++ 編譯器非常需要記憶體，當編譯 Bitcoin Core 時，建議至少要有 1.5 GB 的記憶體可以使用，當系統低於此現時， gcc 能夠調整至附加的 CXXFLAGS 來保存記憶體:


    ./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768"

Dependency Build Instructions: Ubuntu & Debian
----------------------------------------------
建立需求:

    sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils

安裝時需要的 Boost 函式庫檔案的選項:

1. 在至少 Ubuntu 14.04+ 和 Debian 7+ 版本，它們是個別 boost 開發包的屬名，所以下列的可以使用來只安裝 boost 中必要的部分:

        sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev

2. 如果那不能運作，可以全部安裝:

        sudo apt-get install libboost-all-dev

錢包需要 BerkeleyDB ， db4.8 包可以使用 [here](https://launchpad.net/~bitcoin/+archive/bitcoin)。 你可以使用以下的指令來新增存放處和安裝:

    sudo add-apt-repository ppa:bitcoin/bitcoin
    sudo apt-get update
    sudo apt-get install libdb4.8-dev libdb4.8++-dev

Ubuntu 和 Debian 有他們自己的 libdb-dev 和 libdb++-dev 包，但這要安裝 BerkeleyDB 5.1 或更之後的版本，那會因為基於 BerkeleyDB 4.8 上分散式執行，而破壞二元錢包的一致性，如果你不在乎錢包的一致性，輸入 `--with-incompatible-bdb` 來配置。

看 "Disable-wallet mode" 的部分來建立不用錢包的 Bitcoin Core。

可選擇的:

    sudo apt-get install libminiupnpc-dev (see --with-miniupnpc and --enable-upnp-default)

ZMQ 相依套件:

    sudo apt-get install libzmq3-dev (provides ZMQ API 4.x)

Dependencies for the GUI: Ubuntu & Debian
-----------------------------------------

如果你想要建立 Bitcoin-Qt, 確認這些 Qt 開發所需要的包是否安裝了， Qt 5 和 Qt 4 都可以用來建立圖形用戶介面。如果你兩個都裝了，它會使用 Qt 5，輸入 `--with-gui=qt4` 來配置選擇 Qt 4，建立不用圖形用戶介面的則輸入 `--without-gui`。

用 Qt 5 (建議的) 來建立需要以下:

    sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler

或者，用 Qt 4 來建立需要以下:

    sudo apt-get install libqt4-dev libprotobuf-dev protobuf-compiler

libqrencode (可選擇的) 可以用以下來安裝:

    sudo apt-get install libqrencode-dev

安裝後，他們可以被配置找到，然後一個 bitcoin-qt 執行檔會預設建立。

Dependency Build Instructions: Fedora
-------------------------------------
Build requirements:

    sudo dnf install gcc-c++ libtool make autoconf automake openssl-devel libevent-devel boost-devel libdb4-devel libdb4-cxx-devel

Optional:

    sudo dnf install miniupnpc-devel

To build with Qt 5 (recommended) you need the following:

    sudo dnf install qt5-qttools-devel qt5-qtbase-devel protobuf-devel

libqrencode (optional) can be installed with:

    sudo dnf install qrencode-devel

Notes
-----
The release is built with GCC and then "strip bitcoind" to strip the debug
symbols, which reduces the executable size by about 90%.


miniupnpc
---------

[miniupnpc](http://miniupnp.free.fr/) may be used for UPnP port mapping.  It can be downloaded from [here](
http://miniupnp.tuxfamily.org/files/).  UPnP support is compiled in and
turned off by default.  See the configure options for upnp behavior desired:

	--without-miniupnpc      No UPnP support miniupnp not required
	--disable-upnp-default   (the default) UPnP support turned off by default at runtime
	--enable-upnp-default    UPnP support turned on by default at runtime


Berkeley DB
-----------
It is recommended to use Berkeley DB 4.8. If you have to build it yourself:

```bash
BITCOIN_ROOT=$(pwd)

# Pick some path to install BDB to, here we create a directory within the bitcoin directory
BDB_PREFIX="${BITCOIN_ROOT}/db4"
mkdir -p $BDB_PREFIX

# Fetch the source and verify that it is not tampered with
wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
echo '12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef  db-4.8.30.NC.tar.gz' | sha256sum -c
# -> db-4.8.30.NC.tar.gz: OK
tar -xzvf db-4.8.30.NC.tar.gz

# Build the library and install to our prefix
cd db-4.8.30.NC/build_unix/
#  Note: Do a static build so that it can be embedded into the executable, instead of having to find a .so at runtime
../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX
make install

# Configure Bitcoin Core to use our own-built instance of BDB
cd $BITCOIN_ROOT
./autogen.sh
./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/" # (other args...)
```

**Note**: You only need Berkeley DB if the wallet is enabled (see the section *Disable-Wallet mode* below).

Boost
-----
If you need to build Boost yourself:

	sudo su
	./bootstrap.sh
	./bjam install


Security
--------
To help make your bitcoin installation more secure by making certain attacks impossible to
exploit even if a vulnerability is found, binaries are hardened by default.
This can be disabled with:

Hardening Flags:

	./configure --enable-hardening
	./configure --disable-hardening


Hardening enables the following features:

* Position Independent Executable
    Build position independent code to take advantage of Address Space Layout Randomization
    offered by some kernels. Attackers who can cause execution of code at an arbitrary memory
    location are thwarted if they don't know where anything useful is located.
    The stack and heap are randomly located by default but this allows the code section to be
    randomly located as well.

    On an AMD64 processor where a library was not compiled with -fPIC, this will cause an error
    such as: "relocation R_X86_64_32 against `......' can not be used when making a shared object;"

    To test that you have built PIE executable, install scanelf, part of paxutils, and use:

    	scanelf -e ./bitcoin

    The output should contain:

     TYPE
    ET_DYN

* Non-executable Stack
    If the stack is executable then trivial stack based buffer overflow exploits are possible if
    vulnerable buffers are found. By default, bitcoin should be built with a non-executable stack
    but if one of the libraries it uses asks for an executable stack or someone makes a mistake
    and uses a compiler extension which requires an executable stack, it will silently build an
    executable without the non-executable stack protection.

    To verify that the stack is non-executable after compiling use:
    `scanelf -e ./bitcoin`

    the output should contain:
	STK/REL/PTL
	RW- R-- RW-

    The STK RW- means that the stack is readable and writeable but not executable.

Disable-wallet mode
--------------------
When the intention is to run only a P2P node without a wallet, bitcoin may be compiled in
disable-wallet mode with:

    ./configure --disable-wallet

In this case there is no dependency on Berkeley DB 4.8.

Mining is also possible in disable-wallet mode, but only using the `getblocktemplate` RPC
call not `getwork`.

Additional Configure Flags
--------------------------
A list of additional configure flags can be displayed with:

    ./configure --help


Setup and Build Example: Arch Linux
-----------------------------------
This example lists the steps necessary to setup and build a command line only, non-wallet distribution of the latest changes on Arch Linux:

    pacman -S git base-devel boost libevent python
    git clone https://github.com/bitcoin/bitcoin.git
    cd bitcoin/
    ./autogen.sh
    ./configure --disable-wallet --without-gui --without-miniupnpc
    make check

Note:
Enabling wallet support requires either compiling against a Berkeley DB newer than 4.8 (package `db`) using `--with-incompatible-bdb`,
or building and depending on a local version of Berkeley DB 4.8. The readily available Arch Linux packages are currently built using
`--with-incompatible-bdb` according to the [PKGBUILD](https://projects.archlinux.org/svntogit/community.git/tree/bitcoin/trunk/PKGBUILD).
As mentioned above, when maintaining portability of the wallet between the standard Bitcoin Core distributions and independently built
node software is desired, Berkeley DB 4.8 must be used.


ARM Cross-compilation
-------------------
These steps can be performed on, for example, an Ubuntu VM. The depends system
will also work on other Linux distributions, however the commands for
installing the toolchain will be different.

First install the toolchain:

    sudo apt-get install g++-arm-linux-gnueabihf

To build executables for ARM:

    cd depends
    make HOST=arm-linux-gnueabihf NO_QT=1
    cd ..
    ./configure --prefix=$PWD/depends/arm-linux-gnueabihf --enable-glibc-back-compat --enable-reduce-exports LDFLAGS=-static-libstdc++
    make


For further documentation on the depends system see [README.md](../depends/README.md) in the depends directory.
