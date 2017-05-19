Creating a multilib toolchain for Slackware, from scratch
=========================================================

When you want to build a native multilib toolchain for Slackware64 (glibc,
gcc and binutils), there are some problems to overcome.

* binutils
  This is an easy one: just add "--enable-multilib" to the configure command
  when building the package. In Slackware64, this has already been done
  for you, so no further action is required.

* glibc
  This requires a multilib gcc compiler. Also, the SlackBuild needs
  several edits. The glibc.SlackBuild in Slackware64 will only build the
  64bit stuff, so we need some explicit additions to compile and package
  the 32bit components as well. Fred Emmott of Slamd64 has done the heavy
  lifting to make this happen. However, while Slamd64 has split glibc into
  a 64bit and a 'compat32' package, I decided to build one big package with
  the full multilib support included, just like the binutils already have.

* gcc
  This one is the hardest to crack. The two big issues are (1) you need a
  multilib-capable gcc compiler to build a multilib gcc compiler suite and
  (2) you need gnat to build gcc-gnat (the ADA compiler). You can of course
  "borrow" a set of multilib-capable compiler packages (from Slamd64 for
  instance) but it is cooler to work only with Slackware64.


Detailed compilation steps
==========================

* Build a barebones, statically compiled gcc only. This is possible
using the default gcc of Slackware64. I changed the gcc build script to a
"gcc-static.SlackBuild", this is the script you should run.

* Run "upgradepkg" to upgrade 'gcc' to the new static gcc compiler package
(keep the remaining gcc-* packages installed!).

* Run ". /etc/profile" to reset your environment. I found out that it
never hurts to do this from time to time because of all the libraries that
are changing.

* Build a "bootstrap" version of a multilib glibc. Since only a part of
the gcc compiler suite is available, some patches need to made to glibc so
that it compiles successfully. We need this so that we can build a shared
multilib-capable gcc later. I modified the glibc build script and called
that "glibc-multilib.SlackBuild". The "bootstrap" packagebuild should be
run as "./glibc-multilib.SlackBuild --bootstrap".

* Upgrade your glibc-* packages to the new multilib versions of the packages.

* Use this temporary gcc/glibc bootstrap toolchain to build a shared,
multilib set of packages for the "c,c++,ada" languages. The pre-existing
gnat compiler will be used by the static gcc to recompile gnat for
multilib. I re-wrote the gcc.SlackBuild a bit so that I can specify
more easily what languages I want to build. The updated script is
called gcc-multilib.Slackuild. Run it as follows: 
"LANGS='c,c++,ada' GNATEXTERNAL="YES" ./gcc-multilib.SlackBuild"

* Remove _all_ gcc-* packages and install the new set of 'gcc' 'gcc-g++'
and 'gcc-gnat' packages

* Rebuild the glibc packages, this time without the "--bootstrap" parameter
to "./glibc-multilib.SlackBuild". This is our final set of multilib-capable
glibc packages.

* Build the complete set of multilib gcc compiler packages:
"./gcc-multilib.SlackBuild" and run "upgradepkg --reinstall --install-new
gcc-*.txz" on the new packages.

Done!


NOTE
====

When building a multilib gcc/glibc from scratch on Slackware64, and the
installed compiler is still built with '--disable-multilib' you need to edit:

  /usr/lib64/gcc/x86_64-slackware-linux/4.3.3/specs

and change the lines:

*multilib:

  . !m64 !m32;.:../lib64 m64 !m32;32:../lib !m64 m32;

to:

*multilib:

  . !m64 !m32;.:../lib64 m64 !m32;../lib !m64 m32;

or else you will get errors like:

checking for suffix of object files... configure: error: cannot compute suffix of object files: cannot compile

=============================================================================

Wrote by Eric Hameleers <alien@slackware.com> 25-jun-2009

Modified by Widya Walesa <walecha_99_[at]_gmail_[dot]_com> 09-May-2017
