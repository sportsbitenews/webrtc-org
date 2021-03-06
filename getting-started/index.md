---
layout: default
title: Getting Started
permalink: /reference/getting-started/
---


{% include toc-hide.html %}


### Before you start

First, be sure to install the [prerequisite software][1].

[1]: prerequisite-sw/

The currently supported platforms are Windows, Mac OS X, Linux, Android and iOS. See [this page][2] for iOS instructions.

[2]: ios/


### Getting the code

Create a working directory, enter it, and run:

~~~~~
$ gclient config --name src http://webrtc.googlecode.com/svn/trunk
~~~~~


#### Linux & Android specific steps. *

Select build system (optional for all OSs except Android where ninja is
mandatory). *

~~~~~
$ gclient sync --force
~~~~~

Starred (*) items are described in their own section below and should be
performed, if at all, in place.


#### Notes

  * If you're a committer, substitute `https` for `http`.

  * On Windows, use `gclient.bat` instead (or prefix the commands by invoking
    python).

  * Android requires that you build on a Linux machine.

The `gclient sync` command fetches dependencies and generates native build files for your environment using [gyp][3] (Windows: ninja/Visual Studio, OS X: ninja/XCode, Linux: ninja/make, Android: ninja). [Ninja][4] is the default build system for all platforms. It is possible to just generate new build files by calling:

~~~~~
$ gclient runhooks --force
~~~~~

[3]: http://code.google.com/p/gyp/
[4]: https://code.google.com/p/chromium/wiki/NinjaBuild


#### Linux & Android Specific Steps

If building for Linux or Android these steps should be inlined above.  

For Linux or Android:

~~~~~
$ export JAVA_HOME=<location of OpenJDK 7>
~~~~~

For Android:

~~~~~
$ echo "target_os = ['android', 'unix']" >> .gclient
$ gclient sync
$ cd src
$ source ./build/android/envsetup.sh
$ export GYP_DEFINES="$GYP_DEFINES OS=android"
~~~~~

#### Select build system

You can directly specify which build system to use. This can be done if you
don't want to use ninja. Set the `GYP_GENERATORS` environment variable to the
string:

  * `make` for Makefiles
  * `msvs` for Visual Studio
  * `msvs-ninja` for Visual Studio project building with ninja
  * `xcode` for Xcode

Note, when the build environment is set to generate Visual Studio project
files, gyp will by default, generate a project for the latest version of
Visual Studio installed on your computer. It is possible to specify the
desired Visual Studio version as described below:

Set environment variable `GYP_MSVS_VERSION=<version>` before runhooks or
manually run the following gyp command from the src/ directory (this replaces
`gclient runhooks`):

~~~~~
$ webrtc/build/gyp_webrtc -G msvs_version=<version>
~~~~~

`<version>` is on the form `YYYY`.


#### Using git instead of svn

While the authoritative repo uses subversion, it's possible to mostly ignore that during development and use git instead.  A basic recipe for setting up git-svn in a new checkout on linux is:

~~~~~
gclient config --name src 'git+https://chromium.googlesource.com/external/webrtc' && \
gclient sync -j200 && \
cd src && \
git svn init --prefix=origin/ https://webrtc.googlecode.com/svn -Ttrunk --rewrite-root=http://webrtc.googlecode.com/svn && \
git config svn-remote.svn.fetch trunk:refs/remotes/origin/master && \
git svn fetch && \
git checkout master && \
git branch -vv && \
echo Yay
~~~~~


### Building

Binaries are by default (i.e. when building with ninja) generated in
out/Debug/ and out/Release/ for debug and release builds respectively.


#### With ninja

~~~~~
$ cd src
~~~~~

Debug:

~~~~~
$ ninja -C out/Debug
~~~~~

Release:

~~~~~
$ ninja -C out/Release
~~~~~


#### With Visual Studio

Use Visual Studio to open and build the src/all.sln solution file.


### Contributing patches

Please see [Contributing bug fixes][5] for information on how to get your
changes included in the webrtc codebase.

[5]: /reference/contributing/


### Example Applications

WebRTC contains several example applications which can be found under
src/webrtc/examples and src/talk/examples. Higher level applications are
listed first.


### AppRTCDemo (Android application using WebRTC Native APIs via JNI)

Target name `AppRTCDemo`. The JNI wrapper is documented [here][6]. `AppRTCDemo` is documented [here][7].

[6]: https://code.google.com/p/webrtc/source/browse/trunk/talk/app/webrtc/java/README
[7]: https://code.google.com/p/webrtc/source/browse/trunk/talk/examples/android/README


### Peerconnection (Application using WebRTC Native APIs)

Peerconnection consist of two applications. A server application, with target
name peerconnection_server, and a client application, with target name
`peerconnection_client`. Note that we currently don't support
`peerconnection_client` for Mac and Android.

The client application has simple voice and video capabilities. The server
enables client applications to initiate a call between clients by managing
signaling messages generated by the clients.


#### Setting up P2P calls between peerconnection_clients

Start `peerconnection_server`. You should see the following message indicating
that it is running:

~~~~~
Server listening on port 8888
~~~~~

Start any number of `peerconnection_clients` and connect them to the server.
The client UI consists of a few parts:

**Connecting to a server:** When the application is started you must specify
which machine (IP-address) the server application is running on. Once that is
done you can press **Connect** or the return button.

**Select a peer:** Once successfully connected to a server you can connect to
a peer by double clicking or select+press return on a peer's name.

**Video chat:** When a peer has been successfully connected to, a Video chat
will be displayed in full window.

**Ending chat session:** Press **Esc**. You will now be back to selecting a
peer.

**Ending connection:** Press **Esc** and you will now be able to select which
server to connect to.


#### Testing `peerconnection_server`

Start an instance of `peerconnection_server` application.

Open `src/talk/examples/peerconnection/server/server_test.html` in your
browser. Click **Connect**. Observe that the `peerconnection_server` announces
your connection. Open one more tab using the same page. Connect it too (with a
different name). It is now possible to exchange messages between the connected
peers.


### Call (Application that establishes a call using libjingle)

Target name call (currently disabled). Call uses xmpp (as opposed to SDP used
by WebRTC) to allow you to login using your Gmail account and make audio/video
calls with your gmail friends. It is built on top of `libjingle` to provide
this functionality.

Further, you can specify input and output RTP dump for voice and video. It
provides two samples of input RTP dump: `voice.rtpdump` which contains a
stream of single channel, 16Khz voice encoded with G722, and `video.rtpdump`
which contains a 320x240 video encoded with H264 AVC at 30 frames per second.
The provided samples will inter-operate with Google Talk Video. If you use
other input RTP dump, you may need to change the codecs in `call_main.cc`
(lines 215 - 222).


### WebRTCDemo (Android application using media engines)

Target name `WebRTCDemo`. This app does not use WebRTC native APIs. It can
send and receive media streams if manually connected to another `WebRTCDemo`
that is directly accessible (e.g. firewalls might prevent you from
establishing a connection). Further it allows setting, enabling and disabling
audio and video processing functionality (e.g. echo cancellation, NACK, audio
codec and video codec).


### Relay server (specialized server application that can be used with Call)

Target name `relayserver`. Relays traffic when a direct peer-to-peer
connection can't be established.


### STUN server

Target name `stunserver`. Implements the STUN protocol for Session Traversal
Utilities for NAT as documented in [RFC5389][8].

[8]: http://tools.ietf.org/html/rfc5389


### TURN server

Target name `turnserver`. In active development to reach compatibility with
[RFC5766][8].
