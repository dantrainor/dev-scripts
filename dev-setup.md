Setup Development Environment
=============================

This document describes how to get started with a usable, functional, stable  development environment to ultimately develop against RHHI.Next.

- [Components](#components)
  * [dev-scripts](#components-dev-scripts)
  * [facet](#components-facet)
- [Environment Prerequisites](#environment-prerequisites)
  * [Host System](#prerequisites-host-setup)
    + [Additional Configuration](#prerequisites-additional-configuration)
    + [Software Prerequisites](#prerequisites-software)
    + [Clone the dev-scripts repository](#clone) 
- [Development Environment Installation with dev-scripts](#dev-scripts)
  * [Overview](#dev-scripts-overview)
  * [01\_install|_requirements.sh](#01sh)
  * [02\_configure\_host.sh](#02sh)
  * [03\_ocp_repo_sync.sh](#03sh)
  * [04\_setup_ironic.sh](#04sh)
  * [05\_build\_ocp\_installer.sh](#05sh)
  * [06\_deploy\_bootstrap\_vm.sh](#06sh)
  * [07\_deploy\_masters.sh](#07sh)

- [Cleaning up your workspace with dev-scripts](#cleanup)
- [Developing against facet](#facet)
- [Q/A](#qa)
  * [How do I make facet listen on an IP / host other than ‘localhost’?](#qlocalhost)
  * [How do I connect to the masters?](#qmasters)
  * [ How about connecting to the bootstrap node?](#qbootstrap)
  * [What about the bridges?](#qbridges)
  * [I’ve heard “production api” or “production build” - what’s that?](#qprod)

# Components

dev-scripts
- Set of scripts to configure some libvirt VMs and associated virtualbmc processes to enable deploying to them as dummy baremetal nodes
- Launches openshift-installer, creating a (by default) 3-node KNI infrastructure, with a bootstrap node
- Uses (by default) the libvirt backend
- Creates a Bootstrap node, responsible for creating an OpenShift cluster
- Creates an OpenShift cluster comprised of virtual machines, on your dedicated development host

facet
- Provides a UI and RESTful API for our services
  - Day 1 Provisioning API
  - Uses an embedded HTTP server to serve Day 1 UI, which will be the primary consumer of this API
  - Provisioning host configuration validation at startup
  - Launch the Ironic containers using podman on the provisioning host
  - Downloads the current images of RHCOS that are needed for a deployment (bootstrap VM, baremetal hosts)
  - Runs installer, launch bootstrap VM, and drive Ironic APIs
- Central integration point for doing MetalKube deployments of OpenShift
- Facet Architecture

# Environmental Prerequisites

You will need a dedicated hardware host to be effective with your development tasks.  This is due to the fact that your deployment 
environment will host several virtual machines using libvirt.  These all get created as part of the execution of dev-scripts.  
Later versions of this guide may address passthrough virtualization which allows for development in a cloud space.

## Host System

- Hardware host with at least 64G of RAM
- Dedicated CentOS 7 System
  - Fully updated
- Either console access, or ability to reload the system with a tool such as Beaker
  - Failure is unexpected, but not unheard of, when iterating through development so quickly as RHHI.Next grows.  Having a backup plan is the safest bet

## Additional Configuration

Ensure that the username you are developing with, has passwordless sudo access.  This can be any username.  This example uses the username ‘rhhi’:
```
# useradd rhhi
# echo "rhhi ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/rhhi
# chmod 0440 /etc/sudoers.d/rhhi
```

## Software Prerequisites

Luckily, dev-scripts handles much of the prerequisite software installation heavy lifting.  

### Clone the dev-scripts repository

As the rhhi user, clone the dev-scripts repository to a location of your choosing:

```
$ git clone git@github.com:metalkube/dev-scripts.git
```

Change in to the dev-scripts directory:
```
$ cd dev-scripts
```

An OpenShift pull secret is used to pull OpenShift images from the cloud.  To generate a pull secret, visit the Red Hat OKD (OpenShift 4) Cloud 
site (Red Hat SSO required).   Click on the “Copy Pull Secret” button towards the bottom of the page, to copy the pull secret in to your clipboard.

The pull secret is stored in an environment file.  An example file exists with the name of config\_example.sh.  Copy this file in to one that matches config\_$USER.sh:
```
$ cp config_example.sh config_${USER}.sh
```

If you wish to use an environment file that specifies the pull secret by any other name, you’ll need to prefix each of your ran 
commands with CONFIG=<environment file>.sh.  However by default, the build scripts will search for a file named config\_$USER.sh, 
which will save you some time.

Edit this file, and replace the value of the PULL\_SECRET with that of what’s copied in your clipboard, which should be your individual pull secret.  
Remember to respect the single quotes already present in the file.


# Development Environment Installation with dev-scripts

For more detailed information, consult the dev-scripts git repository.  Jumping right in though, you should expect to be able to set up an almost 
complete development environment by running the following command:
```
$ make
```

‘make’ looks for a Makefile, inside of which is a canonical list of steps.  These steps execute the following dev-scripts in order:
- 01\_install\_requirements.sh
  - Responsible for installing software prerequisites, adding additional repositories as necessary
  - Yarn, Node.js, golang, Ironic, and the OpenShift Origin Client Tools, and libvirt are all examples of software that gets installed
- 02\_configure\_host.sh
  - Mostly responsible for running tripleo-quickstart for the virtualized deployments, configuring libvirt, iptables, creating the baremetal and brovc bridges for system connectivity, and configuring NetworkManager for things such as DNS resolution of internal hostnames
- 03\_ocp\_repo\_sync.sh
  - Clones openshift-installer, facet, and statik from github.com
    - Additionally, rebases both openshift-installer and facet repos on to master
    - Checks out a branch called ‘we\_dont\_need\_no\_stinkin\_patches’
    - Additional information under the section “Developing against facet”
    - Uses yarn to install and build the production facet from source (a static golang binary)
- 04_setup_ironic.sh
  - Builds podman containers for Ironic and Ironic Inspector
  - Configures Ironic Inspector for pxebooting of virtualized systems
  - Adds an IP address to the brovc bridge
  - Massages the Red Hat CoreOS (RHCOS) image along with configuring interfaces inside of it, which nodes will run
- 05_build_ocp_installer.sh
  - Builds the OpenShift Installer
- 06_deploy_bootstrap_vm.sh
  - Uses openshift-install to create an ignition file, used to create a bootstrap VM.  The OpenShift architecture requries that a bootstrap VM is used to create a set of three OpenShift masters
  - Creates a guest of an appropriate size on the Host, and configures it appropriately
  - Attempts to connect to the bootstrap node to ensure that it’s been built and started properly
- 07_deploy_masters.sh
  - Uses the OpenStack python-tripleoclient to create baremetal nodes to act as the masters
  - A config drive is used by the nodes when they come up, which starts the process of installing the masters and ensuring that they can communicate with each other

Two additional scripts are included as part of dev-scripts.  They will rarely need to be invoked directly, but are used by ‘make clean’ to clean up the host system:
- ocp_cleanup.sh
  - Removes the bootstrap node along with its volumes, disk images, and associated ignition file
  - Removes any residual OpenShift configuration data (found in the ocp/ directory)
  - Removes the dnsmasq configuration created by 02_configure_host.sh

- host-cleanup.sh
  - Kills the Ironic podman containers running on the host
  - Uses an ansible playbook to tear down all the masters and workers


# Cleaning up your workspace with dev-scripts

Cleanup is relatively simple by invoking the same ‘make’ process we used for setting up the environment in the first place.

To clean up the entire environment, run:
```
$ make clean
```

# Developing against facet
Up until this point, we’ve just set up the Host, the Masters, three Workers, and cloned the necessary repositories.  This has not started facet.

Moving forward, the environment variable $GOPATH will be used to reference the Golang library path.  We can use the ‘go’ binary to create some environment variables in addition to $GOPATH:
```
$ eval `go env` 
```

The 03\_ocp\_repo\_sync.sh script we ran previously, automatically cloned the facet repo.  The repo now exists at $GOPATH/src/github.com/metalkube/facet.  This is necessary for the default facet production build, but needs additional work for active development.

To run the production build of facet, use the following command:
```
$ go run $GOPATH/src/github.com/metalkube/facet/main.go server
```

Upon execution, facet can be accessed by the URL printed to the output.

From here, facet development can be expanded by using the documentation, and standard development practices apply moving forward.


# Q/A

**Q**:  How do I make facet listen on an IP / host other than ‘localhost’?  
**A**:  It might be desirable to change the listening address of the Go server.  To do so, edit pkg/server/server.go around like 53, replacing the default ‘localhost’ with a different IP address, or 0.0.0.0 for all IPs.  

**Q**:  How do I connect to the masters?  
**A**:  Connecting to the masters shouldn’t be necessary, but in a pinch, you can access them by ssh core@master-<index>.api.ostest.test.metalkube.org  

**Q**:  How about connecting to the bootstrap node?  
**A**:  This might be more common.  You can access it by ssh core@osetest-bootstrap.api.ostest.test.metalkube.org  

**Q**:  What about the bridges?  
**A**:  There are two bridges in use by RHHI.Next:
- baremetal
  - Public API access
    - Primary interface on the Provision Host (also hosts the ostest-bootstrap VM) and Master nodes
- brovc
  - Provisioning
    - Secondary interface on the Provision Host (also hosts the ostest-bootstrap VM) and Master nodes  
To see these bridges and which interfaces are associated with them, run:
```
sudo brctl show
```













[Foo](#foo)
















