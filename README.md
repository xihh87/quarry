### Rubygems binary packages repository a.k.a Quarry

Quarry is a tool that manages rubygems binary repository for Linux Arch. The binary packages repository hosted at http://pkgbuild.com/~anatolik/quarry, currently only 64 bit is supported. If you want to use the quarry repository add following lines to */etc/pacman.conf* file:

```
[quarry]
Server = http://pkgbuild.com/~anatolik/quarry/x86_64/
````

Unlike packages from AUR using the binary repository is better because:
 - installing packages from binary repo is much easier and faster.
 - quarry packs minimal set of files needed for a user. E.g. rubygems with native extensions contain 3 (three!) copies of the same *.so files, rubygems install a lot of garbage such as test file, readmes, etc..
 - quarry handles dependencies correctly.
 - a new version appears in Arch repo soon after it is published in rubygems, we are talking about hours latency here.

#### Quarry tool internals

Source code is hosted at github https://github.com/anatol/quarry and released under GPL3 license.

Quarry manages gems listed in *whitelist_packages* file plus all dependencies of these packages. If you want to see your package in the binary repo add it to https://github.com/anatol/quarry/blob/master/whitelist_packages

Converting a gem into binary Arch package is simple and straighforward for most gems. Some rubygems need additional configuration. This specific information is stored in *config.pkg/$PACKAGE.yaml* config files. These are YAML files with following fields:

  * **depends** - array of additional dependencies. Quarry extracts ruby dependencies from gem specification file, but if you want to add native dependencies you should list it here.
  * **makedepends** - array of additional makedepends.
  * **optdepends** - a map of optional dependencies. It has the same structure as PKGBUILD optdepends field - (dependency, description).
  * **include** - by default Quarry copies only *bin* and *lib* gem directories. If you want to copy other files/directories then add it here.
  * **rename** - a map of pairs that rename files in /usr/bin/. Most rubygems files are kept in rubygems specific folders under /usr/lib/ruby/gems. But some rubygems create files under /usr/bin and such files can conflict with popular packages. To avoid this we added mechanism to prevent conflict by renaming files in /usr/bin. This config option is map of 'from_name: to_name' renames.
  * **gem_install_args** - addisional argumets used during `gem install` step. It can be useful to pass parameters like `--use-system-libraries`

If slot version has no config then it checks config for less specific slot. E.g. if there was a slot version *rails-3.2.6* then it would check configs for *rails-3.2.6*, *rails-3.2*, *rails-3*, *rails* until it finds an existing file. 

If gem requires patching then you can add $PACKAGE.patch file and it will be applied automatically before installing the gem.

File *rebuild_packages* contains list of gems to force rebuild with new pkgrel. It might be useful if e.g. configuration for the gem has changed.
