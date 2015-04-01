# Using software modules on updated nodes
March 31st, 2015

==============

## 1. Introduction

The recent upgrade to Wheezy on some the nodes of the Gaia cluster comes with changes to the software module architecture too. In this document, we will explain what are these changes and what it means when you want to use the cluster.

Along with this upgrade, we implement a new software module architecture that allows for better presentation of the software stack as well as better software maintenance. These improvements are part of the RESIF project that automatizes the management of the software modules on the clusters.

## 2. How to adapt your workflow

Part of your workflow requires simple and little modification to work on this new environment.

The most noticeable change is the new names for the software modules. Names now follow a Hierarchical naming scheme that gives more and clearer information on the software than before, the format is the following:  
**software_class/software_name/software_complete_version**  
where:  
- **software_class** describes the category among the following classes: [base, bio, cae, chem, compiler, data, debugger, devel, lang, lib, math, mpi, numlib, phys, system, toolchain, tools, vis]
- **software_name** is the name of the software (e.g. GROMACS, MATLAB, R or ABySS)
- **software_complete_version** is the full version of the software: containing the version of the software itself followed by the type and version of the main dependencies it relies on (e.g. compiler) with the following format: software_version-dependencies_versions

We took this upgrade as an opportunity to update some of the provided software too, some of the software you used may now be available in a newer version than before.

To check what is the name of a software after the upgrade, use the `module avail` command. As an example, to find the name of the module to load if you want to know the module name for GROMACS, procede as follows:

    (node)$> module avail GROMACS
    ----------- /opt/apps/resif/devel/v0.9-20150310/core/modules/bio -----------
    bio/GROMACS/4.6.1-ictce-5.3.0-hybrid    bio/GROMACS/4.6.1-ictce-5.3.0-mt
    bio/GROMACS/4.6.5-goolf-1.4.10-hybrid    bio/GROMACS/4.6.5-goolf-1.4.10-mt (D)
And then simply load the module you want to use:

    (node)$> module load bio/GROMACS/4.6.1-ictce-5.3.0-hybrid

Another change coming with this upgrade is the `module` tool used on the cluster: Lmod now replaces environment-modules. Lmod allows for better performance and implements more functionalities that you can discover on its [official website](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod), but your usual workflow will work the same as before. The only major difference is that the standard way to modify the modulepath environment variable is no longer to modify directly the `MODULEPATH` environment variable, but to use the `module use` and `module unuse` commands. Here is an example detailing the old and the new way of adding the `/tmp/my_modules` path to the modulepath:

- Old method:

        (node)$> export MODULEPATH=/tmp/my_modules:$MODULEPATH
- New method:

        (node)$> module use /tmp/my_modules

This is all you need to adapt in your workflow.

## 3. How to add a missing software

To install a new software, you can use EasyBuild the same way as before after having loaded its module. How to use EasyBuild is not covered here, to learn more about it, please refer to its [documentation](http://easybuild.readthedocs.org/en/latest/).

You can also use RESIF which is the method that we will cover here.

### Installation of RESIF

RESIF requires the following prerequisites (already available on the platform):

- [Python 2.6](https://www.python.org/downloads) or above
- pip (included in the latest Python)
- [git](http://git-scm.com/downloads)

Install RESIF:

        (node)$> pip install --install-option="--prefix=$HOME/.local" resif

Add the following paths to the environment:

        (node)$> export PATH=$PATH:~/local/bin
        (node)$> export PYTHONPATH=$PYTHONPATH:~/local/lib/python2.6/site-packages

Initiliaze RESIF, this will download the required module sources to build new software:

        (node)$> resif init

### Installation of additional software

First you need to create a file that lists the applications you want to install.  
To search for the names of the application configuration files, use RESIF as follow:

        (node)$> resif search bzip2
        [...]
        * bzip2-1.0.6.eb
        [...]
Create a file (we assume it is in the home directory), name it `swsets.yaml` and insert the following content:

        mysoftware:
          - bzip2-1.0.6.eb

This is a [YAML](http://yaml.org/) format file that RESIF reads, the internal layout is the following:

        software_set1_name:
          - software1_configurationfile
          - software2_configurationfile
        software_set2_name:
          - software3_configurationfile
It can list as many software, divided in as many software sets as you require to structure our installation. A software set is basically a collection of software, to learn more about it, please refer to the [EasyBuild documentation](http://resif-pypi.readthedocs.org/en/latest/).

Now install the software using `resif build`:

        (node)$> resif build --installdir ~/.local/resif --swsets-config ~/swsets.yaml mysoftware
This will install the software using ~/.local/resif as the root of the installation.

To make the software modules available through the `module` command, you need to add their path:

        (node)$> module use ~/.local/resif/mysoftware/modules/all

Now, you can see `bzip2` at the very beginning of the output of the list of the software modules:

        (node)$> module avail
        ----- /home/users/username/.local/resif/mysoftware/core/modules/all -----
        tools/bzip2/1.0.6

The software is installed, and you can load its profile with `module load tools/bzip2/1.0.6`.

RESIF offers many more possibilities than this basic functionality, for more details check the [corresponding tutorial](https://github.com/ULHPC/tutorials/tree/devel/advanced/RESIF) or the [documentation](http://resif-pypi.readthedocs.org/en/latest/).