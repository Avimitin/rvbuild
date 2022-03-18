# RVBUILD

This is a script that help me build package on RISCV machine.
Most of the server I can access are far from me.
So it is laggy to type command in ssh.
I want this script to help me automate some build jobs,
so I don't need to run lot's of command in that laggy terminal.

# Some concept about this script

## 1. How to select the server

First user needs to define a list of available SSH server.
Then this script will iterate the list and find the server with loadest load,
and set it as the main operating server.

This process will be buffered. The script will create a `.ctx.json` file.
And write the server name and the test time into it.

```json
{
  "server": "shinx",
  "ttimestamp": 0
}
```

Since most packagers are working in the same UTC, so the buffer will be kept
with only 30 minutes. This can be a configurable value.

## 2. How to run a build

This script will concate command line string to behave a build.
A build is separate into 4 stages.

### 2.1 Prepare

The script will first create directory on the build server.
It will manage all packages inside one directory.
By default it will create a `$HOME/riscv/packages` directory under the
home directory. And all the PKGBUILD file will be checked out inside it.

Then it will create a cache directory for the package file.
It will be `$HOME/.cache/rvpkgcache/` by default.

You can configured the path by setting the environment variable `$RVPKGPATH`
and `$RVPKGCACHE`.

### 2.2 Build

Run this script by command:

```console
rvbuild <pkgname>
```

The script will first try to find if this package is already exist in the
`RVPKGPATH` directory.

If it is exist, it will cd into the package directory and run the build.

If not, the pkgname will be concated into the asp command:

```console
asp update && asp checkout <pkgname>
```

Then the script will cd into the checkouted directory and run the below command.

```console
extra-riscv64-build -- -d "$RVPKGCACHE:/var/cache/pacman/pkg/"
```

### 2.3 Fix

If the build is not exit successfully, the script will try to use rsync to
download PKGBUILD and source directory. This files will be placed in the
same directory as the script with the below directory structure.

* script
* src/pkgname/
* PKGBUILD

When you run the `rvbuild -r <pkgname>` (-r enable rebuild mode), the script
will send the local PKGBUILD to the remote server and run the build again.

The script will not handle the source, it is responsible for packager to
handle it.

### 2.4 Done

When you run command `rvbuild done`, the script will clean up the local directory.

---

This script is open source under the GNU GPLv3 License.
