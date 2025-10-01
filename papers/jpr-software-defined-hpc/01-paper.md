---
numbering:
  headings: true
  headings_1: false
title: Software Defined HPC with Open OnDemand
exports:
  - format: pdf
    template: arxiv_two_column
authors:
  - name: John-Paul Robinson
    affiliations:
      - University of Alabama at Birmingham, Research Computing
    orcid: 0009-0007-4063-2599
    email: jpr@uab.edu
  - name: Louis Chen
    affiliations:
      - University of Alabama at Birmingham, Research Computing
  - name: Eesaan Atluri
    affiliations:
      - University of Alabama at Birmingham, Research Computing
  - name: Krish Moodbidri
    affiliations:
      - University of Alabama at Birmingham, Research Computing
  - name: William Warriner
    affiliations:
      - University of Alabama at Birmingham, Research Computing
abstract: |
  Software defined infrastructure for high-performance computing (SDHPC) can reduce user impacts during maintenance operations.
  In our implementation, the complete system of users and resources was divided into two logical groups.
  Based on group membership, users are exposed to only the resources associated with their group.
  An infrastructure of application routers, deployed by continuous integration and continuous deployment (CI/CD) pipelines, route users to the appropriate gateways.

  We demonstrate the utility of this infrastructure to our motivating use case of limiting user downtime during maintenance operations. We conclude with observations on functionality and highlight future directions.

  Infrastructure has given us so much flexibility that...

keywords:
  - High Performance Computing
  - Software Defined
  - Infrastructure as Code
  - CICD
  - Cloud
  - AI Infrastructure
---

(introduction)=
# Introduction

The promise of scientific discovery drives the computing needs of researchers.
Advances in research computing technology arise in response to those needs.
Researchers must adapt to a rapidly-changing competative landscape.
As such, research computing staff must quickly adapt systems to changing needs and improving technology, while minimizing impact to researcher operations.
Traditional high-performance computing (HPC) infrastructure is defined by fixed hardware layouts.
In contrast, a software defined infrastructure for high-performance computing (SDHPC) is responsive and flexible, enabling in-flight upgrades with reduced researcher impact.

Traditional HPC use centered on text-based, command-line  interactions where users connect via SSH to dedicated login nodes to engage with research workflows @Lonvick2006.
The development of immersive web interfaces to HPC systems provided a friendlier user experience, increasing access.
Among these interfaces, Open OnDemand (OOD) emerged as the most successful @Hudak2018.
Now a fundamental component of many HPC systems, OOD brought user expectations for HPC systems in alignment with the self-directed experience of cloud-native applications.

OOD owes its success to a reduction of cognitive load for researchers.
The reference deployment of OOD provides access to several interactive, web-native applications.
Jupyter Notebooks, R Studio, and a VNC desktop capable of presenting native, windowed applications, are all available behind a simple resource request form.
Researchers start by engaging with their HPC system via the browser using familiar web applications and, when they are ready, can migrate to batch tools at the command-line as needed.
Implementation details, such as submitting a job script to the scheduler, are hidden behind the software-defined abstractions of the resource request form and application definition.
Because OOD hides these details, researchers can more readily envision HPC enabling their work.

OOD operates by mapping browser interactions to per-user web servers.
When a user connects to the HPC system via HTTPS, the OOD server spawns a web server process running under their account identity, as defined on the HPC system.
Whenever the user launches an application, processes started by their web server run within this identity context.
The net effect is that user engagement with the HPC system is shepherded by software definitions—identity, operating system rules, OOD application definitions, and the scheduler.
User interaction via OOD is governed by the same permissions enforced by operating system on all the user's processes, whether started via OOD or via a traditional command-line accessed via SSH.

NOTE: OOD solved a problem that let us do the thing. The "wart" in the OOD solution is the traditional SSH access. This is one of the problems we solved.

NOTE: On the IT side... OOD provides an improvement to application interface management. Our work augments this by providing an improvement to user access management.

<!--- ref gridsphere and science gateways that were dedicated tools and that tarun or something that we looked at around 2017 -->

