---
title: Dead simple local meta packages for Arch Linux Pacman
author: Stefan Klinger <http://stefan-klinger.de>
---

Meta packages should be as simple as one command and a file listing
the dependencies.


Why meta packages
=================

Context
-------

With Arch Linux `pacman`(8) there are basically [two ways][1] to
organise packages in groups: “Package groups” and “meta packages”.

The “package groups” concept is rather a tagging mechanism, where the
packager (the person defining the package) defines the group(s) a
package belongs to.  This comes with some logics implemented in
`pacman`, such as asking the user which package from a group to
install, or to remove all packages in a group.

The concept of “meta packages” comes from the other direction: The
packagers of the grouped packages have little control (conflicts
aside) over what you, the user, put into a meta package.  In creating
a meta package, you can combine a pretty arbitrary set of, from the
viewpoint of their packagers, unrelated packages.


Situation
---------

You will find plenty tutorials for creating meta packages with
`makepkg`(8), which refuses to run as root for good reason.  Each meta
package demands its own `PKGBUILD` file, residing in its own
individual directory.  After creation of the package file (they do pile
up), run `pacman` as root.

I hope to make local meta packages (this may not extend to meta
packages intended for distribution) as easy as a single file listing
the dependencies, and one simple command to run.


Dead simple meta packages
=========================

The provided script cuts some corners in creating the package file for
`pacman`, it does not go through `makepkg`.  This seems appropriate,
since the additional complexity of preparing the installable content
(there is none) is not needed.  However, the approach hinges on the
package format being understood correctly, maybe **there are bugs**
lurking somewhere…

A meta package is constructed from a simple listing of dependencies,
each optionally with version constraints.  Lines with a leading `#`
are ignored.

    # cat example
    foot
    meson>1.0.0


Installation
------------

    # metapkg example

This creates a *temporary* package file for the package
`metapkg_example` — it has no contents on its own, but uses the
dependencies given in file `example`.

The temporary package is then installed via `pacman -U` and
*discarded* afterwards (unless `-k` was passed to the script).  You do
not need it any more, its effect on the system is now in the `pacman`
database.  You may inspect this with

    # pacman -Qi metapkg_example


Listing
-------

The installed meta packages all have the `metapkg_` prefix in their
name.  This makes it easy to manage them with the usual tools.  E.g.,
list the installed meta packages:

    # pacman -Qsq '^metapkg_'

Yes, `metapkg` deviates from the naming convention of meta package
names ending with `-meta`.  The intention is to distinguish meta
packages managed by `metapkg` from those distributed via official
package repositories.


Removal
-------

The meta package can be simply removed by

    # pacman -Rs metapkg_example

which is standard `pacman` procedure to remove a package and its
dependencies.

Of course, you may also choose to remove only the meta package with
`-R` instead of `-Rs`.  Then unneeded dependencies of the meta package
will remain as orphans.  Try `pacman -Qdtq` for a listing of those.


Changing the meta package
-------------------------

After modifying the source file `example`, all you need to do is run

    # metapkg example

again.

`pacman` will automatically pick up the changes, because the version
number of the meta package is the modification time of the dependency
list used, thus creating a newer meta package every time.  The format
of the timestamp is `%Y%m%d%H%M%S`, cf. `date`(1).

Unfortunately, `pacman` does not trace dependencies being dropped by a
(meta) package during updates.  I.e., if a package A-1.0 with
dependency B would be updated to a new version A-2.0, which does not
need B any more, then B would not be removed from the system, even if
A-1.0 was its only dependee.  Instead, B would remain as an orphan.
To catch this, `metapkg` observes new orphans introduced by a modified
meta package, and offers to remove them right away.


Finally
=======

This is an experimental endeavour.  Maybe meta packages do not turn
out as useful as I hope them to be…


[1]: https://wiki.archlinux.org/title/Meta_package_and_package_group
