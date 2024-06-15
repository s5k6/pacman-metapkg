
TODO
----

  * Pacman fails to remove dropped dependencies.

  * Requirement to run script as non-rrot, but install as root is
    cumbersome.


Pacman
------

From the following, omit the `-q` to get version information.

  * List explicitly installed (-e) packages which additionally are not
    required (-t) by other packages:

        pacman -Qetq   # explicit

  * List real orphans — packages that were installed as dependencies
    (-d) but are no longer required (-t) by any installed package.

        pacman -Qdtq   # orphans


Metapackages
------------

A metapackage is simply a list of dependencies, optinally with version
constraints.

    $ cat example

Creation of a metapackage (must be non-root):

    $ ./mkmetapkg example
    …
    /tmp/metapkg/metapkg_example-1718459133-1718459366-any.pkg.tar.zst

**Note:** The packagename has a `metapkg_` prefix added to the package
name.

The created package is almost empty.  Its version number is the mtime
of the dependency list used, its release number is the mtime of
packaging.

Install the file using `pacman -U`, i.e.,

    # pacman -U /tmp/metapkg/metapkg_example-1718459133-1718459366-any.pkg.tar.zst

**Note:** If dependencies are droppd from the metapackage, then they
will not be uninstalled when upgrading the metapackage to a newer
version or release.
