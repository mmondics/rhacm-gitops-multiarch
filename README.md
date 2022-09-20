# Red Hat Advanced Cluster Management with Multiarchitecture OpenShift

Red Hat Advanced Cluster Mangement (RHACM) provides the tools and capabilities to address various challenges with managing multiple clusters and consoles, distributed business applications, and inconsistent security controls across Kubernetes clusters that are deployed on-premises, or across public clouds.

The demonstration in this respository focuses on **consistent application and infrastructure as code deployments across multiarchitecture Kubernetes clusters**. However, RHACM provides numerous other capabilities related to cluster creation and updating, Ansible automation, and governance that will not be covered in this demonstration.

## Table of Contents

- [Red Hat Advanced Cluster Management with Multiarchitecture OpenShift](#red-hat-advanced-cluster-management-with-multiarchitecture-openshift)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Background](#background)
  - [Understanding RHACM](#understanding-rhacm)
  - [Installing RHACM](#installing-rhacm)
  - [Importing OpenShift Clusters into RHACM](#importing-openshift-clusters-into-rhacm)
  - [Exploring RHACM](#exploring-rhacm)
  - [Understanding RHACM Applications](#understanding-rhacm-applications)
  - [Deploying Applications across Multiarchitecture OpenShift Clusters](#deploying-applications-across-multiarchitecture-openshift-clusters)
  - [Deploying Infrastructure-as-a-Service across Multiarchitecture OpenShift Clusters](#deploying-infrastructure-as-a-service-across-multiarchitecture-openshift-clusters)
  - [Wrap Up](#wrap-up)
  - [More Resources](#more-resources)

## Prerequisites

1. `oc` and `kubectl` CLIs installed. 
2. OpenShift cluster on IBM zSystems or LinuxONE
3. OpenShift cluster on x86-based infrastructure (on premises or in a public cloud)
   1. Alternatively, any platform listed as supported [here](https://access.redhat.com/articles/6968787)
4. Meet the network connectivity requirements explained [here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html/networking/index#doc-wrapper).
5. Cluster administrator authority for both OpenShift clusters
6. Basic OpenShift knowledge - how to log in to the console and CLI, navigation and use of each 
   
## Background

As IT organizations adopt the hybrid multicloud model, Kubernetes clusters have a tendency to proliferate across IT environments, becoming more difficult to manage and maintain in a consistent manner. It's no small task for cloud administrators to keep track of which clusters are running certain applications. For example, if there is a business critical application that is required to run on both two x86 OpenShift production clusters in different regions, as well as an OpenShift cluster running on IBM zSystems, they would need to go on three different OpenShift clusters (likely owned by separate teams) and manually check that all of the application components are running and in good health. Then whenever that application needs to be updated, they need to again connect to three different OpenShift clusters, remove the existing applications, deploy the updated applications, make sure everything works, and manually check that all three applications are deployed the same, or create homegrown automation to do this for them. This might be feasible for three OpenShift clusters, but what if you have teams spinning up new Kubernetes clusters every day in public clouds? What if your organization starts working with Internet of Things (IoT) devices that each has their own mini Kubernetes cluster on each device? How many clusters can you manually maintain - a dozen? One hundred? 

**There comes a point in hybrid multicloud environments where cloud administrators needs some automation and a single source of truth for application management and maintenance.**

![clusters-vs-difficulty](clusters-vs-difficulty.drawio.png)

Another scenario - forget the applications... how can the cloud administrator make sure that all three (dozen? (hundred? (thousand??))) clusters themselves are consistent? Are they expected to manually configure each cluster for persistent storage, required operators, LDAP integrations, etcd encryption, role-based access control, and the hundred other customizations performed when a new cluster is created? And again, what if something changes? What if a new version of an operator is released and it needs to be installed on each cluster? What if someone leaves the company and their access needs revoked from every cluster?

**As with applications running in hybrid cloud environments, the Kubernetes clusters themselves require a standardized and automated model to maintain consistency.**

![cluster-consistency](cluster-consistency.drawio.png)

## Understanding RHACM

RHACM can help with both of the scenarios described above. Cluster administrators or Site Reliability Engineers (SREs) can manage both *applications* and Kubernetes *infrastructure* from a single pane of glass. RHACM supports GitOps methodology where a Git repository (such as GitHub) acts as the single source of truth for application and infrastructure code, then RHACM deploys the contents of the Git repository to desired clusters through a system of label matching. 

![rhacm-drawio](rhacm.drawio(2)(1).png)

*You may need to open this full image in a new tab to see it better.*

After the application or infrastructure is deployed to the Kubernetes clusters, RHACM then constantly watches both the deployed application/infrastructure as well as the Git repository "source of truth". So whenever the code is updated in the Git repository, RHACM will automatically push those updates into the Kubernetes clusters. The reverse scenario is also true - if an application or infrastructure on Kubernetes changes, RHACM will automatically apply the Git repository to bring the applications/infrastructure back in sync.

Other than the obvious benefits of automation, speed, and consistency, one of the less-frequently discussed benefits of this GitOps methodology is the reduction of Kubernetes access and skills required by developers. With RHACM and GitOps, developers only have to worry about development and do not need to be given access to OpenShift or need to know how to deploy an application on OpenShift. They just have to commit and push updates to Git the same way they always have, and their application will automatically propogate to all desired OpenShift clusters in a consistent manner. No more looking up how to deploy an application, forgetting how to create a configMap, or needing to expose the application as a route with the proper hostname. That's all taken care of for you.

One thing to remember, Git is a source code management tool. Some of the benefits of Git are version control, visibility into who made which changes, and the ability to roll back to a previous version or point in time. Because Git is the source of truth that RHACM deploys from, the applications or infrastructure deployed by RHACM adopt these benefits as well. You can easily track who made changes and when they were made, and you can roll back those changes to a known state in Git if need be. With RHACM, this concept extends to entire OpenShift clusters. You can create, recreate, or recover clusters from a known state. This level of visibility and control over as many Kubernetes clusters as you wish was unprecedented before GitOps.

## Installing RHACM

RHACM is a containerized application deployed as an operator on an OpenShift cluster, known as the *Hub cluster*. RHACM and this Hub cluster have visibility into, and control of, other Kubernetes clusters known as *Managed clusters*.

This demonstration assumes you will install RHACM on an OpenShift on x86 cluster. This is an arbitrary decision and if you would rather install RHACM on zSystems, that works just as well. (Or any of the other architectures listed in the "Supported for Hub" column [here](https://access.redhat.com/articles/6968787)).

1. From the Hub OpenShift cluster, navigate to the OperatorHub and search for Red Hat Advanced Cluster Management.

2. Click the Install button

3. Select your desired Update Channel. Typically the latest version is the best choice.

4. Leave the default settings for the remaining options and click Install.

    ![rhacm-install-options](https://raw.githubusercontent.com/mmondics/media/main/images/rhacm-install-options.png)

5. Once the operator is installed, click the Create MultiClusterHub button.

    If you have navigated away from the installation page, you can find this button by navigating in the OCP console to Administrator -> Operators -> Installed Operators -> Advanced Cluster Management for Kubernetes.

1. Leave the default options and click the Create button.

2. After installing the MultiClusterHub, access the RHACM console from the OpenShift console by clicking on the Administrator dropdown and changing to Advanced Cluster Management.

    ![access-rhacm](https://raw.githubusercontent.com/mmondics/media/main/images/access-rhacm.png)

1. Log in with your OpenShift credentials.

  You now have RHACM installed on the x86 Hub cluster. The next step is to configure RHACM to manage an OpenShift cluster on IBM zSystems.

## Importing OpenShift Clusters into RHACM

1. In the RHACM console, navigate to Infrastructure -> Clusters.

    RHACM is capable of both *creating new OpenShift clusters*, and *importing existing OpenShift clusters*. In this case, because [you already have](#prerequisites) an existing OpenShift on IBM zSystems cluster, you will be importing it.

1. Click the Import Cluster button.

1. Provide a distinguishable name for the managed IBM zSystems cluster. 

    For example, because the IBM zSystems cluster used when developing this demo is named `atsocpd2`, a distinguishable name would be `atsocpd2-s390x`.

1. Next, select Run import commands manually for the Import mode.

    There are other ways to import a cluster as well, with a server URL and token or a Kubeconfig, but the end result will be the same.

1. Click the next button.

1. Skip the automation page. 

    As mentioned in a previous section, RHACM supports the use of Ansible playbooks to run at different stages of the cluster lifecycle. This integration allows for customization or remediation of OpenShift clusters using Ansible playbooks. This is outside the scope of this demonstration, however cool it is.

1. On the Review page, click the Generate command button.

    You will be taken to a new page for the yet-to-be imported OpenShift on IBM zSystems cluster. 

1. Log in to your managed OpenShift on IBM zSystems cluster using the `oc` command line. 

1. Click the Copy command button from the RHACM console, and paste it into your CLI session where you are logged into OpenShift.

    This will paste a *quite* long command made up of base64 encoded information.

    After the commmand runs, you will see various objects created in the managed cluster:

    ```text
    customresourcedefinition.apiextensions.k8s.io/klusterlets.operator.open-cluster-management.io created
    namespace/open-cluster-management-agent created
    serviceaccount/klusterlet created
    clusterrole.rbac.authorization.k8s.io/klusterlet configured
    clusterrole.rbac.authorization.k8s.io/open-cluster-management:klusterlet-admin-aggregate-clusterrole unchanged
    clusterrolebinding.rbac.authorization.k8s.io/klusterlet unchanged
    deployment.apps/klusterlet created
    secret/bootstrap-hub-kubeconfig created
    klusterlet.operator.open-cluster-management.io/klusterlet created
    ```

    And after a minute or so, you will see the IBM zSystems cluster information start to populate the RHACM console.

    ![rhacm-s390x-imported](https://raw.githubusercontent.com/mmondics/media/main/images/rhacm-s390x-imported.png)

    RHACM can now manage this OpenShift on IBM zSystems cluster by updating it, hibernating nodes, deploying applications to it, deploying infrastructure or configuration changes to it, enforcing govenance rules and all of the other great capabilities provided by RHACM.

## Exploring RHACM

Before using RHACM to deploy an application to the managed OpenShift cluster, you should get more familiar with navigating the RHACM console. 

1. Use the left-side menu to go back to the overall Clusters page.

    ![rhacm-clusters](https://raw.githubusercontent.com/mmondics/media/main/images/rhacm-clusters.png)

    Notice on this page that RHACM automatically identified the Status, Infrastructure, Version, and other information about the two clusters that it stored as labels. One thing to note - even though RHACM is running on an x86 architecture (VMware vSphere) cluster, it will have no problem managing and deploying to the IBM zSystems cluster.

    The `local-cluster` listed on this page is the Hub cluster where RHACM is running. So you have a situation where RHACM can manage and deploy the OpenShift cluster that it is running on. 

    From this page you can upgrade or hibernate/resume clusters. Updating clusters from RHACM makes it easier to be sure that clusters are all at the correct versions without having to log into each one individually. Hibernating clusters can be a good idea, especially for public-cloud based clusters, because it will temporarily shut down the cluster nodes when they arent needed, saving an organization money.

    From this page, you can also put clusters into Cluster Sets or Cluster Pools. 

    - Cluster Sets essentially group cluster resources together so that you can manage all of their resources with one set of role-based access controls. This can be useful for teams who need to use multiple clusters as if they were one single entity, and also allows for inter-cluster network connectivity with [Submariner](https://submariner.io/).

    - Cluster Pools are pools of multiple clusters that can be made available for checkout by developers or for use by temporary workloads. When the clusters aren't checked out, they are in a hibernating state so as to reduce cloud provider cost.

1. Navigate to the Overview page using the left-side menu.

    On the Overview page, you can see overall information for all of the managed (and Hub) clusters. Although there are only two clusters involved for this demonstration, you can imagine how helpful it would be to find information about cluster fleet health and status if there were dozens, hundreds, or thousands of clusters being managed.

    ![rhacm-overview](https://raw.githubusercontent.com/mmondics/media/main/images/rhacm-overview.png)

    You might notice that RHACM has found zero applications in the two clusters. The reason for this is that an Application is a concept specific to RHACM - this is not an application in the sense of a OpenShift deployment or pod running on a cluster. 

    You will learn more about RHACM Applications (and deploy one) in the next section.

## Understanding RHACM Applications

RHACM uses a combination of Kubernetes objects and its [own specific components](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html/about/welcome-to-red-hat-advanced-cluster-management-for-kubernetes#red-hat-advanced-cluster-management-for-kubernetes-terms) related to multicluster management. Just like Kubernetes objects, RHACM objects are defined in [YAML](https://www.redhat.com/en/topics/automation/what-is-yaml).

For the next few explanations of terms, refer to the image below (from the [RHACM documentation](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/applications/index#application-model-and-definitions).) You can also find more details about the RHACM application model at the same link.

![rhacm-application-model](https://raw.githubusercontent.com/mmondics/media/main/images/rhacm-application-model.jpg)

When deploying infrastructure-as-code or containerized Kubernetes applications with RHACM, the highest level resource is an **Application** (see how this can be confusing?). *A RHACM Application is used to group Kubernetes objects that make up an applicatio*n. RHACM can also discover Applications installed on managed OpenShift clusters that were deployed with the Red Hat GitOps or ArgoCD operators. In their YAML definitions, Applications refer to our next term, Subscriptions.

**Subscriptions** are the RHACM object that allow clusters to *subscribe* to source repositories such as Git, Helm, or Object storage repositories. For this, Subscriptions refer to Channels.

*Note*: This demonstration focuses on the GitOps approach to mutlicluster management, so Helm and Object storage are outside of the current scope. If you would like to learn more about them, you can read more [here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/applications/index#managing-apps-with-helm-cluster-repositories) and [here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/applications/index#managing-apps-with-object-storage-repositories).

Subscriptions have the added responsibility of managing *where* the source repository objects will be deployed. For this, subscriptions refer to Placement Rules.

**Channels** represent the Git repository that is the single "Source of Truth" discussed earlier in this demonstration. Supported Git repositories include [GitHub](https://github.com/), [GitLab](https://about.gitlab.com/), [Bitbucket](https://bitbucket.org/), and [Gogs](https://gogs.io/). You can also use private/entitled Git repositories in a seucre way by storing access credentials in a Kubernetes secret. 

**Placement Rules** are used by RHACM to define which target clusters an Application will deploy to. This can be defined by specifying labels or Cluster Sets. 

Now that you know RHACM Applications and their components, next you will create one from the RHACM console.

## Deploying Applications across Multiarchitecture OpenShift Clusters

## Deploying Infrastructure-as-a-Service across Multiarchitecture OpenShift Clusters

## Wrap Up

## More Resources