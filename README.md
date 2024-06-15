---
author: Stefan Klinger <http://stefan-klinger.de>
---


Why meta packages
=================

Context
-------

With Arch Linux `pacman` there are basically [two ways][1] to organise
packages in groups: “Package groups” and “meta packages”.

The “package groups” concept is rather a tagging mechanism, where the
packager (the person defining the package) defines the group(s) a
package belongs to.  This comes with some logics implemented in
`pacman`, such as asking the user which package from a group to
install, or to remove all packages in a group.

The “meta packages” concept lies outside of `pacman`: The packagers of
the individual packages have little say in what you put into a meta
package (conflicts aside).  In creating a meta package, you can
combine a pretty arbitrary set of, from the viewpoint of their
packagers, unrelated packages.


Situation
---------

Creating packages through `makepkg` (which refuses to run as root)
with `PKGBUILD` files needing individual directories for each meta
package, and then running `pacman` as root anyways, seemed
unnecessarily complicated.

The provided script cuts some corners in creating the package file for
`pacman`, it does not go through `makepkg`.  This is appropriate,
since the additional complexity is not needed.  However, the approach
hinges on the package format being understood correctly.


Dead simple meta packages
=========================

A meta package is constructed from a simple listing of dependencies,
each optionally with version constraints.  Lines with a leading `#`
are ignored.  Leading and trailing spaces are not allowed.

    # cat example
    foot
    meson>1.0.0


Installation
------------

    # metapkg example

This creates a temporary package named `metapkg_example` with no
contents on its own, but using the dependencies given in file
`example`.

The temporary package is then installed via `pacman -U` and discarded
afterwards.  You do not need it any more, its effect on the system is
now in the `pacman` database.  You may inspect it with

    # pacman -Qi metapkg_example


Listing
-------

The installed meta packages all have the `metapkg_` prefix in their
name.  This makes it easy to manage them with the usual tools.  E.g.,
list the installed meta packages:

    # pacman -Qsq '^metapkg_'


Removal
-------

The meta package (rather: the artefacts of it being installed) can be
simply removed by

    # pacman -Rs metapkg_example

which is standard `pacman` procedure to remove a package and its
dependencies.  Of course, you may choose to remove only the meta
package with `-R` instead of `-Rs`.  Then unneeded dependencies of the
meta package will remain as orphans.  Try `pacman -Qdtq` for a listing
of those.


Updating the packages in a meta package
---------------------------------------

After modifying the source file `example`, all you need to do is run

    # metapkg example

again.  `pacman` will automatically pick up the changes, because the
version number of the temporary package is the mtime of the dependency
list used, and the release number is the time of packaging, thus
creating a newer meta package every time.

Unfortunately, `pacman` does not trace dependencies being dropped by a
(meta) package during updates.  I.e., if a package A-1.0 with
dependency B would be updated to a new version A-2.0, which does not
need B any more, then B would not be removed from the system, even if
A-1.0 was its only dependee, and B was only installed as a dependency.
Instead, B would remain as an orphan.  To catch this, `metapkg`
observes new orphans introduced by a modified meta package, and offers
to remove them right away.


[1]: https://wiki.archlinux.org/title/Meta_package_and_package_group