OOD transforms HPC into a web-native user experience.
SDHPC transforms the HPC system into a cloud-native application.
Application routers for SSH and HTTPS enable us to modify the HPC environment for arbitrary subsets of the user community.
Identity-based routing requires the application routers to authenticate users in order to direct their connections to the correct resources.
Web authentication is handled by a web single sign-on (SSO) enabled Apache reverse-proxy that can route users based on group membership.
To handle SSH authentication and routing, we contributed an extension to sshpiper, an open-source SSH proxy built on top of the Golang SSH package, that enables routing based on group membership  @Lian2025.

<!--- we should show a picture of this here? -->

The application routers introduce an opportunity for horizontal scalability, further enabling SDHPC to deliver cloud-native infrastructure.
Continuous integration and continuous deployment (CICD) methodologies enable greater scaling of cloud-native infrastructures @Ugwueze2024.
To leverage the opportunity, we created a GitLab CI/CD workflow to build and deploy our application routers and OOD web services @gitlab-cicd.
Our SDHPC CICD workflow ensures we can deliver features and bug fixes through regular, reproducible, version-controlled deployments.

NOTE: Built cloud-native infra for interface to HPC system/env. These things, working together, make SDHPC a possibility.

- Extend sshpiper
- Gitlab CI/CD
  - Core infra
  - HPC services

**NEEDS** segue paragraph, tie to first PP.

(related-work)=
# Related Work

**NEEDS** a paragraph with an overview of the section.

<!-- establishing value of OpenStack -->
The success of research computing activities depends on access to information technology (IT) infrastructure: compute, storage, and networking.
OpenStack, an open source software (OSS) Cloud computing platform, provides software defined abstractions over that infrastructure @openstack2025.
NOTE: User access to OpenStack/VM infra are not common in RC systems.
NOTE: Decoupling hardware from the abstractions presented to the researcher is the central idea enabling SDHPC.
IT staff secure the physical systems, and researchers are empowered to build cloud-native applications from the software defined abstractions.

<!-- things what use openstack -->
The NSF-funded Jetstream2 project is an at-scale cloud computing environment available to researchers through the NSF ACCESS program [@Hancock2021; @Boerner2023].
The NFS ACCESS program is the successor to the XSEDE program which nurtured the development of national HPC resources to support research workflows @Towns2014.
Jetstream2 is built using OpenStack to provide cloud-native abstract research applications that need compute, storage, and network resources.
Jetstream2 allocates capacity for CPU, GPU, and storage using ACCESS-granted service unit credits.
Jetstream2 demonstrates at-scale operations of OpenStack clouds and provides autonomous access to advanced hardware for the development of research workflows.

CRI\_XCBC is a legacy software project created by the XSEDE program.
It provides a working example of a deployable, extensible HPC stack that used Ansible playbooks to deploy and OpenHPC defined infrastructure @CRIXCBC2024.
We maintain a site-specific fork of CRI\_XCBC to as our foundation for software defined cluster environments.
We extended this code-base to include OOD web services as part of the Ansible-deployed OpenHPC cluster @Tripathi2020.
We further extend this Ansible code-base in this work.

The "data center as a computer" model facilitates systems development, abstracting each subsystem as the component in a personal computer @Barroso2019.
Compute resources, both HPC and cloud, form the central processing unit (CPU).
Compute-adjacent storage, such as block devices and parallel network file storage, is system memory.
Object storage is hard drives, and the internal networks are mainboard traces and cables.
These abstractions guide organization of multi-site and multi-system infrastructure deployments.
Additionally, the model is easier to reason about, improving stakeholder communication and strategizing.

(sdhpc)=
# Software Defined HPC

HPC clusters have long aligned with the principles of software defined infrastructure  @Reed2014.
The original Beowulf model deployed fleets of identical commodity compute nodes under the control of a head node that also supplied core infrastructure for HPC operations @Becker1999.
Cloud infrastructure leverages these same approaches to provide scalable compute, storage, and network capacity @Lenk2009.

