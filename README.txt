LFS-Pacman
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
You'll also need to build several packages beyond LFS to create your stage 2
build environment. http://www.linuxfromscratch.org/~dj/add-pacman.svnstash is a
patch the LFS book that contians the necessary changes. Additionally, if you
wish to use jhalfs to build the book, you'll need to patch it to allow for more
than 100 sets of instructions per chapter. The patch for this change is at
http://www.linuxfromscratch.org/~dj/jhalfs-4196-100pkgs-1.patch which will
allow you to use jalfs to build for pacman bootstrap. Finally, the kernel build
provided here is akin to a distro kernel, you will need to have cpio and the
linux firmwae packages available. Again, I've provided a patch for this at
http://www.linuxfromscratch.org/~dj/add-linux-firmware-and-cpio.svnstash which
will work in conjunction with the above LFS-Book patch.

================================================================================

Once the stage 2 build is complete, you can begin building packages located in
the "Main" repo to replace everything already installed. I would recommend
simultaneously installing directly to a stage 3 parition, but this is not
strictly necessary. I have created a package group "core" to include all of the
sofware up to this point. This is the minimum requirement to manage the system
and to boot. 

I've also added Main and Extra repositories in the pacman configuration above.
You can develop a workflow that incorporates this by working withing them.
After building a package, copy the pkg file to the Main or Extra directory,
and run the repo-add command again.

$ repo-add /srv/pacman/repos/Main/Main.db.tar.gz \
           /srv/pacman/repos/Main/package-0.1-1.pkg.tar.xz

Then do a 'pacman -Syu' followed by 'pacman -S package -dd --overwrite "*"'.
The overwrite and dd switches will be requried until all of the LFS packages,
and the packages above have been reinstalled (else pacman will refuse to
install the package because files already exist, or dependencies are not met).
After these are completed, you can drop the force and nodeps flags. As an aside,
you'll have a repository ready to go that can be used with the regular Arch
install disk to prepare a new system, just point it to your remote repository
and use 'pacstrap /mnt core' to install the minimal required system to boot.

Enjoy.

-- DJ Lucas
