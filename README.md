
Metapackages
------------

A metapackage is simply a list of dependencies, optionally with
version constraints.  Lines with a leading `#` are ignored.  Leading
and trailing spaces are not allowed.

    $ cat example

Running

    # metapkg example

creates a temporary package named `metapkg_example` with no contents
on its own, but using the given dependencies.

The package is installed by `pacman -U` and then discarded.  It can be
simply removed by

    # pacman -Rsn metapkg_example

which is standard `pacman` procedure.

Its version number is the mtime of the dependency list used, its
release number is the mtime of packaging.  So after modifying
`example`, the selection of installed packages can be simply updated
by running

    # metapkg example

again.

Unfortunately, `pacman` does not trace dependencies being dropped by a
package during updates.  To this end, `metapkg` observes new orphans
being introduced by such an upgrade, and offers to remove them.
