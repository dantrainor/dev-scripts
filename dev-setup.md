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

An OpenShift pull secret is used to pull OpenShift images from the cloud.  To generate a pull secret, visit the Red Hat OKD (OpenShift 4) Cloud site (Red Hat SSO required).   Click on the “Copy Pull Secret” button towards the bottom of the page, to copy the pull secret in to your clipboard.

The pull secret is stored in an environment file.  An example file exists with the name of config_example.sh.  Copy this file in to one that matches config\_$USER.sh:
```
$ cp config_example.sh config_${USER}.sh
```











[Foo](#foo)


















- [Heading](#heading-2)
  * [Sub-heading](#sub-heading-2)
    + [Sub-sub-heading](#sub-sub-heading-2)


# Heading levels

> This is a fixture to test heading levels

<!-- toc -->

## Heading

This is an h1 heading

### Sub-heading

This is an h2 heading

#### Sub-sub-heading

This is an h3 heading

## Heading

This is an h1 heading

### Sub-heading

This is an h2 heading

#### Sub-sub-heading

This is an h3 heading

## Heading

This is an h1 heading

### Sub-heading

This is an h2 heading

#### Sub-sub-heading

This is an h3 heading
