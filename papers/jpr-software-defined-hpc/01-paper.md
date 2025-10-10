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
      - University of Alabama at Birmingham, IT Research Computing
    orcid: 0009-0007-2599-4063
    email: jpr@uab.edu
    roles:
      - Conceptualization
      - Writing – original draft
      - Project administration
      - Methodology
  - name: Louis Chen
    affiliations:
      - University of Alabama at Birmingham, IT Research Computing
    orcid: 0009-0005-7118-1712
    roles:
      - Software
      - Validation
  - name: Eesaan Atluri
    affiliations:
      - University of Alabama at Birmingham, IT Research Computing
    orcid: 0009-0004-1699-2849
    roles:
      - Software
      - Validation
  - name: Krish Moodbidri
    affiliations:
            - University of Alabama at Birmingham, IT Research Computing
    roles:
      - Software
      - Validation
  - name: William Warriner
    affiliations:
      - University of Alabama at Birmingham, IT Research Computing
    orcid: 0000-0002-9733-9187
    roles:
      - Writing – review & editing
      - Validation
abstract: |
  Software defined infrastructure for high-performance computing facilitates service availability.
  An infrastructure of HTTP and SSH application routers is developed and deployed using continuous integration and continuous deployment pipelines.
  We demonstrate the utility of this infrastructure to our motivating use case of limiting user downtime during maintenance operations.
  In this implementation, the complete system of users and resources is divided into two logical groups.
  Based on group membership, users are exposed to only the resources associated with their group.
  We provide insights on SSH application router performance and conclude with observations on functionality and highlight future directions.

acknowledgments: |
  We extend our gratitude to several individuals who generously shared their expertise and helped us improve our understanding of tools leveraged in this work.
  Doug Johnson at the Ohio Supercomputer Center for sharing insights on approaches to large scale data migration.
  John Michael Lowe and Jeremy Fischer at Indiana University for deep insight on Jetstream2 operations.
  We appreciate their professional courtesy but accept all responsibility for the implementation choices and conclusions presented in this work.

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
Scientists must adapt to a competitive, rapidly-changing research landscape.
As such, research computing staff must quickly adapt systems to changing requirements and improving technology, while minimizing impact to researcher operations.
Traditional high-performance computing (HPC) infrastructure is defined by fixed services closely aligned with hardware layouts.
In contrast, a software defined infrastructure for high-performance computing (SDHPC) is responsive and flexible, enabling in-flight upgrades with reduced researcher impact.

Traditional HPC use centered on text-based, command-line interactions where users connect via SSH to dedicated login nodes to execute batch-oriented research workflows @Lonvick2006.
The development of immersive web interfaces to HPC systems provided a friendlier user experience, broadening access.
Among these interfaces, Open OnDemand (OOD) has emerged as the most successful @Hudak2018.
Now a fundamental component of many HPC systems, OOD has aligned user expectations with the self-directed experience of cloud-native applications.

OOD owes its success to a reduction of cognitive load for researchers.
The reference deployment of OOD provides access to several interactive, web-native applications.
Jupyter Notebooks, R Studio, and a virtual desktop capable of presenting native, windowed applications, are all available behind a simple resource request form.
Researchers start by engaging with their HPC system via the browser using familiar web applications and, when they are ready, can migrate to batch tools at the command-line.
Implementation details, such as submitting a job script to the scheduler, are hidden behind the software-defined abstractions of the resource request form and application definition.
Because OOD hides these details, researchers can more readily incorporate HPC into their research workflows.

OOD operates by mapping browser interactions to per-user web servers.
When a user connects to the HPC system via HTTPS, the OOD server spawns a web server process running under their account identity, as defined on the HPC system.
Whenever the user executes actions on the web site, processes started by the web server run within their identity context.
The net effect is that user engagement with the HPC system is shepherded by software definitions—identity, operating system rules, OOD application definitions, and the scheduler.
User interaction via OOD is governed by the same permissions enforced by the operating system on all the user's processes, whether started trough OOD or traditional command-line accessed via SSH.

