Mac OS X Build Instructions and Notes
====================================
This guide will show you how to build digitalcoind(headless client) for OSX.

Notes
-----

* Tested on OS X 10.6 through 10.9 on 64-bit Intel processors only.
Older OSX releases or 32-bit processors are no longer supported.

* All of the commands should be executed in a Terminal application. The
built-in one is located in `/Applications/Utilities`.

Preparation
-----------

You need to install XCode with all the options checked so that the compiler
and everything is available in /usr not just /Developer. XCode should be
available on your OS X installation media, but if not, you can get the
current version from https://developer.apple.com/xcode/. If you install
Xcode 4.3 or later, you'll need to install its command line tools. This can
be done in `Xcode > Preferences > Downloads > Components` and generally must
be re-done or updated every time Xcode is updated.

There's an assumption that you already have `git` installed, as well. If
not, it's the path of least resistance to install [Github for Mac](https://mac.github.com/)
(OS X 10.7+) or
[Git for OS X](https://code.google.com/p/git-osx-installer/). It is also
available via Homebrew.

You will also need to install [Homebrew](http://brew.sh)
in order to install library dependencies.

The installation of the actual dependencies is covered in the Instructions
sections below.

Instructions: Homebrew
----------------------

#### Install dependencies using Homebrew

        brew install autoconf automake libtool boost miniupnpc openssl pkg-config protobuf qt

Note: After you have installed the dependencies, you should check that the Homebrew installed version of OpenSSL is the one available for compilation. You can check this by typing

        openssl version

into Terminal. You should see OpenSSL 1.0.1f 6 Jan 2014.

If not, you can ensure that the Homebrew OpenSSL is correctly linked by running

        brew link openssl --force

Rerunning "openssl version" should now return the correct version. If it
doesn't, make sure `/usr/local/bin` comes before `/usr/bin` in your
PATH. 

#### Installing berkeley-db4 using Homebrew

The homebrew package for berkeley-db4 has been broken for some time.  It will install without Java though.

Running this command takes you into brew's interactive mode, which allows you to configure, make, and install by hand:
```
$ brew install https://raw.github.com/mxcl/homebrew/master/Library/Formula/berkeley-db4.rb -–without-java 
```

These rest of these commands are run inside brew interactive mode:
```
/private/tmp/berkeley-db4-UGpd0O/db-4.8.30 $ cd ..
/private/tmp/berkeley-db4-UGpd0O $ db-4.8.30/dist/configure --prefix=/usr/local/Cellar/berkeley-db4/4.8.30 --mandir=/usr/local/Cellar/berkeley-db4/4.8.30/share/man --enable-cxx
/private/tmp/berkeley-db4-UGpd0O $ make
/private/tmp/berkeley-db4-UGpd0O $ make install
/private/tmp/berkeley-db4-UGpd0O $ exit
```

After exiting, you'll get a warning that the install is keg-only, which means it wasn't symlinked to `/usr/local`.  You don't need it to link it to build digitalcoin, but if you want to, here's how:

    $ brew --force link berkeley-db4


### Building `digitalcoind`

1. Clone the github tree to get the source code and go into the directory.

        git clone https://github.com/digitalcoin/digitalcoin.git
        cd digitalcoin

2.  Build digitalcoind:
        ./autogen.sh
        ./configure
        make

3.  It is a good idea to build and run the unit tests, too:

        make check

Creating a release build
------------------------
You can ignore this section if you are building `digitalcoind` for your own use.

digitalcoind/digitalcoin-cli binaries are not included in the Digitalcoin-Qt.app bundle.

If you are building `digitalcoind` or `Digitalcoin-Qt` for others, your build machine should be set up
as follows for maximum compatibility:

All dependencies should be compiled with these flags:

 -mmacosx-version-min=10.6
 -arch x86_64
 -isysroot $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.6.sdk

For MacPorts, that means editing your macports.conf and setting
`macosx_deployment_target` and `build_arch`:

    macosx_deployment_target=10.6
    build_arch=x86_64

... and then uninstalling and re-installing, or simply rebuilding, all ports.

As of December 2012, the `boost` port does not obey `macosx_deployment_target`.
Download `http://gavinandresen-digibyte.s3.amazonaws.com/boost_macports_fix.zip`
for a fix.

Once dependencies are compiled, see release-process.md for how the Digitalcoin-Qt.app
bundle is packaged and signed to create the .dmg disk image that is distributed.

Running
-------
It's now available at `./digitalcoind`, provided that you are still in the `src`
directory. We have to first create the RPC configuration file, though.

Run `./digitalcoind` to get the filename where it should be put, or just try these
commands:

    echo -e "rpcuser=digitalcoinrpc\nrpcpassword=$(xxd -l 16 -p /dev/urandom)" > "/Users/${USER}/Library/Application Support/Digitalcoin/digitalcoin.conf"
    chmod 600 "/Users/${USER}/Library/Application Support/Digitalcoin/digitalcoin.conf"

When next you run it, it will start downloading the blockchain, but it won't
output anything while it's doing this. This process may take several hours;
you can monitor its process by looking at the debug.log file, like this:

    tail -f $HOME/Library/Application\ Support/Digitalcoin/debug.log

Other commands:

    ./digitalcoind -daemon # to start the digitalcoin daemon.
    ./digitalcoin-cli --help  # for a list of command-line options.
    ./digitalcoin-cli help    # When the daemon is running, to get a list of RPC commands
