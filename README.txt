LFS-systemd-Pacman
==========
This is simply my idea of the perfect Linux distro. I'm keeping it here for me, 
but feel free to take what you like should you happen to stumble upon it. I've 
combined much of LFS with Arch's package manager (pacman) and rolled my own.

It was built with my preferences alone. I prefer a system to be usable out of 
the box, but also to be as minimal as is possible. Unfortunately, I'm also a 
long time Gnome fan and old habits die hard, so it probably goes without saying 
that combination makes for a bit of a conflict. While I was fairly happy with
Arch Linux for my daily driver, they too had some pain points that I disagreed 
with. I won't go into detail, but Arch is a great distro, just not perfect for
me (as nothing is). And, once again, I was looking for my perfect distro. So,
here is the start of it. 

There will be some deviations from LFS, and I'll add comments
in the PKGBUILD files where necessary to make note of these. Of note for Arch
users, I've divided the package groups into Main and Extra. Since 
you are building from source, you should already be familiar with reciprocal 
dependencies. I've accounted for these in the PKGBUILD files using 
conditionals, and have some pretty stringent requriements for the PKGBUILD
files. See PKGBUILD-Standards.txt in the root of the tree for more details.
You'll also need to build the following packages from BLFS before anything here
will be useful:

http://www.linuxfromscratch.org/blfs/view/systemd/general/libgpg-error.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libassuan.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libgcrypt.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libksba.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/npth.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libusb.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libusb-compat.html
http://www.linuxfromscratch.org/blfs/view/systemd/postlfs/openssl.html
http://www.linuxfromscratch.org/blfs/view/systemd/basicnet/curl.html
http://www.linuxfromscratch.org/blfs/view/systemd/postlfs/cacerts.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/unzip.html
http://www.linuxfromscratch.org/blfs/view/systemd/server/sqlite.html
http://www.linuxfromscratch.org/blfs/view/systemd/postlfs/gnupg.html
http://www.linuxfromscratch.org/blfs/view/systemd/postlfs/gpgme.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libffi.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/python2.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libxml2.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/lzo.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libarchive.html
http://www.linuxfromscratch.org/blfs/view/systemd/pst/sgml-common.html
http://www.linuxfromscratch.org/blfs/view/systemd/pst/docbook.html
http://www.linuxfromscratch.org/blfs/view/systemd/pst/docbook-xsl.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/libxslt.html
http://www.linuxfromscratch.org/blfs/view/systemd/general/asciidoc.html

Additionally, you'll need to install fakeroot and pacman...

================================================================================

FAKEROOT:
http://ftp.debian.org/debian/pool/main/f/fakeroot/fakeroot_1.21.orig.tar.gz
http://www.linuxfromscratch.org/~dj/fakeroot-1.21-dlerror-1.patch

patch -Np1 -i ../fakeroot-1.21-dlerror-1.patch
autoreconf -fi
./configure --prefix=/usr  \
            --disable-static \
            --with-ipc=sysv
sed '/SUBDIRS/s@doc @@' -i Makefile
make
make install

================================================================================

PACMAN:
https://sources.archlinux.org/other/pacman/pacman-5.0.2.tar.gz

./configure --prefix=/usr \
            --sysconfdir=/etc \
            --localstatedir=/var \
            --enable-doc \
            --with-scriptlet-shell=/bin/bash \
            --with-ldconfig=/sbin/ldconfig
make
make -C contrib
make install
make -C contrib install
install -vDm644 contrib/PKGBUILD.vim \
                /usr/share/vim/vim80/syntax/
cat > /etc/pacman.conf << "EOF"
#
# GENERAL OPTIONS
#
[options]
# The following paths are commented out with their default values listed.
# If you wish to use different paths, uncomment and update the paths.
#RootDir     = /
#DBPath      = /var/lib/pacman/
#CacheDir    = /var/cache/pacman/pkg/
#LogFile     = /var/log/pacman.log
#GPGDir      = /etc/pacman.d/gnupg/
#HookDir     = /etc/pacman.d/hooks/
HoldPkg     = pacman glibc
#XferCommand = /usr/bin/curl -C - -f %u > %o
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u
#CleanMethod = KeepInstalled
#UseDelta    = 0.7
Architecture = auto

# Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
#IgnorePkg   =
#IgnoreGroup =

#NoUpgrade   =
#NoExtract   =

# Misc options
#UseSyslog
#Color
#TotalDownload
CheckSpace
#VerbosePkgLists

# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
#RemoteFileSigLevel = Optional

