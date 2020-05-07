# Freifunk Berlin Firmware
https://wiki.freifunk.net/Berlin:Firmware

This is the build-system for the Firmware of Freifunk Berlin.
The firmware is based on vanilla [OpenWrt](https://openwrt.org/start) with some modifications (to fix
broken stuff in OpenWrt itself or for example LuCI) and additional default packages/configuration settings.

## Contact / More information

More user relevant information about the firmware are on the wiki page at: https://wiki.freifunk.net/Berlin:Firmware. There you can also find the
* [ReleaseNotes](https://wiki.freifunk.net/Berlin:Firmware/v1.0.2)
* a tutorial ([en](https://wiki.freifunk.net/Berlin:Firmware:En:Howto) / [de](https://wiki.freifunk.net/Berlin:Firmware:Howto)) on router configuration

For questions write a mail to <berlin@berlin.freifunk.net> or come to our weekly meetings.
If you find bugs please report them at: https://github.com/freifunk-berlin/firmware/issues

## Development

### Info

For the Berlin Freifunk firmware we use vanilla OpenWrt with additional patches
and packages. The Makefile automates firmware
creation and apply patches / integrates custom freifunk packages. All custom
patches are located in *patches/* and all additional packages can be found at
http://github.com/freifunk-berlin/packages_berlin.

### Build Prerequisites

Please take a look at the [OpenWrt documentation](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem?s[]=prerequisites#prerequisites)
for a complete and uptodate list of packages for your operating system. Make
sure the list contains `quilt`. We use it for patch management.

On Ubuntu/Debian:
```
apt-get install git build-essential libncurses5-dev zlib1g-dev gawk time \
  unzip libxml-perl flex wget gawk libncurses5-dev gettext quilt python3 libssl-dev
```

On openSUSE:
```
zypper install --type pattern devel_basis
zypper install git ncurses-devel zlib-devel gawk time \
  unzip perl-libxml-perl flex wget gawk gettext-runtime quilt python libopenssl-devel
```
On Arch/Antergos:
```
pacman -S base-devel git ncurses lib32-zlib gawk time unzip perl-xml-libxml \
 flex wget gettext quilt python2 openssl
```

### Building all firmwares

To get the source and build the firmware locally use:

```
git clone https://github.com/freifunk-berlin/firmware.git
cd firmware
make
```

The build will take some time. You can improve the build time with [build options](https://openwrt.org/docs/guide-developer/build-system/use-buildsystem)
such as `-j <number of cores>`. `V=s` will give more verbose error messages.

An internet connection is required during the build process. A good internet
connection can improve the build time.

You need approximately 10GB of space for the build.

### Directory Layout

You can find the actual firmware images generated by the ImageBuilder (and the ImageBuilder itself)
in `firmwares`. The layout looks like the following:

```
firmwares/
    TARGET/
        backbone/
           images..
        default/
           images..
        ...
        OpenWrt-ImageBuilder-....tar.xz
        OpenWrt-SDK-....tar.xz
        initrd/
           images..
        packages/
           packages/<ARCH>
              base/*.ipk
              luci/*.ipk
              packages/*.ipk
              packages_berlin/*.ipk
              routing/*.ipk
           targets/MAINTARGET/SUBTARGET/packages/
              *.ipk
```

As you notice there are several different image variants ("backbone", "default", etc.).
These different *packages lists* are defined in `packages/`.
See the "Features" section above for a description of the purpose of each package list.
With the "OpenWrt-Imagebuilder" you can assemble your own image variant with your
*packages lists* without having to compile everything yourself. The "OpenWrt-SDK" is
the fastest way to build your own packages or programs without compiling OpenWrt itself.
The "initrd" directory contains some initrd-images for netboot, which are required on
some boards to initially install OpenWrt.

### customizing make

`make` will use by default `TARGET` and `PACKAGES_LIST_DEFAULT` defined in
`config.mk`. You can customize this by overriding them:

```
make TARGET=mpc85xx PACKAGES_LIST_DEFAULT=backbone
```
in addition you can build your own image from a prebuilt imagebuilder by something like:

```
make images IB_FILE=<file> TARGET=... PACKAGES_LIST_DEFAULT=...
```

The default target is `ar71xx-generic`. For a complete list of supported targets look in `configs/` for the target-specific configs.
Each of these targets need a matching file in `profiles/` with the profiles (boards) that should be build with the imagebuilder.

additional options

* IS_BUILDBOT :
  * this will be "yes" when running on the buildbot farm and helps to save some disc-space by removing files not required anymore. On manual builds you should not set this to "yes", as you have to rebuild the whole toolchain each time.
* SET_BUILDBOT :
  * "env" the Makefile will honor the "IS_BUILDBOT" environment
  * "yes" the Makefile will always act as "IS_BUILDBOT" was set to "yes"
  * "no"  the Makefile will always act as "IS_BUILDBOT" was set to "no" / is unset. This way we can run builds on the buildbot like a local build.

### Continuous integration / Buildbot

The firmware is [built
automatically](http://buildbot.berlin.freifunk.net/one_line_per_build) by our [buildbot farm](http://buildbot.berlin.freifunk.net/buildslaves). If you have a bit of CPU+RAM+storage capacity on one of your servers, you can provide a buildbot slave (see [berlin-buildbot](https://github.com/freifunk/berlin-buildbot)).

All branches whose name complies to the "X.Y.Z" pattern are built and put into the "stable" downloads directory:
[http://buildbot.berlin.freifunk.net/buildbot/stable/](http://buildbot.berlin.freifunk.net/buildbot/stable/)

All branches with names not fitting the "X.Y.Z" pattern are built and put into the "unstable" directory:
[http://buildbot.berlin.freifunk.net/buildbot/unstable/](http://buildbot.berlin.freifunk.net/buildbot/unstable/)
Note that in the directory there is no reference to the branch name; unstable builds can be identified by build number only.

#### Creating a release

Every release has a [semantic version number](http://semver.org); each major version has its own codename.
We name our releases after important female computer scientists, hackers, etc.
For inspiration please take a look at the related
[ticket](https://github.com/freifunk-berlin/firmware/issues/24).

For a new release, create a new branch. The branch name must be a semantic version
number. Make sure you change the semantic version number and, for major releases,
the codename in the README and config files (./configs/*)

The buildbot will build the release and place the files in the stable direcotry
once you pushed the new branch to github.

### Patches with "git format-patch"

**Important:** all patches should be pushed upstream!

If a patch is not yet included upstream, it can be placed in the corresponding subdirectory below the`patches`
directory. To create a correct patch-file just use the [`git format-patch`](https://git-scm.com/docs/git-format-patch) command.

#### Create a patch

In order to add a patch file update your build environment by running:

```bash
make clean patch
```
Then switch to the openwrt directory:

```bash
cd openwrt
```

or continue to the relevant feed directory:

```bash
cd feeds/luci
```

use the normal `git commit` workflow to apply your changes to the code. When done convert your last commit 
into a patch by running:

```bash
git format-patch --start-number <n> HEAD^
```
where `n` is the next free number of the correlating patch-subdirectory. You can use something like `HEAD^^^^`
to create patch-files from you last 4 commmits, or even use a git-rev directly. Feel free to squash multiple 
commits into a single one before creating the patch-file or use something like 

```bash
git format-patch --stdout HEAD^^^^ > patches/routing/0008-awesome.patch
```
to create a single file of these 4 commits

#### Modify a patch

To update an existing patch do the same as above:

```bash
make clean patch
cd openwrt
cd feeds/luci
```
Then just add a new commit with your changes and squash it with the commit relating to the patch-file.
To update the patch-file use the same `git format-patch` sequence as you did when creating the patch
initially.

#### Delete a patch

To remove a patch-file you have to remove it from the patch-subdirectory and update the build-
environment:

```bash
git rm patches/openwrt/0010-unrelevant-change.patch
make patch
```

### Submitting patches

#### Freifunk Berlin

Please create a pull request for the project you want to submit a patch.
If you are already member of the Freifunk Berlin team, please delete branches once they have been merged.

#### OpenWrt

Create a commit in the openwrt directory that contains your change. Use `git
format-patch` to create a patch:

```
git format-patch origin
```

Send a patch to the OpenWrt mailing list with `git send-email`:

```
git send-email \
  --to=openwrt-devel@lists.openwrt.org \
  --smtp-server=mail.foo.bar \
  --smtp-user=foo \
  --smtp-encryption=tls \
  0001-a-fancy-change.patch
```

Additional information: https://dev.openwrt.org/wiki/SubmittingPatches
