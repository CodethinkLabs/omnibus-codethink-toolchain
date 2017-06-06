# Codethink's Omnibus packaging for the GNU Toolchain

This is an example of building and packaging the GNU toolchain on multiple
platforms using Chef Software's [Omnibus](https://github.com/chef/omnibus/).

Our aim is to provide the C, C++ and Fortran compilers from GCC version 7 on
several platforms, including:

  * Red Hat Linux
  * Oracle Solaris
  * IBM AIX

This project is not directly based on the Chef Software
[omnibus-software](https://github.com/chef/omnibus-software)
and [omnibus-toolchain](https://github.com/chef/omnibus-toolchain)
repos. Although these projects share some goals, the focus is different
and the small size of the packaging instructions means it's easier to
duplicate things here than it is to maintain non-upstreamable forks of
the Chef Software projects.

All code is licensed under the
[Apache License version 2.0](https://www.apache.org/licenses/LICENSE-2.0).

## Instructions

This README doesn't cover installing the necessary dependencies. In brief
you need a working GNU toolchain, GNU Flex, Ruby, Bundler and Omnibus. Maybe
more.

On Red Hat style systems we also require the rpm-build package.

### Building on a dedicated build worker

By default Omnibus operates as a system build service, which is useful on
dedicated build machines. To run in this mode, just execute the following
command as root:

    omnibus build codethink-toolchain

### Test builds for developers

If you are a developer running this on your laptop, you hopefully balked
at the idea of running a random command as root. To test things locally,
you'll probably want to download everything into the current directory,
which you can do by adding this to the Omnibus command line:

    --override base_dir:./local

You'll also need to precreate the target install directory and give your
user access:

    sudo mkdir /opt/codethink-toolchain
    sudo chown $(whoami):$(whoami) /opt/codethink-toolchain

Builds will then store all intermediate files in the ./local directory, and
will not require root privileges to install.

### Optional stuff

You can change the number of parallel workers by adding `workers:N` to the
--override parameter. For example, to set a base dir of ./local and enforce
`make -j 2`, you would run:

    omnibus build codethink-toolchain --override base_dir:./local workers:2

## Platform notes

### x86_64-redhat-linux

We build 64-bit toolchain binaries on x86_64 hosts. The compilers default to
producing 64-bit binaries, but multilib is supported and so you can tell them
to output 32-bit binaries by passing `-m32`. This is a little-endian platform.

If you need 32-bit toolchain binaries, the simplest solution is to build the
package on an x86_32 host instead. Multilib should work there too.

### sparc-sun-solaris2.11

We build 32-bit toolchain binaries and the compilers default to producing
32-bit binaries. You can pass `-m64` to produce 64-bit binaries, and multilib
is supported so this should work for all cases. This is a big-endian platform.

Some (e.g. [Gentoo](https://wiki.gentoo.org/wiki/Sparc/Multilib)) say 32-bit is
still preferred on Solaris except for processes that need more than 4GB data or
where performance will be better. Performance might be better with 64-bit
toolchain binaries, but I haven't tested that.

GCC has its own notes about the
[Solaris operating system](https://gcc.gnu.org/install/specific.html#x-x-solaris2)
and the [SPARC processor family](https://gcc.gnu.org/install/specific.html#sparc-x-x).

### powerpc-ibm-aix7.2

We build 32-bit toolchain binaries. The compiler doesn't yet support multilib,
there seem to be issues with this on AIX although the [GCC manual's 'configure'
section](https://gcc.gnu.org/install/configure.html) does suggest that it is
supported on this platforn.

Building 64-bit toolchain binaries on AIX appears to be unsupported upstream
at the time of writing.  See this [bug from
2005](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=25119) for example. Passing
`--host=ppc64-` to configure has no effect. Forcibly adding `-maix64` to the
compile flags, `-X64` to the link and archive flags, and `OBJECT_MODE=64` in
the environment might work but some of these flags get lost by the nested
configure scripts and the build eventually fails due to having a mix of 32
and 64 bit binaries.

GCC has its own notes about the [AIX operating
system](https://gcc.gnu.org/install/specific.html#x-ibm-aix).
