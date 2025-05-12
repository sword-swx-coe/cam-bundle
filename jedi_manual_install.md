# JEDI Dependencies Manual Install

## Background

If you are running JEDI on an HPC system such as NCAR Derecho, then you probably do not need to read this document.  Chances are good that the jedi dependencies are already installed and maintained on your system and all you have to do is to load environment modules.

For more information on whether JEDI modules are already available on your system and, if so, how to use them, see the JCSDA list of [pre-configured sites](https://spack-stack.readthedocs.io/en/latest/PreConfiguredSites.html#aws-ubuntu-24-04).

However, even if you have access to one of these HPC systems, it can be beneficial for development to install JEDI on your own workstation or laptop.

The normal way to do that is to use the [spack-stack](https://github.com/JCSDA/spack-stack) repository on GitHub maintained by .  For further details, see the [spack-stack documentation](https://spack-stack.readthedocs.io/en/latest/index.html).

The JCSDA instructions on how to build spack stack for JEDI are [here](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/using/jedi_environment/spackbuild.html).

I tried these instructions on a new ubuntu laptop and I ran into problems.  I know others have also run into problems with spack-stack.  If you run into problems, one option is to post questions to the [JCSDA forum](https://forums.jcsda.org/) (JEDI category).

This document is another option.  It describes how to do a more manual installation of JEDI.  It might take longer than the built-in configurations but I (MM) personally prefer the greater control it gives you on exactly what to install.  Sometimes the full installation gives you problems with packages you don't really need to run cam-jedi.  And, in my opinion, it is easier to troubleshoot when things don't work as expected.  

These notes describe how I installed JEDI recently on two ubuntu systems (my new linux laptop and a WSL domain on my Windows desktop).  You can follow a similar procedure on other linux or Mac operating systems.

## Prerequisites

Start with [these instructions](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/using/jedi_environment/spackbuild.html).  In this document, we will refer to this web page as the "JCSDA instructions".

The first few paragraphs explain how to install some basic packages on your system before the more specific jedi dependencies, with external links for Mac or linux operating systems.  This normally requires that you have administrator (root) access on your system.  If you do not, you will have to work with someone who does.

Clicking on the links there will take you to [this page](https://spack-stack.readthedocs.io/en/latest/NewSiteConfigs.html), which we will refer to as the "Configuration instructions".  For Ubuntu, I want down to section 6.2.2.

Note that you can skip the parts that mention R2D2.  So, for example, I skipped the installation of `mysql-server` and `libmysqlclient-dev`.

You can also skip the installation of `environment-modules` if you want to use lmod instead, as described in the next section.

I also skipped steps 2 and 3 in section 6.2.2 because I want to use lmod (see next section) and I don't want to bother with spack environments.

## Environment modules

Before proceeding to section 6.2.3, I set up lmod.  This is optiona

If you have worked with HPC systems, you are probably familiar with environment modules.  This is when you enter `module load` to load the software packages you need for a given application.

It is easy to install environment modules on your own laptop or workstation too.  There are two main types of environment modules that are in general use.  One is based on the `tcl` language and the other, called [lmod](https://lmod.readthedocs.io/en/latest/) is based on the `lua` language.

Both options are available with spack, and spack can create modules automatically as you install packages.

I personally prefer `lmod`, which is used on many HPC systems.  For ubuntu, I find that the easiest way to install lmod is through the `luarocks` package as follows:

```bash
sudo apt-get install luarocks
sudo luarocks install luaposix
sudo luarocks install luafilesystem
sudo apt-get install lmod
```

```bash
sudo ln -s /usr/share/lmod/lmod/init/profile /etc/profile.d/z00_lmod.sh
```

On my WSL system, this was sufficient.  

However, on my laptop, I had to explicitly link the lmod init scripts to my startup profile.

```bash
sudo ln -s /usr/share/lmod/lmod/init/profile /etc/profile.d/z00_lmod.sh
sudo ln -s /usr/share/lmod/lmod/init/cshrc /etc/profile.d/z00_lmod.csh
```

But weirdly, even that did not work.  I also had to explictly source the init script in my `~/.bashrc` file (you can do something similar in your `~/.tcshrc` file if you use `tcsh`):

```bash
# for lmod
. /etc/profile.d/z00_lmod.sh
```

To check, log out of a terminal and log in again.  Enter `module list` and you should see something like this

```bash
~:$ module list
No modules loaded
```

That means it it properly installed and configured.  You can also try other lmod commands like `module help` and `module spider`.  For complete usage information see the [lmod documentation](https://lmod.readthedocs.io/en/latest/).


## Clone the spack-stack repo and set up spack

Start with [these instructions](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/using/jedi_environment/spackbuild.html).  In this document, we will refer to this web page as the "JCSDA instructions".

Warning: The spack-stack is based on [spack](https://spack.io/), which is a cross-platform package manager.  The spack-stack repository includes its own version of spack.  It is important to use the spack-stack version (which is a fork of the original) because it includes build instructions for packages that are not available in the generic spack distribution.

So, when you clone the spack-stack repo, make sure you include the `--recursive` option as described in the JCSDA instruction, which will download the spack fork along with the other components of the repo.  

I recommend downloading the repo and then checking out the latest tag, which happens to be 1.9.1 at the time of writing (May 2025).  Note this is slightly different from the clone command in the JCSDA instructions.

```bash
git clone --recursive https://github.com/JCSDA/spack-stack spack-stack
cd spack-stack
git checkout 1.9.1
```

To initialize spack, you have to source the `setup.sh` script as follows:

```bash
source .setup.sh
```

But, I find that if I do not do this in the `spack-stack` directory, it gives me the following warning:

`fatal: not a git repository (or any of the parent directories): .git`

It will still work, but to initialize spack on every log in  and to get rid of the annoying warning message, I added this to my `.bashrc` 

```bash
# initialize spack
cd /home/miesch/jedi/spack-stack && source ./setup.sh && cd $HOME
```

The second command lists what compilers spack knows about.  If you installed gcc as described above, it should find that.

#### Configure spack

The next step in the configuration instructions is to create a space environment and "concretize" the jedi dependencies.  Howver, these instructions did not work for me.  I had multple problems with both the `spack create` command and the `concretize` command.  

But, you don't really need a spack environment.  And, you do not really need all the packages in the various site templates to run a simple application like `cam-jedi` (currently pretty simple) or `ufo`.  So, rather than trying to debug those commands, I prefer a simpler, more hands-on approach of installing packages manually.

The [spack documentaion](https://spack.readthedocs.io/en/latest/) is very good and very easy to read. Refer there for details on how to customize your installation.

Though not required, it is good practice to install the same versions of the software dependencies that are specified in the latest release of `spack-stack`.  This can help avoid some versions of packages that may not work together well, which does happen occasionally.

As specified in the [spack documentation](https://spack.readthedocs.io/en/latest/packages_yaml.html), you can specify your preferred versions and build options for packages in a `packages.yaml` file that you put here: `~/.spack/packages.yaml`.

Most of the relevant packages to build the jedi core repositories are in the `configs/common/packages.yaml` file.  So, copy this file over to your personal configuration:

```bash
~/jedi/spack-stack:$ cp configs/common/packages.yaml ~/.spack
```

Now customizatize your package configuration.  First run these commands to let spack find and then list your compilers:

```bash
spack compiler find
spack compilers
```

For me, this lists `gcc@13.3.0`, which was installed in one of the steps above.  So, this line tells spack to use that compiler to build all the JEDI dependancies and to use openmpi version 5.0.3 for all parallel software that requires mpi.

```bash
spack config add "packages:all:compiler:gcc@13.3.0"
spack config add "packages:all:providers:mpi:openmpi@5.0.3"
```

This makes changes to your `~/.spack/packages.yaml` file - you should be able to see those changes in an editor to confirm.

For other changes, I found it easier to edit that file directly.  Since I am using gcc, there are some other changes suggested by the `spack-stack/configs/common/packages_gcc.yaml` file.  In particular, I added the last three lines to the `all:providers` entry, as shown here:

```yaml
  all:
    providers:
      gl: [opengl]
      glu: [openglu]
      jpeg: [libjpeg-turbo]
      pkgconfig: [pkg-config]
      yacc: [bison]
      zlib-api: [zlib-ng]
      mpi:
      - openmpi@5.0.3
      blas: [openblas]
      fftw-api: [fftw]
      lapack: [openblas]
```

And I added the second line to the ectrans entry:

```yaml
  ectrans:
    require:
    - '@1.5.0'
    - '~mkl +fftw'
```

Also, I found that this python requirement was causing me problems so I commented it out (jedi has some sophisticated inter-operability between python and C++ but that is not really needed, and I'm not using intel compilers)

```yaml
  # Turn off crypt variant for Python; this leads to build errors
  # with Intel in py-cryptography unless external curl and openssl
  # are removed, which itself is problematic.
  #python:
  #  require: '@3.11.7 ~crypt'
```

and later in the file

```yaml
#  python:
#    externals:
#    - spec: python@3.12.3+bz2+crypt+ctypes+dbm+lzma+pyexpat~pythoncmd+readline+sqlite3+ssl~tkinter+uuid+zlib
#      prefix: /usr
```


If you are using lmod environment modules as described above, you can also enter this to tell spack that's what you want to use (this may create a `~/.spack/modules.yaml` file):

```bash
spack config add modules:default:enable:[lmod]
```

#### Manually install jedi dependencies

Now you are ready to actually install some things.

The first step here is to let spack know about the packages you already have installed on your system

```bash
spack external find --exclude cmake \
    --exclude curl --exclude openssl \
    --exclude openssh
spack external find python
spack external find grep
spack external find sed
spack external find perl
spack external find wget
spack external find texlive
```

Now try installing your first package.  Start with something simple.

```bash
spack install udunits
```

Now if you enter `spack find` you should see udunits there, but also some other things that were needed for udunits.  Now generate environment modules for these:

```bash
spack module lmod refresh
```

To find where the modules are, cd to `spack-stack/share` and run this:

```bash
find . -name \*.lua
```

For me, this led me to this directory, where there are subdirectories for udunits and several other things.

```bash
/home/miesch/jedi/spack-stack/spack/share/spack/lmod/linux-ubuntu24.04-x86_64/Core
```

I added this line to my `~/.bashrc` file so it will be executed whenever I log in:

```bash
module use /home/miesch/jedi/spack-stack/spack/share/spack/lmod/linux-ubuntu24.04-x86_64/Core
```

Then I activated it with `source ~/.bashrc`.

Now you should be able to see the spack-installed modules:

```bash
module avail
module load udunits
module list
```

Now install a big one that requires mpi.   Along the way this will install openmpi.
```bash
spack install hdf5
```

If everything worked, you should be able to create and load the module:

```bash
spack module lmod refresh
module load openmpi
module load hdf5
module list
```

Note that you have to load openmpi before you load hdf5.  You can now try running some executables, e.g.

```bash
h5dump --version
mpiexec --version
```

And, you'll see that many other modules have been installed to support openmpi and hdf5.  This is what my module list currently looks like:

```bash
~:$ module list
Currently Loaded Modules:
  1) glibc/2.39-jozorw3          12) gettext/0.21-exbhth7
  2) gcc-runtime/13.3.0-oaqv5fr  13) krb5/1.21.3-ptjmd5k
  3) libpciaccess/0.17-sxmfrgq   14) libedit/3.1-20240808-4phylvh
  4) xz/5.6.3-lqvjjvh            15) libxcrypt/4.4.38-ghfq6yw
  5) zlib-ng/2.2.3-abnf734       16) openssh/9.9p1-cfddelo
  6) libxml2/2.13.5-soh5th2      17) pmix/5.0.5-aqlo6ew
  7) ncurses/6.5-6wol2w4         18) openmpi/5.0.6-qgnmtub
  8) hwloc/2.11.1-f6iwe64        19) pkg-config/0.29.2-gilgf2p
  9) openssl/3.4.1-5xhujym       20) hdf5/1.14.3-efc3vfk
 10) libevent/2.1.12-tgtj7sp     21) expat/2.7.0-4kdu7sw
 11) numactl/2.0.18-hes3cce      22) udunits/2.2.28-6cfz7zg
```

Continue to install more:

```bash
spack install openblas
spack install boost
spack install eigen
spack install netcdf-c
spack install netcdf-cxx
spack install netcdf-fortran
spack install nccmp
spack install fftw
spack install qhull
spack install cgal
spack install mkl
spack install nlohmann-json
spack install nlohmann-json-schema-validator
spack install gsibec
spack install ecbuild
spack install eckit
spack install fckit
spack install ectrans
spack install ecmwf-atlas
spack install odc
spack install eccodes
spack install jedi-cmake
```

I don't think this is required so if it doesn't work don't worry about it.  But worth a try
```bash
spack install nco
```

Don't forget to run this to generate the lmod modules after you install any spack packages (you can do this as many times as you like - you do not have to do it after each install)

```bash
spack module lmod refresh
```

This should be sufficient for ufo and cam-jedi development work.  If time goes on and you find the need to install more things for other components (like ewok and r2d2), then you can always keep addint to your "jedi stack".

Also, you can maintain more than one jedi stack with more than one compiler if you just follow the procedure above for your compiler of choice (e.g. intel or clang instead of gcc).

#### Create a jedi meta-module

A meta-module is a module that loads other modules.  

You can create your own meta-modules as follows.

First create a directory to put them in, for example:

```bash 
mkdir -p ~/modules/jedi
cd ~/modules/jedi
```

Now edit a file called `gcc.lua` and copy and paste the following contents into it:

```lua
help([[
Load environment for running JEDI applications with GNU compilers and OpenMPI.
]])

local pkgName    = myModuleName()
local pkgVersion = myModuleVersion()
local pkgNameVer = myModuleFullName()

conflict(pkgName)

load("cmake")
load("openmpi")
load("udunits")
load("openblas")
load("boost")
load("eigen")
load("hdf5")
load("netcdf-c")
load("netcdf-cxx")
load("netcdf-fortran")
load("nccmp")
load("qhull")
load("cgal")
load("fftw")
load("intel-oneapi-mkl")
load("nco")
load("nlohmann-json")
load("nlohmann-json-schema-validator")
load("gsibec")

load("ecbuild")
load("eckit")
load("fckit")
load("ecmwf-atlas")
load("odc")
load("eccodes")
load("jedi-cmake")

whatis("Name: ".. pkgName)
whatis("Version: ".. pkgVersion)
whatis("Category: Application")
whatis("Description: JEDI Environment with OpenMPI")
```

Now to let lmod know where the parent directory is (above the jedi directory), put this in your `~/.bashrc` file:

```bash
module use $HOME/modules
```

Try it to see if it works

```bash
source ~/.bashrc
module purge
module list
module load jedi
module list
```

The first module list should produce no items (after the purge).  And the second should load all the modules you had before.  So, in my case, the `module load jedi` command loaded 62 modules.

Now, whenever you want to build jedi, you just have to enter `module load jedi` to have all your dependencies ready to go.

Now, to build jedi, [follow the JCSDA instructions](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/using/building_and_running/building_jedi.html) or the [README.md](README.md) file.

#### Spack commands: a brief reference

A few useful spack commands - see the documentation for full details

To see what packages you have already installed, enter:
```bash
spack find
```

If you want to uninstall everything and start over, enter:

```bash
spack uninstall -ay
```

To completely wipe the modules and restart with that, enter

```bash
spack module lmod refresh --delete-tree -y
```
