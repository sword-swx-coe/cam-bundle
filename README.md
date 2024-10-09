# cam-bundle

## Description

Bundle containing all the repositories needed to build the JEDI interface for NCAR's Community Atmosphere Model (CAM). It has initially been developed for the WACCM-X configuration of CAM but it should be straightforward to generalize it to other configurations.

## Installation

To build the CAM bundle, the first step is to clone the bundle:

```bash
git clone https://github.com/sword-swx-coe/cam-bundle
cd cam-bundle
```

The next thing to do is to create a build directory. This should not be in the cam-bundle directory. Often it’s good to have it at the same level as the cam-bundle directory. So, from the top of the jedi-bundle directory you can do the following:

```bash
mkdir ../build
cd ../build
```

Now, if you are building on NCAR's Derecho platform, you can load the environment modules that are maintained there and that include all of the software dependencies needed to build JEDI:

```bash
source ../cam-bundle/env-setup/gnu-derecho.sh
```

This is assuming your default shell is bash. If you use tcsh, then you would source the gnu-derecho.csh script instead. We have borrowed these scripts and modules from the [MPAS team at NCAR](https://github.com/JCSDA/mpas-bundle), who run JEDI on Derecho regularly. You can read the README file in that MPAS-bundle link for more tips on how to build and test a jedi bundle on Derecho.

You have source this environment file every time you build and/or run JEDI.

However, you only have to do the following command once, the first time you build JEDI:

```bash
git lfs install
```

This initializes GitHub's Large File Service (LFS) which is used to store many of data files that JEDI uses to run the unit tests.

Now, from the build directory, enter:

```bash
ecbuild ../cam-bundle
make update
```

The ecbuild command configures the code for the build using cmake and ecbuild. If the repos are not already there, ecbuild will clone them from Github: after running this you can look in the cam-bundle directory and see all the repos that it downloaded.

While grabbing the code repositories, ecbuild also downloads much of the test data. When I first tried running this command on a compute node on Derecho in combination with the make command below, the ecbuild command alone took well over an hour. I suspect that the network bandwidth to the compute nodes is slower than it is to the login nodes. On a login node, the ecbuild command took 10 minutes.

The make update command is not strictly needed for your first build but it is good to get in the habit of running this after ecbuild and before make. It will grab all the latest updates in the develop branches on github.

It is recommended that you do not use the login node to compile the JEDI code on Derecho. Instead, you can start an interactive compute node with the following command:

```bash
qsub -X -I -l select=1:ncpus=14:mpiprocs=14 -l walltime=01:00:00 -q main -A <project-number>
```

This requests 14 cores for one hour. Then, after this boots up, you can build the bundle as follows:

```bash
source ../cam-bundle/env-setup/gnu-derecho.sh
make -j8
ctest
```

Note that sourcing the environment script is necessary every time you enter a new environment, in this case the compute node. The `-j8` option tells make to do a parallel build with 8 cores. But it still takes a while.

If you want to run the tests only for a particular repository you can run, for example, `ctest -R ufo`. The `-R` option to ctest selects any test name that contains the string that is passed. For more tips on running ctest with the various options (such as verbose and viewing the log) see the [jedi documentation](https://jointcenterforsatellitedataassimilation-jedi-docs.readthedocs-hosted.com/en/latest/inside/testing/unit_testing.html#running-ctest).

For further details about JEDI, including installation instructions see: http://jedi-docs.jcsda.org/
