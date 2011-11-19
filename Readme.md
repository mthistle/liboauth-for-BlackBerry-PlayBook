This is a test project for the BlackBerry Tablet OS development environment QDE which demonstrates the use of liboauth (for C) ported to PlayBook.  

Here is the original post from: <a href="http://droolfactory.blogspot.com/2011/11/configuring-liboauth-c-library-for.html">http://droolfactory.blogspot.com/2011/11/configuring-liboauth-c-library-for.html</a>, reposted here for easy access.

Hope it helps you get started with oath in the Native development environment for PlayBook.

<h1>Overview</h1>

You've decided to write a PlayBook App with the NDK (Native Development Kit).  That's awesome, let's learn how to port an OSS library you want to use in your new app.

For those of you who are ./configure experts, please point out anything I have wrong or maybe could do differently which might make this easier.

Ok so first off, I am not a ./configure expert but I am perfectly willing to beat my head against the compiling and linking wall until I get something that works.

Some prerequisites:
<ul>
<li>I am on a Mac, switching between a Snow Leopard and a Lion box.  So this should work for either.  If you are on Windows, please feel free to fork my post and write up how you did this on Windows.  If you do, then please refer back here for the Mac Devs who find your site.
<li>I expect that you have installed the BlackBerry Tablet OS NDK in the default locations.
<li>I am using 2 PlayBooks for testing.  One running 1.0.7 and one running 2.0.  You can expect this to work for either, if not let me know and I will verify that it is working no the latest OS.
<li> You know what the Terminal.app is and aren't afraid to use it.
</ul>

Let's get started.

<h2>Porting liboauth (<a href="http://liboauth.sourceforge.net/">http://liboauth.sourceforge.net/</a>)</h2>

I choose liboauth since it involved the least amount of work for the library's I have been working with.  I built liboath-0.9.4 for PlayBook.  I only built the device (arm) version since I am not using the simulator; I like to test on the metal; but with some minor variations to the options it should not be hard to get a simulator compatible version built.

<h3>First: Set the environment for building</h3>

To do this source the NDK included environment script:
<blockquote><pre>source /Developer/SDKs/bbndk-1.0/bbndk-env.sh</pre>
</blockquote>
This should setup your environment for the QNX compiler so we can build for the PlayBook.  Here is what that looks like (at time of writing):
<blockquote><pre>QNX_TARGET="/Developer/SDKs/bbndk-1.0/target/qnx6"
QNX_HOST="/Developer/SDKs/bbndk-1.0/host/macosx/x86"
QNX_CONFIGURATION="/Users/YOUR_USERNAME_HERE/Applications/Library/Research In Motion/BlackBerry Native SDK"
MAKEFLAGS="-I$QNX_TARGET/usr/include"
DYLD_LIBRARY_PATH="/Developer/SDKs/bbndk-1.0/host/macosx/x86/usr/lib:$DYLD_LIBRARY_PATH"
PATH="$QNX_HOST/usr/bin:$QNX_HOST/usr/qde/eclipse/jre/bin:$PATH"
export QNX_TARGET QNX_HOST QNX_CONFIGURATION MAKEFLAGS DYLD_LIBRARY_PATH PATH</pre></blockquote>

<h3>Second: Modify the config.sub to include the CPU instruction set for ARMv7</h3>

When we set the target and host in the ./configure command we will be using the PlayBook CPU type of armv7, we will need to add that to the config.sub since it is missing from liboauth's config.sub.  I did this by searching for armv, where you will find and switch out the line shown in the following diff, notice I just added 7 into the armv[2345] statement:

<blockquote><pre>diff --git a/config.sub b/config.sub
index c2d1257..dd4ef08 100755
--- a/config.sub
+++ b/config.sub
@@ -249,7 +249,7 @@ case $basic_machine in
        | alpha | alphaev[4-8] | alphaev56 | alphaev6[78] | alphapca5[67] \
        | alpha64 | alpha64ev[4-8] | alpha64ev56 | alpha64ev6[78] | alpha64pca5[67] \
        | am33_2.0 \
-       | arc | arm | arm[bl]e | arme[lb] | armv[2345] | armv[345][lb] | avr | avr32 \
+       | arc | arm | arm[bl]e | arme[lb] | armv[23457] | armv[345][lb] | avr | avr32 \
        | bfin \
        | c4x | clipper \
        | d10v | d30v | dlx | dsp16xx \</pre></blockquote>

<h5>Sidetrack: If you want to look at some more details on the CPU, check out <a href="http://en.wikipedia.org/wiki/Texas_Instruments_OMAP">http://en.wikipedia.org/wiki/Texas_Instruments_OMAP</a></h5>

<h3>Third: Run ./configure with some options</h3>

Here is the ./configure I used.  I included the OpenSSL and the Curl libraries.

<blockquote><b>./configure CXX="QCC -V4.4.2,gcc_ntoarmv7le_cpp" CC="qcc -V4.4.2,gcc_ntoarmv7le" --prefix=$PWD/playbook NM=ntoarmv7-nm RANLIB=ntoarmv7-ranlib STRIP=ntoarmv7-strip AR=ntoarmv7-ar LD=ntoarmv7-ld LDFLAGS="-L/Developer/SDKs/bbndk-1.0/target/qnx6/armle-v7/usr/lib/" CURL_CFLAGS="-DFD_SETSIZE=64 -g -O2" CURL_LIBS="-Wl,-s -Wl,-Bsymbolic -Wl,--gc-sections" --build=i386-apple-darwin11.2.0  INCLUDES="/Developer/SDKs/bbndk-1.0/target/qnx6/armle-v7/usr/include" --target=armv7-nto --host=armv7-nto</b></blockquote>

If the above worked then great, you can now run make, else, you will need to try to debug the problem with your environment/configuration.

<h3>Fourth: Run make</h3>

Run make and then make install to install into the path we chose in our configure statement which was set with the --prefix=$PWD/playbook.  

What is $PWD, it is the environment variable for the directory you are building the library within.  

Basically, the make install will make a new directory called playbook and place all the built artifacts into that directory.

<h3>Lastly: Pull liboauth into your project</h3>

I'll leave that for the user to work out but here is a sample <a href="https://github.com/mthistle/liboauth-for-BlackBerry-PlayBook">liboauth on BlackBerry PlayBook qde project</a> on github that has the built library and the test/selftest_wiki.c file renamed to main.c included to verify that the library is working.

<h2>Finishing</h2>

With a little tweaking of the ./configure, porting an OSS project to the PlayBook is not to hard.  When you run into compiler issues the story can be a little different (read, lots of hair pulling) but is not insurmountable.

Good luck with your app!
