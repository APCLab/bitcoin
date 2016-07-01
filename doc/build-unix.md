UNIX BUILD NOTES
====================
這是一些關於如何在 Unix 上建立 Bitcoin Core 的筆記。

(for OpenBSD specific instructions, see [build-openbsd.md](build-openbsd.md))

Note
---------------------
總是用絕對路徑來配置和編譯比特幣和dependencies，舉例來說，當決定相依套件的路徑:

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

如果dependencies正確的話，這也會建立 bitcoin-qt。

Dependencies
---------------------

需要的dependencies:

 函式庫      | 目的             | 描述
 ------------|------------------|----------------------
 libssl      | Crypto           | Random Number Generation, 橢圓曲線密碼學
 libboost    | Utility          | 函式庫包含執行緒、資料結構...等
 libevent    | Networking       | 作業系統獨立非同步網路連結

可選擇的dependencies:

 函式庫      | 目的             | 描述
 ------------|------------------|----------------------
 miniupnpc   | UPnP Support     | 翻防火牆
 libdb4.8    | Berkeley DB      | 錢包儲存 (只有Wallet啟動時需要)
 qt          | GUI              | GUI工具包 (只有GUI啟動時需要)
 protobuf    | Payments in GUI  | 支付協定用資料交換格式 (只有GUI啟動時需要)
 libqrencode | QR codes in GUI  | 可選擇的產生 QR codes (只GUI啟動時需要)
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

Ubuntu 和 Debian 有他們自己的 libdb-dev 和 libdb++-dev 包，但這要安裝 BerkeleyDB 5.1 或更之後的版本，那會因為基於 BerkeleyDB 4.8 上分散式執行，而破壞二進制錢包的一致性，如果你不在乎錢包的一致性，輸入 `--with-incompatible-bdb` 來配置。

看 "Disable-wallet mode" 的部分來建立不用錢包的 Bitcoin Core。

可選擇的:

    sudo apt-get install libminiupnpc-dev (see --with-miniupnpc and --enable-upnp-default)

ZMQ dependencies:

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

安裝後，他們可以被配置找到，然後一個 bitcoin-qt 執行文件會預設建立。

Dependency Build Instructions: Fedora
-------------------------------------
建立需求:

    sudo dnf install gcc-c++ libtool make autoconf automake openssl-devel libevent-devel boost-devel libdb4-devel libdb4-cxx-devel

可選擇的:

    sudo dnf install miniupnpc-devel

用 Qt 5 (建議的) 來建立需要以下:

    sudo dnf install qt5-qttools-devel qt5-qtbase-devel protobuf-devel

libqrencode (可選擇的) 可以用以下來安裝:

    sudo dnf install qrencode-devel

Notes
-----
這個發布是用 GCC 來建立的，然後 "strip bitcoind" 來去除除錯符號，減少了大約 90% 的執行文件大小。


miniupnpc
---------