# NOTE: You must run `pacman-key --init` before first using pacman; the local
# keyring can then be populated with the keys of all official packagers.

#
# REPOSITORIES
#   - can be defined here or included from another file
#   - pacman will search repositories in the order defined here
#   - local/custom mirrors can be added here or in separate files
#   - repositories listed first will take precedence when packages
#     have identical names, regardless of version number
#   - URLs will have $repo replaced by the name of the current repo
#   - URLs will have $arch replaced by the name of the architecture
#
# Repository entries are of the format:
#       [repo-name]
#       Server = ServerName
#       Include = IncludePath
#
# The header [repo-name] is crucial - it must be present and
# uncommented to enable the repo.

# An example of a disabled remote package repository with multiple servers
# available. To enable, uncomment the following lines. You can add preferred
# servers immediately after the header and they will be used before the
# default mirrors.
#[core]
#SigLevel = Required
#Server = ftp://ftp.example.com/foobar/$repo/os/$arch/
# The file referenced here should contain a list of 'Server = ' lines.
#Include = /etc/pacman.d/mirrorlist

# An example of a custom package repository.  See the pacman manpage for
# tips on creating your own repositories.
#[custom]
#SigLevel = Optional TrustAll
#Server = file:///home/custompkgs

[Main]
SigLevel = Optional TrustAll
Server = file:///srv/pacman/repos/Main

[Extra]
SigLevel = Optional TrustALl
Server = file:///srv/pacman/repos/Extra
EOF

cat > /etc/makepkg.conf << "EOF"
#
# /etc/makepkg.conf
#

#########################################################################
# SOURCE ACQUISITION
#########################################################################
#
#-- The download utilities that makepkg should use to acquire sources
#  Format: 'protocol::agent'
DLAGENTS=('ftp::/usr/bin/curl -fC - --ftp-pasv --retry 3 --retry-delay 3 -o %o %u'
          'http::/usr/bin/curl -fLC - --retry 3 --retry-delay 3 -o %o %u'
          'https::/usr/bin/curl -fLC - --retry 3 --retry-delay 3 -o %o %u'
          'rsync::/usr/bin/rsync --no-motd -z %u %o'
          'scp::/usr/bin/scp -C %u %o')

# Other common tools:
# /usr/bin/snarf
# /usr/bin/lftpget -c
# /usr/bin/wget

#-- The package required by makepkg to download VCS sources
#  Format: 'protocol::package'
VCSCLIENTS=('bzr::bzr'
            'git::git'
            'hg::mercurial'
            'svn::subversion')

#########################################################################
# ARCHITECTURE, COMPILE FLAGS
#########################################################################
#
CARCH="x86_64"
CHOST="x86_64-pc-linux-gnu"

#-- Compiler and Linker Flags
CPPFLAGS=""
CFLAGS="-march=x86-64 -mtune=generic -O2 -pipe"
CXXFLAGS="-march=x86-64 -mtune=generic -O2 -pipe"
LDFLAGS="-Wl,-O1,--sort-common,--as-needed,-z,relro"

#-- Make Flags: change this for DistCC/SMP systems
#MAKEFLAGS="-j2"
#-- Debugging flags
DEBUG_CFLAGS="-g -fvar-tracking-assignments"
DEBUG_CXXFLAGS="-g -fvar-tracking-assignments"

#########################################################################
# BUILD ENVIRONMENT
#########################################################################
#
# Defaults: BUILDENV=(!distcc color !ccache check !sign)
#  A negated environment option will do the opposite of the comments below.
#
#-- distcc:   Use the Distributed C/C++/ObjC compiler
#-- color:    Colorize output messages
#-- ccache:   Use ccache to cache compilation
#-- check:    Run the check() function if present in the PKGBUILD
#-- sign:     Generate PGP signature file
#
BUILDENV=(!distcc color !ccache check !sign)
#
#-- If using DistCC, your MAKEFLAGS will also need modification. In addition,
#-- specify a space-delimited list of hosts running in the DistCC cluster.
#DISTCC_HOSTS=""
#
#-- Specify a directory for package building.
#BUILDDIR=/tmp/makepkg