OOD transforms HPC user engagement into a web-native experience.
SDHPC transforms the HPC system itself into a cloud-native application.
Application routers for SSH and HTTPS enable us to modify the HPC environment for arbitrary subsets of the user community.
This identity-based routing requires that the application routers authenticate users in order to direct their connections to the correct resources.
Web authentication is handled by a web single sign-on (SSO) enabled Apache reverse-proxy routing users based on group membership.
SSH authentication and routing is handled by sshpiper, an open-source SSH proxy built on top of the Golang SSH package @Lian2025.
We contributed an extension to sshpiper that enables group-based routing.

The application routers introduce an opportunity for horizontal scalability, further enabling SDHPC to deliver cloud-native infrastructure capabilities.
Continuous integration and continuous deployment (CICD) methodologies enable greater scaling of cloud-native infrastructures @Ugwueze2024.
To leverage the opportunity, we created a GitLab CICD workflow to build and deploy our application routers and OOD web services @gitlab-cicd.
Our SDHPC CICD workflow ensures we can deliver features and bug fixes through regular, reproducible, version-controlled deployments.

In this work we document SDHPC, a cloud-native framework for managing interaction with HPC clusters.
We introduce a Research Computing System (RCS) model that describes a coherent infrastructure for building scientific applications.
RCS provides both the HPC services that are the focus of SDHPC and a development platform that supports cloud-native automation.
We describe the implementation of SDHPC and the cloud-native automation used to build and operate these services.
We discuss data and data-center migrations that served the motivating use case for SDHPC development.
We provide insights on SSH application router performance and conclude with reflections on the utility of SDHPC and future directions.

(related-work)=
# Related Work

The success of research computing activities depends on access to information technology (IT) infrastructure: compute, storage, and networking.
OpenStack, an open source cloud computing platform, provides software defined control over infrastructure @openstack2025.
IT staff can guard access to the physical systems running OpenStack to ensure secure operations while democratizing access to resource abstractions that enable developers to build applications.
Deploying OpenStack for research infrastructure decouples IT operations and research software development.

The NSF-funded Jetstream2 project, a cloud computing environment and NSF ACCESS resource, offers compute resources to researchers via OpenStack [@Hancock2021; @Boerner2023].
The NSF ACCESS program succeeds the XSEDE program, which nurtured development of national HPC resources in support of research workflows @Towns2014.
Compute and storage resources, allocated to researchers with ACCESS-granted service unit credits, are provided by the cloud-native resource abstractions of OpenStack.
Jetstream2 demonstrates at-scale operations of OpenStack clouds and furnishes autonomous access to advanced hardware for the development of research workflows.

CRI\_XCBC exemplifies deployable, extensible HPC stacks. A legacy software project of the XSEDE program, CRI\_XCBC uses Ansible playbooks to deploy an OpenHPC-defined cluster infrastructure @CRIXCBC2024.
We maintain a site-specific fork serving as our foundation for software defined HPC cluster environments @Robinson2025.
The fork has been extended with Ansible-deployed OOD web services @Tripathi2020.
We further extend this Ansible code-base to support SDHPC in this work.

The "data center as a computer" model facilitates systems development, abstracting complex subsystems as the component in a personal computer @Barroso2019.
Compute resources, both HPC and cloud, form the central processing unit (CPU).
Compute-adjacent storage, such as block devices and parallel network file storage, is system memory.
Object storage are the hard drives, and the internal networks are mainboard chips, traces, and cables.
These abstractions guide organization of multi-site and multi-system infrastructure deployments.
Additionally, the model is easy to reason about, improving stakeholder communication and strategizing.

(sdhpc)=
# Software Defined HPC

HPC clusters have long aligned with the principles of software defined infrastructure @Reed2014.
The original Beowulf model deployed fleets of identical commodity compute nodes under the control of a head node that also supplied core infrastructure for HPC operations @Becker1999.
Cloud infrastructure leverages these same approaches to provide scalable compute, storage, and network capacity @Lenk2009.