We adopted data center as computer to align our research computing infrastructure with a coherent system architecture.
The resulting platform serves as a target for SDHPC operations.
Application routers direct user connections to physical HPC systems infrastructure.
We follow Agile methodologies to facilitate development and management of the SDHPC services @Shore2021.
CICD pipelines build and deploy the application routers and OOD services enabling release engineering for SDHPC.

(sdhpc-rcs)=
## Research Computing System

<!--Building a cloud native experience. -->

The Research Computing System (RCS) is a scalable, cloud-native platform that encapsulates the data center as the computer design, see [Figure %s](#rcs_architecture).
RCS delivers compute, storage, and networking services to research applications.

<!--
if we have time lets remake this figure to be publication-ready

consider replacing with an svg rendering or similar
-->
```{figure} images/RCS Architecture.png
:label: rcs_architecture
:alt: RCS architecture shows a collection of nested boxes designating system boundaries for the system as a whole and each interior subsystem. The boxes are connected with lines representing the network interconnects.

RCS architecture following design of warehouse scale machines. Shows batch, vm, and container compute modalities connected to file, block, and object storage modalities via interconnect and peering to external networks.
```

RCS compute services provide HPC batch, virtual machine, and container modalities.
The compute services leverage storage services that provide parallel file systems, block devices, and object storage.
Compute and storage services are interconnected by networks that provide bandwidth for both east-west and north-south traffic flows.
The RCS peers with external networks, providing access from campus and external Research and Education (R&E) networks via a Science DMZ (SciDMZ) segment @Dart2013.

HPC batch computing is implemented via a traditional, physical compute cluster.
At our site, the HPC cluster is deployed using Bright Cluster Manager.
Compute nodes connect to a GPFS parallel file system via a dedicated Infiniband network.

RCS positions the HPC infrastructure along-side platforms that provide VM and container abstractions to applications.
The VM compute capacity is implemented via an on-premise OpenStack-based cloud.
This provides a comprehensive software defined infrastructure enabling the direct control over compute, storage and network resources needed to build and operate applications.
We use this infrastructure to host the application routers, OOD web application, and CICD workflows describe the in following subsections.
Container compute capacity for RCS is implemented via Kubernetes.
Today, SDHPC exclusively uses the VM and batch subsystems.
Work remains to port SDHPC to a microservices architecture.

In addition to GPFS parallel storage for HPC workloads, our implementation of RCS leverages Ceph storage subsystems to deliver block and object storage services to OpenStack and Kubernetes based workloads.
Ceph additionally provides public S3 endpoints for user data and Cephfs as a capacity tier backend for GPFS to help manage cost.
R&E network peering is provided by a SciDMZ segment that uses Globus as the I/O interface for moving data to GPFS and allows users to manage other storage services via additional Globus connectors @Chard2016.
Campus peering provides network access to all services in the RCS simplifying compliance with campus IT policies.
Enhancements to RCS network interfacing are under consideration to expand services delivered via the SciDMZ.

(sdhpc-app-router)=
## Application Routers

OOD elevates HPC to web-native experience.
Our goal with SDHPC is to elevate HPC to a cloud-native experience.
To achieve this goal it is necessary to have full control over all aspects of user interaction with the HPC platform.
Routing users to appropriate services based on their identity is key to ensuring a consistent exprience.
We built application routers for HTTPS and SSH connections.
The routers can direct user connections to specific endpoints based on user identity and account status.

Routing HTTPS connections is standard fare for Web-native applications.
We built a dedicated front-end HTTPS application router that directs users to an OOD instance based on their group membership.
The HTTPS router acts as a reverse proxy that authenticates the user via web SSO and routes their connection to the appropriate endpoint.
Our HTTPS router requirements were readily satisfied with standard Apache reverse-proxy and web SSO features.
We leverage user account attributes to make the routing decision.

Routing SSH connections is less common.
In addition, routing SSH connections based on user identity introduces special considerations.
The user identity is not confirmed until the SSH connection is terminated and the session is authenticated.
Once this SSH session active, it ordinarily results in a login shell on the endpoint which terminated the connection, implicitly preventing further routing.
To accomplish rule-based SSH routing based on user identity, we implemented an SSH application router using sshpiper, a reverse proxy for SSH built on Golang's SSH library @Lian2025.
We contributed an enhancement to sshpiper that allows a user's group membership to be use in the routing rules.

The SSH application router enables user connection routing to preferred login nodes based on site policy or other operational requirements.
The sshpiper routing works by terminating the client-side SSH connection and establishing a secondary internal SSH connection to the desired login node endpoint.
For password-based authentication, the users credentials are passed to and validated by the login node endpoint.
For key-based authentication, the user authentication is managed using internal cluster keys shared via the file system.
This uses the same infrastructure already in place on the HPC system.
HPC systems will typically allow users to transparently access compute nodes running their jobs without prompting for additional authentication.
This is done by creating an SSH keypair at account creation and automitically adding it to the user's authorized-keys file.
We use this existing HPC infrastructure to facilate sshpiper operations.

In order to align with the self-directed nature of cloud-native platforms, we previously built an account app that allows authorized users to create their HPC account.
User's without HPC accounts are redirected to the account app.
The account app prompts the user to create their account or instructs them that they are not authorized for this resource.
After an authorized user submits their account request, the account app interacts with standard HPC account creation services, exposed via a RabbitMQ message bus.
Once the account is created the user's web connection is redirected to OOD allowing the user to begin their HPC on-boarding.
The self-directed account creation request typically takes less than fifteen seconds to complete.
This capability ensures authorized users experience little more than a slight account provisioning delay the first time the access the HPC system.

We define a basic set of account states like good\_standing, certification\_required, and account\_hold to facilitate operations.
Only users with an account in good\_standing are allowed to interact with the HPC system via OOD or SSH.
Other states direct user connections to the account app so they can address any requirements to reestablish their good\_standing state.
The account\_hold state provides explicit control to support personal over individual user access to HPC resources for service events or other user engagements.

The application routers and account states combine to fully control user interaction with the HPC services provided by RCS.
The HPC system is only available to authorized users and the HTTPS and SSH application routers ensure connections are only estabilished with the endpoints that can provide services to specific users.

<!--- so we can actually say we have the openstack as a front end to our hpc
the advantage here is that we can use this same front end regardless of where a physical cluster may actually be located
openstack gives us cloud-native tooling and sdn to route traffic to desired destinations -->

(sdhpc-cicd)=
## CICD Pipelines

<!--- that should really about working from a defined image and customizing it via the user-data section to connet it to the runtime environment.

this is really where a picture could come in handy
we build images via packer with possible external repo

we build deploys from defined images
add hooks for late binding to target env
this is doen via user-data and direct openstack cli

this infrastructure relies on gitlab runner for packer and other deploy tasks
-->

SSH and HTTP application routers direct user traffic to login nodes, OOD instances, and the account app according to site operational goals.
This creates a coherent user experience for HPC access that is consistent across both HTTP and SSH endpoints.
Implementing these routers as software defined infrastructure (SDI) allows continuous development of platform features and control over the introduction of capabilities as they are deployed.

Our CICD workflows are built using GitLab CI/CD @gitlab-cicd.
We use a dedicated GitLab repository to house the CICD pipelines, following the image factory pattern @Muse2023.
We created CICD pipelines with separate build and deploy phases.
Both phases leverage Ansible to construct relevant artifacts.
The RCS cloud subsystem provides the infrastructure for development, build, and deployment workflows.
This enables the construction of comprehensive cloud-native workflows and facilates tight integration with campus HPC resources via direct connection to relavent provider networks.

Packer templates are used to construct VM images for the proxies, OOD and account app during the build phase.
These templates include extensions to interface with the OpenStack API and can readily work with other cloud providers.
We execute daily builds of our OOD image.
This ensures image construction remains viable and surfaces problems with build dependencies in a timely fashion.
The daily builds also support developement deploys to review feature improvements against our production HPC system.

For the deployment pipelines, we choose to directly invoke OpenStack CLI commands for implementation convenience.
We have used Terraform for the deploy phase of other system components not featured in this work and plan to migrate SDHPC artifact deployment to Terraform in the future.

The separation of build and deploy steps enables us to construct versioned VM binaries that can be used across development, test, and production environments.
The deploy steps focus on customizations that meet the needs of specific environments, providing late-binding hooks and other customizations for dev, test, and prod clusters.

<!---
### Core Infrastructure Builds

**Next 3 PP** can be squished into one efficient PP.

We build OOD VM images that include all dependencies of it's software stack and additional components for integration with our local HPC systems architecture.
The images are constructed using the legacy CRI\_XCBC Anisble repository.
CRI\_XCBC project was created under the NSF XSEDE initiative to provide an infrastructure-as-code (IaC) framework for HPC deployments.

We adopted CRI\_XCBC to provide developer instantiated HPC platforms to implement our original OOD deployment.
OOD is challenging for early career developers to learn, test, and extend without an HPC cluster with which it integrates.
In order to write systems applications, developers must be granted complete authority over the software stack that includes the HPC system stack, batch scheduler, and other core platform services.
DevOps workflows are easier to construct when development and production infrastructure are isolated from each other.
Developers need to be able to iterate repeatedly over the build and deploy pipelines.
Providing developer access to production resources is undesirable and works against the goal of creating repeatable builds and deployments.
Deploying disposable HPC clusters (dev clusters) as part of the development workflow aids construction of SDI and professional growth.

We forked and extended CRI\_XCBC to include Ansible roles to build an OOD image.
Constructing OOD within the CRI\_XCBC framework provided an HPC platform with which OOD integrates.
This also provided the necessary components to develop our web-based Account app that enables self-service HPC onboarding for users.
The account app provides a web UI that invokes native HPC account creation services via a RabbitMQ message bus.
Over time we abandoned use of the OpenHPC IaC elements from the CRI_XCBC framework.
We now rely on stand-alone Heat-based deployment of dev clusters built using Bright Cluster Manager (BCM) and the NVIDIA (formerly Bright Computing) Easy8 developer-focused cluster framework.
We use this cluster management infrastructure on our production cluster.
This move was necessary to support construction of BCM-specific drivers for our RabbitMQ based account creation services.

**Stop squishing**

A dedicated image factory repository uses GitLab CI/CD to construct the OOD and account application images by ingesting external Ansible rules from our CRI\_XCBC fork.
The Ansible framework is executed during the Packer image build.
The images are stored in the OpenStack Glance image repository and made available to developer and production projects.
This approach ensures we can track and test the latest changes to the IaC repositories and provide versioned images for development and production use.
To that end, we execute a daily build of our OOD image to ensure the construction remains viable and surfaces problems with build dependencies in a timely fashion.
Long delays between builds can otherwise lead to unexpected failures when it becomes necessary to update configurations in response to bugs or changes in operation.
The daily build also supports deployment of the development head to review feature improvements against our production HPC system.
We use this approach to enable the introduction of new OOD applications through the addition of an app-specific Ansible role in our CRI\_XCBC repo.

-->
<!-- following may not be necessary
While still rooted in CRI_XCBC, OOD and the Account app have become the only components we build out of this IaC framework.
We run the OOD build and deploy pipelines each day in order to maintain confidence in our ability to construct a complex VM image with many dependencies.
We use the daily builds of OOD as the foundation for multiple deployments.
We provide live deployment of the current development head to allow feature and bug fixes to be explored.
-->
<!---
### Deployment of HPC Services
-->
<!---  this is the interface for modern hpc + globus, we do not address globus routing in this work. -->

<!--
We created deploy pipelines for the HTTP and SSH application routers and the OOD and account application web applications.
The application routers are built from stock OS images, currently Alama9, that are customized during deploy with Ansible rules to include the HTTP and SSH servers configured to authenticate the user and then route to the correct backend service.
We deploy the HTTP and SSH on services to separate VMs for ease of construction and to facilitate service scaling if needed.
The HTTP application router is a standard Apache reverse proxy that includes SAML-based WebSSO using Shibboleth service provider components.
Once the user identity is know, an LDAP lookup against the HPC cluster's account database to gather group memberships.
We use membership in specific groups to choose the target OOD backend.

The SSH application router is built using sshpiper, a Golang SSH proxy server implemented on the Golang SSH library.
The sshpiper server is built from source with Ansible during deploy.
Additional Ansible tasks are used to configure the host to integrate with the HPC cluster system environment.
The key parts of this configuration include system level user account integration to support user account validation for the SSH protocol.
User home directories to be mounted via NFS to support standard user-level ssh configuration, e.g. ordinary management of the authorized_keys files for key-based authentication.
The SSH application router operates much like the familiar login node on any cluster with the exception that all SSH connections are passed through the application router.
No user shells are run on the application router.
The SSH router transparently establishes a second ssh connection to the internal termination point for the desired login node, where the user shell is started.
All user SSH based user interaction with the cluster will occur from the login node that hosts the shell process.

We deploy the OOD and account apps from the VM image created during the build phase described above.
Using pre-built VM images for OOD results in shorted deploy time.
We use the concept of late-binding to the customize the deployed image with the configuration required to interface with a specific cluster environment.
An OOD node has cluster integration expectations similar to login nodes.\
We characterize the late binding as interfacing the node with the clusters name resolution (DNS), account lookup (LDAP), file namespace (NFS), and batch job submission (Slurm).
The Ansible coded deploy steps adjust the pre-built image to interface with these necessary cluster services.

In all cases, we initiate the Ansible deploy steps during node instantiating using cloud-init user-data scripts.
These scripts are injected into the node by the cloud platform and executed on the node during instance startup.
The deploy pipelines directly execute commands that generate the user data script which is then passed to the invoked openstack CLI commands that instantiate the node.
After the node is instantiated, the pipeline assigns public floating IP address for external and cluster-internal networks to enable the node to accept user connections and interact with cluster services.

We use a blue-green deployment model where the node for a service is deployed using a non-production endpoint address.
Operation of the newly deployed node is validated in the blue state.
If the service is operating as expected, the services public IP address is moved to the newly deploy node.
This simplifies our migration to newly deployed services and avoids service interruption because the software define network service of the cloud platform provide a clean transition for traffic flows.
-->
<!---
There are many ways to handle this deployment cut of over.
Our current approach use manual IP address cutover after accept test are performed.
More advanced deploy pipelines could include automatic validation modeled after test-driven development.
These automated build and deploy of our CICD pipelines help ensure we can reliably and continuously release the latest improvements to the user experience.
-->
<!---
This is how it's deployed

- the app routers are built directly in gitlab ci as artificts contstructed directly on the openstack cloud infrastructure
- show construction of cicd pipelines and how the produce their artifacts
-->

(sdhpc-applied)=
# SDHPC for Data and Cluster Migration

```{figure} images/AB_cluster.png
:label: ab_cluster
:width: 100%
:alt: Software Defined HPC framework used to manage a two cluster deployment where a single visible entry point leads to two different cluster fabrics, transparent to the user experience.

An arrangement of the Software Defined HPC (SDHPC) framework to route users to distinct cluster environments based on their user identity and group membership. Group A web and ssh connections are routed to Cluster A and group B connections are routed to Cluster B.
```

The Software Defined HPC (SDHPC) framework provides features that enable a variety of service development and operations workflows.
We introduced SDHPC to maintain front-end application routers for our existing campus HPC user interfaces to support a multi-phase migration of data and infrastructure in our RCS.
The framework was developed in response to a confluence of events.
We were faced with vendor relationship changes, product licensing changes, product lifecycle transitions, data center power constraints, storage demand growth, and the financial constraint of annual budget cycles [@carlz-at-us.ibm.com2020; @Sedlmayer2020].
The GPFS-based, high-performance storage tier for the campus HPC system needed to move to the latest supported version.
Due to evolution in the vendor landscape, this necessitated moving petabytes of existing data to new hardware from a new vendor.
Power constraints in the on-campus data center drove decisions to consolidate HPC CPU resources alongside other RCS compute resources in newly available high-powered racks at a nearby commercial data center already hosting our HPC GPU resources and the cloud and container components of RCS. @DcbloxHDcolo
The new facilities provided sufficient power to support anticipated growth of our compute capacity across RCS.
This consolidation would also return HPC operations to a single Infiniband segment serving all HPC compute resources with access to the updated GPFS platform, which includes an NVMe performance tier to support I/O throughput demands for emerging AI workloads.
The compute node consolidate freed on-campus data center space to allow expansion of the capacity tier storage provided by Ceph.
This growth was designed to leverage annual operating budgets through multi-year infrastructure improvement cycles.
The approach allowed us to maximize GPFS performance for HPC applications and maximize storage capacity for large data sets a unified namespace by leveraging policy-based tiering of infrequently used data to a more granular, lower cost storage platform powered by Ceph.

The SDHPC framework supports balancing competing priorities.
[Figure %s](#ab_cluster) shows our initial use-case to migrate the HPC user community from the original GPFS storage platform to the new, tiered solution.
Implementing the storage migration required reconstructing the RCS HPC subsystem in new data center facilities.
In order to avoid extended downtime for the entire HPC community to accommodate data movement and system relocation, we created an initial footprint of HPC nodes as a separate cluster attached to the new GPFS platform.
We designate Cluster A in the figure as the HPC resource associate with the original storage platform  and Cluster B as the HPC resources associate with the new storage platform.
This approach facilitated incremental movement of user and HPC capacity as data and data center migrations completed.
It also provided a load testing ramp for the new tiered storage solution.

We were able to move subsets of the user community by designating users accounts as members of group A or group B to direct their connections to cluster A or cluster B, respectfully, depending on the location of account data.
These assignments are transparent to the end user.
The SSH and HTTP endpoints remain the same for all users regardless of their assigned cluster.
SSH and HTTP application connections are routed to the appropriate nodes.
The file system namespace is consistent across storage systems using bind mounts in each cluster.
The job submission experience remains the same.
Resource scheduling is modified using the SLURM job submit plugin to route user jobs to compute nodes connected to the storage platform housing their data.
The same group name mechanism used by front-end application routers modified submitted jobs to tag them with a feature to select appropriate nodes within a partition.

Keeping the user experience consistent during this transition minimizes the cognitive load on users by avoiding changes to HPC workflows.
User connections are routed to the cluster with the new storage system when their data migration is complete.
All user data is seeded to the new storage system with regular synchronization runs using dsync from mpiFileUtils.
When the user's migration from Cluster A to Cluster B is scheduled, their account is place on hold to avoid changes to their data.
A final sync from the source system is performed to capture any changes since the most recent seed point.
Their file namespace is stubbed into the new GPFS front end from the Cephfs capacity tier.
Their cluster assignment group is updated and their account hold is released.
From then on their connections are routed to the Cluster B.
From the user perspective, each migrated user effectively experiences a blue-green deployment model.
They are cut over to the new production system when it is validated as operational for their account.

The cloud services of the RCS platform provide an ideal hosting environment for this infrastructure.
The comprehensive software defined infrastructure provided by OpenStack enables the creation of cloud-native development and operations workflows.
We provisioned the components for application routing and cluster B's OOD node in an OpenStack project dedicated to HPC operations.
The project is authorized to access campus and cluster provider networks made accessible through OpenStack SDN services.
This allows the services to accept user connections that can be routed to the desired cluster nodes.
We route users of cluster A to it's existing login and OOD services.
We route users of cluster B to a newly provisioned physical login node and VM instance OOD, hosted in the same project.
We chose to retain a physical login node for Cluster B to avoid performance scaling concerns and because of the ease with which these can be deployed using the traditional cluster managers.
The cloud-based application routers and OOD instance have access all necessary cluster services via the cluster provider network.
We use NFS to share the storage namespace with the SSH application router and OOD VM, as opposed to native GPFS clients as is the case with physical nodes connected to respective InfiniBand fabrics of the cluster environment.
The storage workloads for sshpiper routing and OOD interaction are relatively light and have not presented performance issues using NFS.

**Consider** including sshpiper data we gathered to build our confidence in our methodology somewhere in here. No need for long discussion, no need for graphs, merely some numbers and text as an additional point of interest. Want to indicate we are sensitive to performance concerns.

**Also** The full OOD pipeline build runs in 20 minutes, and deployment runs in 3-5 minutes. The rapid deployment means we can get more testing done, enhancing quality.

The cloud hosting environment is also a natural fit for CICD driven development and production workflows.
A self-hosted GitLab instance provides CICD tooling to develop and deploy the SDHPC into production using a dedicated hpc-factory project.
Developers contribute improvements to the hpc-factory which are validated with integration pipelines.
New releases are delivered to production in a timely manor using deployment pipelines.

This provides familiar software development paradigms that enable contributions from a broader community.
It takes time to learn the full spectrum of HPC operations.
Seasoned professionals are responsible for daily cluster operations and often do not have time for relatively minor updates to non-core services.
Enabling early career developers to focus on more narrowly scoped services and successfully contribute features.

The most significant advantage of this approach is to reduce the friction between development and production environments.
Traditional approaches deploy ancillary HPC services to physical hardware in close proximity to the cluster.
This introduces a technology gap between how production nodes attached to the HPC fabric are managed versus how these services are operated during development.
That friction frequently leads to delays in updating services like OOD because they are deployed with to dedicated hardware with manual steps executed by core operations staff.
Reducing this friction increases the velocity of releases and allows the HPC platform to more easily adapt to user requirements.
Another advantage is that the same services can be deployed to any cloud environment which supports the construction of HPC environments on the most cost-effective platforms.

<!---
conclusion can mention the complexity of the data movement as separate work
this does acheive our ability to fluidly move people and projects
conclusion can note the side effect is more autonomy of science engagement which is helping drive our goal for end-user managed infrastructure.
-->

(conclusion)=
# Conclusion

Infrastructure as code demands programmatic control over infrastructure.
Automated construction of nodes ensures they reliably contain appropriate components and configurations to implement the service.
Programmatic control of infrastructure ensures services are properly integrated with the production environment.
These controls are common during development.
Ensuring the same experience for production reduces deployment friction and leads to improvement in the user experience.
Moving services to a cloud native model is necessary to ensure the steps that are built and tested during development are valid for the production environment.

<!--- note disaggration of ood and account app -->

We introduced user-based application routing for SSH and HTTP connections to the HPC environment.
We leveraged on-site cloud computing resources in our RCS to build CICD pipelines for our application routers and OOD services creating a SDHPC framework.
This framework has facilitated a data and cluster migration project while minimizing user disruptions.
The SDHPC is responsive to bug fixes and supports timely deployment of feature enhancements.

These new platform features help create a managed HPC user experience.
Policy-based routing of user workloads maintains service availability in the face of significant changes to underlying systems.
It has facilitated data migration for a major storage upgrade and will support additional improvements to the HPC system in the future.
Using flexible, modern paradigms like CICD creates testable features with reduced development time and reliable deployments.
Providing stable services that can be enhance through continuous development and capacity expansion is the goal of cloud-native platforms.

Data migrations are complex undertakings.
The longer data sits still the harder it becomes to move.
This experience is leading us to explore continuous data movement to reduce the dwell time and dependency on specific storage hardware.
We are also expanding use of the image factory model to simplify introduction of new HPC service and reduce maintenance overhead.
By creating consistent CICD workflows between development and production to deploy application routers we have moved our HPC cluster to a cloud-native orientation that provides comprehensive control over the user experience.
