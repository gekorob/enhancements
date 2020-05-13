---
title: ovirt-rhv-upi
authors:
  - "@gekorob"
  - "@rolfedh"
reviewers:
  - "@sdodson"
  - "@abhinavdahiya"
approvers:
  - TBD
creation-date: 2020-05-06
last-updated: 2020-05-13
status: implementable
---

# oVirt-RHV UPI

## Release Signoff Checklist

- [x] Enhancement is `implementable`
- [x] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [x] Graduation criteria
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

This enhancement proposes adding tooling and documentation to help users deploy OpenShift 4 on existing _user-provisioned infrastructure_ (UPI) in oVirt.

## Motivation

Most users who want to deploy OpenShift on oVirt/RHV have a highly-customized infrastructure. We can help these users simplify the process of installing OpenShift on oVirt using existing user-provisioned infrastructure such as DNS, DHCP, Load Balancers.

Currently, users who want to deploy OpenShift on existing infrastructure in oVirt must adapt the [bare metal UPI instructions][baremetal-upi], which are not optimal for oVirt.

Using the current installation program for deploying OpenShift 4 on oVirt does not work because it creates an opinionated set of _installer-provisioned infrastructure_ (IPI). Though convenient, this IPI does not meet the needs of users who want to use _existing_ infrastructure.

### Goals

As a first step, we would create provisioning installer codebase and a set of documented steps. These would enable users to deploy OpenShift on UPI that is very similar to IPI.

This first step would include:

* Making oVirt-RHV UPI documentation available here:
https://github.com/openshift/installer/blob/master/docs/user/ovirt/install_upi.md
* Making Ansible playbooks for scripts for oVirt/RHV resource creation available here:
https://github.com/openshift/installer/tree/master/upi/ovirt
* Making a CI job to run the provisioning scripts and test the UPI installer.

### Non-Goals

It is outside the scope of this enhancement to provide explanations about the installation of infrastructure elements that are considered as required and owned by the user (e.g., DNS, DHCP, Load Balancer...).

## Proposal

* Write Ansible playbooks to automate, as much as possible, creating ovirt resources such as virtual machines for the master and worker nodes.
* Write Ansible scripts to configure pre-existing infrastructure elements, such as DNS and DHCP.
* Write the UPI documentation.
* Set up the CI job to run a test suite.


### Implementation Details/Notes/Constraints

The implementation of the UPI workflow aims to reproduce the features of the IPI that enable users to deploy OpenShift on existing infrastructure.

The user should be able to specify a custom configuration that uses the pre-existing infrastructure for the installation, such as the domain name, DNS, DHCP, load balancer, and Web server.

Given the maturity of the oVirt Ansible modules and administrators' familiarity with Ansible, we will provide a set of playbooks to automate the configuration and installation process as much as possible.

Depending on the user environment, we will provide scripts to help the user.

* Add mandatory records (A, PTR, SRV) to the user-provided DNS.
* Upload a specific RHCOS image.
* Configure an httpd server for RHCOS customized parameter ignition.
* Configure a load balancer (HAProxy).
* Configure a DHCP to assign IP addresses to bootstrap machines.
* Bootstrap VMs.

### Risks and Mitigations

This UPI will try to achieve the same results obtained by the automatic and opinionated IPI installer, but in a more customizable way using provisioning scripts like Ansible and the documentation provided.

The CI will represent one of the challenging problems. We will need to carefully schedule the UPI jobs to minimize their impact on the currently limited CI capacity.

We cannot exclude that an additional quota must be required to fulfill all the UPI CI needs.

## Design Details

### Test Plan

We will use existing UPI platforms, such as OpenStack, AWS, and GCP, as the inspiration for our testing strategy:

- A new e2e job will be created to use the Ansible templates.
- At the moment, we think unit tests will probably not be necessary. However, we will cover any required changes to the existing codebase with appropriate tests.

### Graduation Criteria

The proposal is to follow a graduation process based on the existence of a CI running suite with end-to-end jobs. We will evaluate its feedback along with feedback from QE's and testers.

We consider the following as part of the necessary steps:

- UPI document published in the OpenShift repo.
- Ansible playbooks exist.
- CI jobs present and regularly scheduled.
- End to end jobs are stable and passing and evaluated with the same criteria as the IPI.
- Developers of the team have successfully deployed a UPI on RHV following the
documented procedure.

## Implementation History

Significant milestones in the life cycle of a proposal should be tracked in `Implementation History`.

## Drawbacks

The UPI implementation is resource-intensive, not just from a development point of view, but also for CI, QE, Documentation, and others. Resources could be spent to improve the IPI process; however, as already specified in the `Motivation` section, support for UPI is highly requested by the users.

## Alternatives

People not using the IPI workflow can follow the Bare Metal UPI document. That implies more manual work and the necessary knowledge to identify oVirt/RHV specific parts without any automation help.

Extending the IPI to cover several common use cases coming from UPI users is not considered a valid option, because it's against the opinionated nature of the IPI itself.

## Infrastructure Needed

Developers have the infrastructure needed to develop the basic UPI features. Given limited CI resources, we need to carefully evaluate the impact of UPI jobs, future integration with storage, and other items.

[baremetal-upi]: https://github.com/openshift/installer/blob/master/docs/user/metal/install_upi.md
