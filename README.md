# Red Hat Advanced Cluster Management with Multiarchitecture OpenShift

Red Hat Advanced Cluster Mangement (RHACM) provides the tools and capabilities to address various challenges with managing multiple clusters and consoles, distributed business applications, and inconsistent security controls across Kubernetes clusters that are deployed on-premises, or across public clouds.

The demonstration in this respository focuses on **consistent application and infrastructure as code deployments across multiarchitecture Kubernetes clusters**. However, RHACM provides numerous other capabilities related to cluster creation and updating, Ansible automation, and governance that will not be covered in this demonstration.

## Table of Contents

- [Red Hat Advanced Cluster Management with Multiarchitecture OpenShift](#red-hat-advanced-cluster-management-with-multiarchitecture-openshift)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Background](#background)
  - [Installing RHACM](#installing-rhacm)
  - [Importing OpenShift Clusters into RHACM](#importing-openshift-clusters-into-rhacm)
  - [Deploying Applications across Multiarchitecture OpenShift Clusters](#deploying-applications-across-multiarchitecture-openshift-clusters)
  - [Deploying Infrastructure-as-a-Service across Multiarchitecture OpenShift Clusters](#deploying-infrastructure-as-a-service-across-multiarchitecture-openshift-clusters)
  - [Wrap Up](#wrap-up)
  - [More Resources](#more-resources)

## Prerequisites

1. OpenShift on IBM zSystems or LinuxONE
2. OpenShift on x86-based infrastructure (on premises or in a public cloud)
   1. Alternatively, any platform listed as supported [here](https://access.redhat.com/articles/6968787)
3. Meet the network connectivity requirements explained [here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html/networking/index#doc-wrapper).
4. Cluster administrator authority for both OpenShift clusters
5. Basic OpenShift knowledge - how to log in to the console and CLI, navigation and use of each 
   
## Background

As IT organizations adopt the hybrid multicloud model, Kubernetes clusters have a tendency to proliferate across IT environments, becoming more difficult to manage and maintain in a consistent manner. It's no small task for cloud administrators to keep track of which clusters are running certain applications. For example, if there is a business critical application that is required to run on both two x86 OpenShift production clusters in different regions, as well as an OpenShift cluster running on IBM zSystems, they would need to go on three different OpenShift clusters (likely owned by separate teams) and manually check that all of the application components are running and in good health. Then whenever that application needs to be updated, they need to again connect to three different OpenShift clusters, remove the existing applications, deploy the updated applications, make sure everything works, and manually check that all three applications are deployed the same, or create homegrown automation to do this for them. This might be feasible for three OpenShift clusters, but what if you have teams spinning up new Kubernetes clusters every day in public clouds? What if your organization starts working with Internet of Things (IoT) devices that each has their own mini Kubernetes cluster on each device? How many clusters and you manually maintain - a dozen? One hundred? 

**There comes a point in hybrid multicloud environments where cloud administrators needs some automation and a single source of truth for application management and maintenance.**

![clusters-vs-difficulty](clusters-vs-difficulty.drawio.png)

Another scenario - forget the applications... how can the cloud administrator make sure that all three (dozen? (hundred? (thousand??))) clusters themselves are consistent? Are they expected to manually configure each cluster for persistent storage, required operators, LDAP integrations, etcd encryption, role-based access control, and the hundred other customizations performed when a new cluster is created? And again, what if something changes? What if a new version of an operator is released and it needs to be installed on each cluster? What if someone leaves the company and their access needs revoked from every cluster?

**As with applications running in hybrid cloud environments, the Kubernetes clusters themselves require a standardized and automated model to maintain consistency.**

![cluster-consistency](cluster-consistency.drawio.png)

## Installing RHACM

## Importing OpenShift Clusters into RHACM

## Deploying Applications across Multiarchitecture OpenShift Clusters

## Deploying Infrastructure-as-a-Service across Multiarchitecture OpenShift Clusters

## Wrap Up

## More Resources