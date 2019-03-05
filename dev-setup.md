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

- facet
  - Provides a UI and RESTful API for our services
   - Day 1 Provisioning API
   - Uses an embedded HTTP server to serve Day 1 UI, which will be the primary consumer of this API
   - Provisioning host configuration validation at startup
  - Central integration point for doing MetalKube deployments of OpenShift
  - Facet Architecture











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