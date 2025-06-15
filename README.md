# HOW-TO-install-MFCUK-rev-65
A simple but exhaustive guide to install MFCUK 0.3.3 and LIBNFC 1.5.1 in 2025

You first need to download both libnfc 1.5.1 and mfcuk r65. The former is available among github releases wile the latter can be obtained via git rebase.

If you're on Arch you'll need `pcsclite`, if on Debian derivates `libpcsclite-dev`

For libnfc

    mkdir -p ~/builds/nfc
    cd ~/builds/nfc
    wget https://github.com/nfc-tools/libnfc/releases/download/libnfc-1.5.1/libnfc-1.5.1.tar.gz
    tar zxf libnfc-1.5.1.tar.gz
    cd libnfc-1.5.1
    ./configure --prefix=/$HOME/builds/nfc/prefix --with-drivers=all --sysconfdir=/etc/nfc --enable-serial-autoprobe
    make
    make install
    cd ..
./configure options are needed to enable editing options via the usual /etc/nfc/libnfc.conf file and to allow using a PN532 shield with a USB-UART interface.

Then, for mfcuk

    git clone https://github.com/nfc-tools/mfcuk mfcuk-r65
    cd mfcuk-r65
    git reset --hard a895841
    autoreconf -is
    LIBNFC_CFLAGS=-I/$HOME/builds/nfc/prefix/include LIBNFC_LIBS="-L/$HOME/builds/nfc/prefix/lib -lnfc" ./configure --prefix=/$HOME/builds/nfc/prefix
    make

*!WARNING!*  
If gives error "undefined reference to nfc_*" during make procedure, you simply need to:

    cd ~/builds/nfc/mfcuk-r65
    nano /$HOME/builds/nfc/mfcuk-r65/src/Makefile
    //NOW EDIT THE LINE 'LIBS=' adding $(LIBNFC_LIBS)
    -LIBS =
    +LIBS = $(LIBNFC_LIBS)
    CTRL+S
    CTRL+X
    make clean && make

To resolve shared libraries LIBNFC bug

    sudo sh -c "echo /%HOME/builds/nfc/prefix/lib > /etc/ld.so.conf.d/usr-local-lib.conf"
    sudo ldconfig
   
If needed, uncomment last line in /etc/nfc/libnfc.conf to enable PN532 discovery.
You'll now need to run these last 2 commands

    cd /$HOME/builds/nfc/mfcuk-r65/src  
    LD_LIBRARY_PATH=/$HOME/builds/nfc/prefix/lib ./mfcuk -C -R 0:A -v 3 -s 250 -S 250 -O mifare_ext.dmp