[miniupnpc](http://miniupnp.free.fr/) 可以用來 UPnP port mapping，可以從 [here](
http://miniupnp.tuxfamily.org/files/) 下載，UPnP support 會被編譯進去然後預設會是關閉的，對 UPnP 行為有需求的可以看這個配置選項:

	--without-miniupnpc      沒有 UPnP support ， miniupnp 不需要
	--disable-upnp-default   (預設) UPnP support 在運行時預設關閉
	--enable-upnp-default    UPnP support 在運行時預設開啟


Berkeley DB
-----------
如果你需自行建立，建議使用 Berkeley DB 4.8 :

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

**Note**: 你只有在錢包啟動時才需要使用 Berkeley DB (看於下的 *Disable-Wallet mode* 部分).

Boost
-----
如果你需要自行建立 Boost :

	sudo su
	./bootstrap.sh
	./bjam install


Security
--------
儘管一個弱點被發現，以將一定的攻擊化為不可能的開方方式，使編譯的過後的程式更堅固，來幫助 bitcoin 安裝能更安全。stallation more secure 
這可以讓它無效:

Hardening Flags:

	./configure --enable-hardening
	./configure --disable-hardening


Hardening 有下列特色:

* Position Independent Executable 地址無關可執行文件
    建立地址無關代碼可以讓一些 kernels 提供位址空間配置隨機載入的優點，可以攻擊任意記憶體位址的攻擊者會因為他們不知道哪裡放的資料是有用的而失敗， 堆疊和堆積 可以隨機配置，這也讓 code section 也能隨機配置。

    在一個沒有用 -fPIC 來編譯函式庫的 AMD64 處理器，這會導致一個錯誤像是: "relocation R_X86_64_32 against `......' can not be used when making a shared object;"

    測是當已經建立 PIE 執行文件, install scanelf, part of paxutils, and use:

    	scanelf -e ./bitcoin

    輸出應該包含:

     TYPE
    ET_DYN

* Non-executable Stack 不可執行之堆疊區段
    如果推疊是可執行的，弱點緩衝器被發現的話，則基於瑣細堆疊的緩衝器是有可能會 overflow ， 預設中， bitcoin 應該以不可執行之堆疊區段建立，但是如果其中一個它使用的函式庫請求一個可執行的堆疊或因製造錯誤而用編譯器擴充而需要一個可執行的堆疊，它便會默默地建立一個沒受到不可執行之堆疊區段保護的可執行堆疊。

    驗證堆疊是不可執行的，輸入:
    `scanelf -e ./bitcoin`

    輸出應包含:
	STK/REL/PTL
	RW- R-- RW-

    STK RW- 代表這個堆疊是可讀和可的但不可執行。

Disable-wallet mode
--------------------
如果目的只是運行 P2P 節點而不用錢包， bitcoin 可以在無錢包模式下建立:

    ./configure --disable-wallet

在這個案例裡，不用 Berkeley DB 4.8 的相依套件。

在無錢包模式下仍然可以挖礦，但只能使用 `getblocktemplate` RPC call 而非 `getwork`。

Additional Configure Flags
--------------------------
可以顯示附加配置的 flags 的清單:

    ./configure --help


Setup and Build Example: Arch Linux
-----------------------------------
這個例子列出在上次更動的 Arch Linux 上設置和建立一個指令列且無錢包分布的必要步驟:

    pacman -S git base-devel boost libevent python
    git clone https://github.com/bitcoin/bitcoin.git
    cd bitcoin/
    ./autogen.sh
    ./configure --disable-wallet --without-gui --without-miniupnpc
    make check

Note:
有錢包支援的編譯需要用 Berkeley DB 4.8 (package `db`) 之後的版本，輸入 `--with-incompatible-bdb`，或建立和相依在一個本地版的 Berkeley DB 4.8. 這個立即可用的 Arch Linux 包可以輸入`--with-incompatible-bdb` 來建立，這是根據 [PKGBUILD](https://projects.archlinux.org/svntogit/community.git/tree/bitcoin/trunk/PKGBUILD)。
如前所述，當需要維持錢包在 standard Bitcoin Core distributions 和 independently built
node software 的可攜愛性，必須使用 Berkeley DB 4.8 。 


ARM Cross-compilation
-------------------
這個步驟能夠幫助它在像是 Ubuntu VM 上運行，相依系統也能在其他 Linux distributions 運作，然而安裝 toolchain 會不同。

首先，安裝 toolchain:

    sudo apt-get install g++-arm-linux-gnueabihf

建立 ARM 執行文件:

    cd depends
    make HOST=arm-linux-gnueabihf NO_QT=1
    cd ..
    ./configure --prefix=$PWD/depends/arm-linux-gnueabihf --enable-glibc-back-compat --enable-reduce-exports LDFLAGS=-static-libstdc++
    make


有夠相依系統更進一步的資訊，可看相依目錄 [README.md](../depends/README.md) 。