#########################################################################
# GLOBAL PACKAGE OPTIONS
#   These are default values for the options=() settings
#########################################################################
#
# Default: OPTIONS=(strip docs !libtool !staticlibs emptydirs zipman purge !optipng !upx !debug)
#  A negated option will do the opposite of the comments below.
#
#-- strip:      Strip symbols from binaries/libraries
#-- docs:       Save doc directories specified by DOC_DIRS
#-- libtool:    Leave libtool (.la) files in packages
#-- staticlibs: Leave static library (.a) files in packages
#-- emptydirs:  Leave empty directories in packages
#-- zipman:     Compress manual (man and info) pages in MAN_DIRS with gzip
#-- purge:      Remove files specified by PURGE_TARGETS
#-- upx:        Compress binary executable files using UPX
#-- optipng:    Optimize PNG images with optipng
#-- debug:      Add debugging flags as specified in DEBUG_* variables
#
OPTIONS=(strip docs !libtool !staticlibs emptydirs zipman purge !optipng !upx !debug)

#-- File integrity checks to use. Valid: md5, sha1, sha256, sha384, sha512
INTEGRITY_CHECK=(sha256)
#-- Options to be used when stripping binaries. See `man strip' for details.
STRIP_BINARIES="--strip-all"
#-- Options to be used when stripping shared libraries. See `man strip' for details.
STRIP_SHARED="--strip-unneeded"
#-- Options to be used when stripping static libraries. See `man strip' for details.
STRIP_STATIC="--strip-debug"
#-- Manual (man and info) directories to compress (if zipman is specified)
MAN_DIRS=({usr{,/local}{,/share},opt/*}/{man,info})
#-- Doc directories to remove (if !docs is specified)
DOC_DIRS=(usr/{,local/}{,share/}{doc,gtk-doc} opt/*/{doc,gtk-doc})
#-- Files to be removed from all packages (if purge is specified)
PURGE_TARGETS=(usr/{,share}/info/dir .packlist *.pod)

#########################################################################
# PACKAGE OUTPUT
#########################################################################
#
# Default: put built package and cached source in build directory
#
#-- Destination: specify a fixed directory where all packages will be placed
#PKGDEST=/home/packages
#-- Source cache: specify a fixed directory where source files will be cached
#SRCDEST=/home/sources
#-- Source packages: specify a fixed directory where all src packages will be placed
#SRCPKGDEST=/home/srcpackages
#-- Log files: specify a fixed directory where all log files will be placed
#LOGDEST=/home/makepkglogs
#-- Packager: name/email of the person or organization building packages
#PACKAGER="John Doe <john@doe.com>"
#-- Specify a key to use for package signing
#GPGKEY=""

#########################################################################
# COMPRESSION DEFAULTS
#########################################################################
#
COMPRESSGZ=(gzip -c -f -n)
COMPRESSBZ2=(bzip2 -c -f)
COMPRESSXZ=(xz -c -z -)
COMPRESSLRZ=(lrzip -q)
COMPRESSLZO=(lzop -q)
COMPRESSZ=(compress -c -f)

#########################################################################
# EXTENSION DEFAULTS
#########################################################################
#
# WARNING: Do NOT modify these variables unless you know what you are
#          doing.
#
PKGEXT='.pkg.tar.xz'
SRCEXT='.src.tar.gz'

# vim: set ft=sh ts=2 sw=2 et:
EOF

================================================================================

Once these are done, you can begin building packages located in the "Main"
directory to replace everything already installed. I have created a package
group "core" to describe all of the above software along with everything up
to the rebuild of systemd as per BLFS-systemd. Main will contain packages
lited in both books, and Extra will contain packages that are outside of the
scope of BLFS. Obviously, you'll need to use --nodeps and --force freqently
until all of the packages are reinstalled.

I've also added Main and Extra repositories in the pacman configuration above.
You can develop a workflow that incorporates this by creating these repos...

$ install -vdm755 /srv/pacman/repos/{Main,Extra}
$ cd /srv/pacman/repos/Main
$ repo-add Main.db.tar.gz
$ cd ../Extra
$ repo-add Extra.db.tar.gz

After building a package, copy the pkg file to the Main or Extra directory,
and run the repo-add command again.

$ repo-add /srv/pacman/repos/Main/Main.db.tar.gz \
           /srv/pacman/repos/Main/package-0.1-1.pkg.tar.xz

Then do a 'pacman -Syu' followed by 'pacman -S package --force --nodeps'.
The force and nodeps switches will be requried until all of the LFS packages,
and the packages above have been reinstalled (else pacman will refuse to
install the package because files already exist, or dependencies are not met).
After these are completed, you can drop the force and nodeps flags. As an aside,
you'll have a repository ready to go that can be used with the regular Arch
install disk to prepare a new system, just point it to your remote repository
and use 'pacstrap /mnt core' to install the minimal required system to boot.

More to come...

Enjoy.

-- DJ Lucas