We adopted the "data center as a computer" model to align our research computing infrastructure with a coherent system architecture.
The resulting platform serves as a target for SDHPC operations.
Application routers direct user connections to physical HPC systems infrastructure.
We follow Agile methodologies to facilitate development and management of the SDHPC services @Shore2021.
CICD pipelines build and deploy the application routers and OOD services enabling release engineering for SDHPC.

(sdhpc-rcs)=
## Research Computing System

The Research Computing System (RCS) is a scalable, cloud-native platform that encapsulates the "data center as a computer" design, demonstrated graphically in [Figure %s](#rcs_architecture).
RCS delivers compute, storage, and networking services to research applications.

```{figure} images/RCS Architecture.png
:label: rcs_architecture
:alt: Network graph diagram of RCS architecture made of a collection of boxes, each containing text, connected by lines.

RCS architecture follows warehouse-scale computer design. Batch HPC, virtual machine (VM), and container compute modalities (top) are all connected to an L3 ethernet and infiniband interconnect (center). The interconnect links to a parallel file system (FS) and to multi-modal storage with FS, block, and object modalities (memory and HDD, bottom). The interconnect also provides peering to external networks (bottom).
```

RCS compute services provide batch HPC, virtual machine (VM), and container modalities.
Compute services leverage storage services that provide parallel file systems, block devices, and object storage from pools of commodity storage hardware.
Compute and storage services are interconnected by networks that provide bandwidth for both east-west and north-south traffic flows.
RCS peers with external networks, providing access from campus and external Research and Education networks via a Science DMZ (SciDMZ) segment @Dart2013.

HPC batch computing is implemented via a traditional, physical cluster of compute nodes.
At our site, the HPC cluster is deployed using Base Command Manager (BCM), formerly Bright Cluster Manager.
Compute nodes connect to a GPFS parallel file system via a dedicated Infiniband network.
RCS positions the HPC cluster infrastructure alongside companion services that provide VM and container abstractions to applications.

VM compute services are implemented via an on-premises, OpenStack-based cloud.
OpenStack provides a comprehensive, software-defined infrastructure enabling direct control over the compute, storage, and network resources needed to build and operate applications.
We use this infrastructure to host the application routers, OOD web application, and CICD workflows described in the following subsections.

Container compute capacity for RCS is implemented via Kubernetes.
Today, SDHPC exclusively uses the VM and batch subsystems.
Work remains to leverage microservice architectures in SDHPC.

Our implementation of RCS provides two storage subsystems: GPFS and Ceph.
GPFS serves as parallel storage for HPC workloads, while Ceph delivers block and object storage services to OpenStack and Kubernetes based workloads.
Public S3 endpoints are available for user data through Ceph object storage, and CephFS serves as a capacity tier backend for GPFS to help manage storage costs.
Research and Education network peering is provided by a SciDMZ segment that uses Globus as data transfer interface for GPFS.
Globus allows users to manage other storage services via additional connectors @Chard2016.
Campus peering provides network access to all services in the RCS, simplifying compliance with campus IT policies.
Enhancements to RCS network interfacing are under consideration to expand services delivered via the SciDMZ.

(sdhpc-app-router)=
## Application Routers

OOD elevates HPC to web-native experience.
Our goal with SDHPC is to elevate HPC to a cloud-native experience for system users and operators.
Achieving this goal requires full control over all aspects of user interaction with the HPC platform.
Routing users to appropriate services based on their identity is crucial to ensuring a consistent experience.
We built application routers for HTTPS and SSH connections.
To that end, we built application routers capable of directing user HTTPS and SSH connections to specific endpoints based on identity and account status.

Routing HTTPS connections is standard fare for web-native applications.
We built a dedicated front-end HTTPS application router that directs users to an OOD instance based on their group membership.
The HTTPS router acts as a reverse proxy, authenticating users via web SSO and routing their connection to the appropriate endpoint.
Our HTTPS router requirements were readily satisfied with standard Apache reverse-proxy and web SSO modules.
We leverage user account attributes to make routing decisions.

In practice, SSH connection routing is less common, and routing SSH connections based on user identity introduces special considerations.
User identity is not confirmed until the SSH connection is terminated and the session authenticated.
Once the SSH session is active, it ordinarily results in the creation of a login shell on the host which serves as the connection endpoint, implicitly preventing further routing.
To accomplish rule-based SSH routing based on user identity, we implemented an SSH application router using sshpiper, a reverse proxy for SSH built on Golang's SSH library @Lian2025.
We contributed an enhancement to sshpiper which allows a user's group membership to be considered in its routing rules.

The SSH application router enables routing to preferred login nodes based on site policy or other operational requirements.
With sshpiper, routing works by daisy-chaining two SSH connections.
The first connection is terminated at application router from the client side, and the second connection is established from the application router to the desired login node.
For password-based authentication, the user's credentials are passed to, and validated by, the login node endpoint by the secondary connection.
Key-based authentication is accomplished by using a separate internal SSH key to authenticate the secondary connection to the login node.
HPC systems typically allow users transparent access to compute nodes running their jobs without prompting for additional authentication.
They implement this behavior by creating account-specific SSH keypairs as part of the account creation process, automatically adding the keys to the user's authorized-keys file.
We leverage this existing HPC infrastructure to facilitate sshpiper operation by accessing the user's SSH configuration via the shared file system.

To align with the self-directed nature of cloud-native platforms, we previously built a web app for self-directed account creation and management.
When attempting to authenticate to OOD, users without HPC accounts are redirected to the account app.
If authorized, the app prompts users to create an account, otherwise informs them they are not authorized.
After an authorized user submits their account creation request, the app interacts with standard HPC account creation services, exposed via a RabbitMQ message bus.
When the account is created, the user's web connection is redirected to OOD, allowing the user to begin HPC on-boarding.
The account creation request typically takes less than fifteen seconds to complete.
This capability ensures authorized users experience little more than a slight account provisioning delay the first time they access the HPC system.

We define a basic set of account states like "good\_standing", "certification\_required", and "account\_hold" to facilitate operations.
Only users with accounts in "good\_standing" are allowed to interact with the HPC system via OOD or SSH.
Users in other states have their connections redirected to the account app, informing them of their account state, giving them a chance to address any requirements to reestablish their "good\_standing" state.
The "account\_hold" state provides explicit control to support personnel over individual user access to HPC resources for service events or other user engagements.

Combined, our application routers and account states enable full control over user interaction with HPC services provided by RCS.
With the routers and account states in place, our HPC system remains available only to authorized users and connections are only established with resources appropriate to each user.

(sdhpc-cicd)=
## CICD Pipelines

SSH and HTTP application routers direct user traffic to login nodes, OOD instances, and the account app according to site operational goals.
Routing creates a coherent user experience for HPC access that is consistent across both HTTP and SSH endpoints.
Implementing these services as Infrastructure as Code (IaC) allows continuous development of platform features and control over the introduction of capabilities as they are deployed @Ramos2015.

Our CICD workflows are built using GitLab CICD within a self-hosted, on-premises GitLab server @gitlab-cicd.
We use a dedicated GitLab repository to house the CICD pipelines, following the image factory pattern @Muse2023.
We created CICD pipelines with separate build and deploy phases @uabrc2025.
Both phases leverage Ansible to construct relevant artifacts.
The RCS cloud subsystem provides the infrastructure for development, build, and deployment workflows.
This enables the construction of comprehensive cloud-native application construction workflows and facilitates tight integration with HPC resources via direct connection to relevant provider networks.

Packer templates are used to construct VM images for the proxies, OOD, and account app during the build phase @HashiCorpPacker2025.
These templates include extensions which allow interfacing with the OpenStack API available in RCS, and additional modules readily work with other cloud providers, ensuring workflow portability.
To ensure image construction remains viable and to surface problems with build dependencies in a timely fashion, we execute daily OOD image builds.
The daily builds support nightly deploys, enabling us to review OOD feature improvements and bug fixes against our production HPC system.

For the deployment pipelines, we choose to directly invoke OpenStack CLI commands for implementation convenience.
We have used Terraform for the deploy phase of other system components (not featured in this work) and plan to migrate SDHPC artifact deployment to Terraform in the future @HashiCorpTerraform2025.

The separation of build and deploy steps enables us to construct versioned VM binaries that can be used across development, test, and production environments.
The deploy steps focus on customization to meet the needs of specific environments, providing late-binding hooks for integration with dev, test, and prod clusters.

(sdhpc-applied)=
# SDHPC for Data and Cluster Migration

The SDHPC framework provides features enabling a variety of service operation and development scenarios.
We introduced SDHPC to maintain stable HTTP and SSH endpoint definitions for users during a multi-phase migration of the HPC services in our RCS.

```{figure} images/AB_cluster.png
:label: ab_cluster
:width: 100%
:alt: Schematic diagram showing connections between application routers (top) and the OOD and login nodes (middle) of two distinct clusters (bottom).

In the arrangement of Software Defined HPC (SDHPC) shown, user routes are established between application routers (top) and login and OOD nodes (middle). Group A web and ssh connections are routed to the login and OOD nodes (middle-left), respectively, of Cluster A (bottom-left). Likewise for group B and Cluster B (right).
```

Our SDHPC framework was developed in response to a confluence of events.
In addition to vendor, product license, and product lifecycle challenges, we faced power constraints, increasing storage demand, and the limits of annual budget allocations.

The GPFS parallel storage for our HPC system required a version upgrade to ensure support continuity.
Changes in the vendor landscape required moving petabytes of data to a new GPFS implementation from a new vendor @Sedlmayer2020.
Product license changes led us to develop a cost-effective storage system that transparently moves inactive files from the GPFS performance tier to a capacity tier implemented with CephFS @carlz-at-us.ibm.com2020.
Our implementation presents users with a unified file namespace for HPC applications, maximizing performance for active analyses with GPFS, and capacity for file retention with CephFS.

Power constraints within the on-campus data center drove decisions to consolidate all RCS compute services in newly-available, high-power racks at a nearby commercial data center @DcbloxHDcolo.
Consolidated compute services include the HPC batch compute cluster and its associated GPFS performance tier, the VM cloud platform, and the container platform.
The new facility provides sufficient power for planned growth of all RCS compute capacity.
The facility is integrated with the on-campus data center using dark fiber provided by the University of Alabama System Regional Optical Network (UASRON).
This configuration allows the on-campus data center to continue hosting less power-dense Ceph capacity storage services and to provide peering points for campus and regional networks.

The SDHPC framework helps navigate complex requirements.
[Figure %s](#ab_cluster) highlights the HPC storage and data center migration use-case that drove its development.
Implementing the migration required complete reconstruction of our HPC infrastructure in the new data center facility.
To avoid extended downtime for our entire user community, we planned the data migration around moving isolated subsets of users in batches across the storage systems.
We planned the compute migration by staging an initial footprint of HPC compute capacity as a separate cluster attached to the new GPFS platform in the new facility.

We designate Cluster A in #fig:ab_cluster as the HPC resources associate with the original storage platform and Cluster B the resources associated with the new storage platform.
This provides distinct environments onto which users can be mapped depending on their data migration status, reflected by membership in group A or group B.
This split cluster approach allows moving users and HPC capacity as batches of data are migrated.
This incremental migration provides a natural load-testing ramp for the new storage solution.

The SDHPC HTTP and SSH application routers provide stable endpoints for all users, regardless of their migration status.
Moving a user is accomplished by changing the group membership of their account from group A to group B.
This group membership directs their connections to cluster A or cluster B, respectively, depending on the location of their data.
Group and Cluster ressignment is transparent to the user.

To further simplify the user experience, we provide the same file system namespace and scheduler partitions in both cluster environments.
The file system namespace is made consistent across cluster environment by using appropriate bind mounts.
The job submission experience is made consistent by using the SLURM job submit plugin to route user jobs to compute nodes connected to the storage platform housing their data.
The same group memberships directing user connections are used to modify submitted jobs, tagging them with a feature to select appropriate nodes within a requested partition, allowing a single point of control.

Keeping the user experience consistent during this transition minimizes user cognitive load, avoiding changes to their HPC workflows.
Our data migration workflow seeds user data to the CephFS capacity tier on the new storage system, with regular update syncs, using the dsync application from mpiFileUtils @osti_mpifileutils.
When a user's migration from Cluster A to Cluster B is scheduled, their account is placed in the "account\_hold" state to avoid changes while data is in-flight.
A final sync from the source system captures changes since the most recent seed point.
The user's file namespace is stubbed into the new GPFS performance tier from the CephFS capacity tier.
After a final data validation, the user's state change is unwound and they are placed in group B, ensuring subsequent HTTP and SSH connections are routed to Cluster B.
Each migrated user effectively experiences a blue-green deployment model [@Fowler2010; @Humble2013].

The RCS OpenStack cloud service provides an ideal hosting environment for production SDHPC services.
We provision application routers and cluster B's OOD node to an OpenStack project dedicated to HPC production operations.
The project is authorized to access campus and cluster provider networks, made available to the project through the OpenStack software defined networking (SDN) services.
SDN services enable the application routers to accept user connections and route them to the appropriate cluster.
Furthermore, the virtual infrastructure helps frame the functionality of application routers as traffic flow control agents for HPC connections.

The RCS cloud service is also a natural fit for CICD-driven workflows.
Developer contributions to the image factory can be validated with integration pipelines.
New releases are delivered to production using the same pipelines used in development.
This cloud-native software development workflow enables rapid feature iteration during product development cycles and reduces friction between development and production deployments.
Deployment of testable integration environments takes only a few minutes, reducing test barriers and increasing test frequency, leading to enhanced service quality.

(ssh-router-perf)=
# SSH Application Router Performance

We considered the potential for performance impacts of SDHPC through the introduction of cloud-native (VM-based) application routers.
HTTP reverse-proxies are a standard part of the OOD deployment and common for at-scale cloud deployments.
As such, SDHPC HTTP application router performance did not present any concerns and was not explicitly studied.
Instead, we chose to align the HTTP application router configuration with VM sizing recommendations provided in the OOD documentation.
In order to understand impacts on SSH performance, however, we measured data transfer throughput for a variety of SSH connection scenarios with and without the use of the sshpiper application router.

:::{table} SSH Throughput Tests. All values in Megabytes per second. Results of SCP transfers of a 2 Gigabyte file to destination hosts, averaged over thirty tests.
:label: table-sshperf
:align: center

| Scenario | SCP Target              | Mean   | Error Margin | Confidence       |
|----------|-------------------------|--------|--------------|------------------|
| A        | Project VM              | 369.33 | 7.92         | 361.42 -- 377.25 |
| B        | Login node              | 119.80 | 7.11         | 112.69 -- 126.92 |
| C        | Proxy via physical node | 98.45  | 4.81         | 93.64 -- 103.26  |
| D        | Proxy via Project VM    | 84.44  | 5.32         | 79.13 -- 89.76   |
| E        | Proxy via sshpiper      | 81.47  | 1.52         | 79.95 -- 82.99   |
:::

In [](#table-sshperf) we show the throughput of SSH data transfers using SCP to copy a two Gigabyte file (base-ten) to a target node.
All tests originate from a VM in an OpenStack project.
All SSH components, except for the sshpiper application router, are implemented using standard OpenSSH packages provided by vendor distributions.
The results represent the performance of wired clients in different scenarios.
In scenario A the originating VM is used to transfer data to a VM local to the OpenStack project used for testing.
This provides an overall control for expected maximum SSH performance achievable by the originating VM.
We do not expect to replicate this performance in other scenarios because all other tests are bound by the performance of the network throughput to the HPC login node, 10Gbps in the current environment.
Scenario B serves as control for SSH performance to the production interface of the existing HPC login node.
Ideally, the performance of the remaining scenarios would closely match this control.

The remaining scenarios each represent different SSH proxy configurations, all of which use two SSH connections to reach the login node.
These scenarios are most comparable to our SDHPC SSH application routing, capturing the impact of double SSH connection.

Scenario C is a standard SSH jump host configuration where an SSH connection is established to the jump host, and a second SSH connection is established from client to login node through the jump host.
With this configuration, we avoid all possible hypervisor and SDN overhead, giving us a basis for comparison with subsequent scenarios.
We used the production login node as the jump host and a cluster compute node as stand-in for a physical login node accessed via proxy.
This SSH path mirrors the cluster topology supporting sshpiper, a front end SSH target directly connected "within" the cluster to a target login node.
From the data, we see that the introduction of the second SSH connection decreased throughput by about 18% compared with scenario B, suggesting introduction of SSH application routing will impact real-world throughput.

Scenario D represents the same SSH jump host configuration as scenario C, but replaces the physical front-end node with a VM running in the same OpenStack project.
This matches the intended cloud-native deployment for our SDHPC SSH application router but without introduction of the new Golang-base SSH implementation in sshpiper.
This configuration introduces a 30% drop in performance compared with scenario B.

Finally, Scenario E represents the planned production sshpiper deployment of the SDHPC SSH application router.
This allows us to understand the performance of the Golang SSH package leveraged by sshpiper to provide the SSH proxy functionality.
Comparing scenario E to D, we observe a slight decrease in throughput, giving a 32% drop compared with scenario B.
The sshpiper SSH application router, however, enjoys a significant reduction in the error margin over all other scenarios. <!-- see my comments in slack -->
This suggests stable performance for interactive command-line use.
Although the sshpiper-based application router has lower throughput, we considered the stable performance ideal for interactive use.
We also consider the throughput acceptable for less demanding data transfer workloads.
We offer high-speed data transfer via Globus endpoints for demanding workloads.
While Scenario E shows that SSH application routing only achieves two-thirds the throughput over the un-routed Scenario B, the tests indicate this is most likely a result of the daisy-chained SSH connection rather than a limitation of sshpiper itself.
Crucially, sshpiper is the only scenario that supports per-user routing of SSH connections based on account attributes.

A more detailed study of SSH application router performance is warranted.
It will be informative to understand the origin of performance impacts from daisy-chained SSH connections.
The sshpiper application router performance is not significantly below traditional SSH jumphost proxy scenarios in the same deployment scenarios and could likely be improved with conventional performance tuning, e.g. using physical hardware instead of VMs.
Nonetheless, we can envision the deployment of highly scalable environments given the SDHPC SSH application routing functionality.

(conclusion)=
# Conclusion

The SSH and HTTP application routing features of SDHPC facilitate the creation of a cloud-native HPC user experience.
Policy-based routing of user workloads maintains service availability in the face of significant changes to underlying infrastructure.
SDHPC simplified data migration for a major storage upgrade and will support future improvements to the HPC service in RCS.
Using cloud-native development paradigms and CICD pipelines for SDHPC operations also improves service reliability by simplifying feature testing and reducing the length of development cycles.
Providing stable interfaces that support controlled introduction of platform features and infrastructure improvements is the goal of cloud-native platforms.
Nonetheless, work remains to minimize impacts on data transfer performance when routing SSH connections.

We are expanding our use of the image factory model to simplify introduction of new HPC services and reduce maintenance overhead.
Creating consistent CICD workflows for development and production increases feature contribution opportunities.
IaC demands programmatic control over infrastructure.
Automated construction of systems improves their reliability.
Moving service construction to a cloud-native model ensures IaC tested during development remains valid for production.
SDHPC is responsive to bug fixes and supports timely deployment of feature enhancements.

Data migrations are complex undertakings, and the longer data sits still the harder it becomes to move.
So, it is important to ensure that data always resides on the most appropriate storage resources.
This migration experience has led us to explore ways to implement continuous data movement.
We want to avoid having data pool on the oldest hardware merely because that is where it originally landed.
This approach should help avoid heavy data lifts when current platforms are retired in the future.
Effectively developing these data workflows requires automated processes that can implement placement based on site policy.
SDHPC provides the framework for improving HPC services on the RCS.
