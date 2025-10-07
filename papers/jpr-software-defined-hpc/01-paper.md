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
  We provide insights on SSH applicaiton router perforamnce and conclude with observations on functionality and highlight future directions.

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
Scientists must adapt to a competative, rapidly-changing research landscape.
As such, research computing staff must quickly adapt systems to changing requirements and improving technology, while minimizing impact to researcher operations.
Traditional high-performance computing (HPC) infrastructure is defined by fixed services closely aligned with hardware layouts.
In contrast, a software defined infrastructure for high-performance computing (SDHPC) is responsive and flexible, enabling in-flight upgrades with reduced researcher impact.

Traditional HPC use centered on text-based, command-line  interactions where users connect via SSH to dedicated login nodes to execute batch-oriented research workflows @Lonvick2006.
The development of immersive web interfaces to HPC systems provided a friendlier user experience, broadening access.
Among these interfaces, Open OnDemand (OOD) has emerged as the most successful @Hudak2018.
Now a fundamental component of many HPC systems, OOD has aligned user expectations with the self-directed experience of cloud-native applications.

OOD owes its success to a reduction of cognitive load for researchers.
The reference deployment of OOD provides access to several interactive, web-native applications.
Jupyter Notebooks, R Studio, and a VNC desktop capable of presenting native, windowed applications, are all available behind a simple resource request form.
Researchers start by engaging with their HPC system via the browser using familiar web applications and, when they are ready, can migrate to batch tools at the command-line.
Implementation details, such as submitting a job script to the scheduler, are hidden behind the software-defined abstractions of the resource request form and application definition.
Because OOD hides these details, researchers can more readily incorporate HPC into their research workflows.

OOD operates by mapping browser interactions to per-user web servers.
When a user connects to the HPC system via HTTPS, the OOD server spawns a web server process running under their account identity, as defined on the HPC system.
Whenever the user executes actions on the web site, processes started by the web server run within their identity context.
The net effect is that user engagement with the HPC system is shepherded by software definitions—identity, operating system rules, OOD application definitions, and the scheduler.
User interaction via OOD is governed by the same permissions enforced by the operating system on all the user's processes, whether started trough OOD or traditional command-line accessed via SSH.

OOD transforms HPC into a web-native user experience.
SDHPC transforms the HPC system into a cloud-native application.
Application routers for SSH and HTTPS enable us to modify the HPC environment for arbitrary subsets of the user community.
Identity-based routing requires that the application routers authenticate users in order to direct their connections to the correct resources.
Web authentication is handled by a web single sign-on (SSO) enabled Apache reverse-proxy that can route users based on group membership.
SSH authentication and routing is handled by sshpiper, an open-source SSH proxy built on top of the Golang SSH package @Lian2025.
We contributed an extension that enables routing based on group membership.

The application routers introduce an opportunity for horizontal scalability, further enabling SDHPC to deliver cloud-native infrastructure capabilities.
Continuous integration and continuous deployment (CICD) methodologies enable greater scaling of cloud-native infrastructures @Ugwueze2024.
To leverage the opportunity, we created a GitLab CI/CD workflow to build and deploy our application routers and OOD web services @gitlab-cicd.
Our SDHPC CICD workflow ensures we can deliver features and bug fixes through regular, reproducible, version-controlled deployments.

In this work we document SDHPC, a cloud-native framework for managing interaction with HPC clusters.
We introduce a Research Computing System (RCS) model that describes a coherent infrastructure for building scientific applications.
RCS provides both the HPC services that are the focus of SDHPC and a development platform that supports cloud-native automation.
We decribe the implementation of SDHPC and the cloud-native automation used to build and operate these services.
We discuss data and data-center migrations that served the  motivating use case for SDHPC development.
We provide insights on SSH application router performance and conclude with reflections on the utility of SDHPC and future directions.

(related-work)=
# Related Work

<!-- establishing value of OpenStack -->
The success of research computing activities depends on access to information technology (IT) infrastructure: compute, storage, and networking.
OpenStack, an open source cloud computing platform, provides software defined control over infrastructure @openstack2025.
IT staff can guard access to the physical systems running OpenStack to ensure secure operations while democratizing access to resource abstractions that enable developers to build applications.
Deploying OpenStack for research infrastructure decouples IT operations and research software development.

<!-- things what use openstack -->
The NSF-funded Jetstream2 project is a cloud computing environment available to researchers through the NSF ACCESS program [@Hancock2021; @Boerner2023].
The NSF ACCESS program is the successor to the XSEDE program which nurtured the development of national HPC resources to support research workflows @Towns2014.
Jetstream2 is built using OpenStack and provides cloud-native abstractions to  research applications that need compute, storage, and network resources.
Jetstream2 allocates capacity for CPU, GPU, and storage using ACCESS-granted service unit credits.
Jetstream2 demonstrates at-scale operations of OpenStack clouds and provides autonomous access to advanced hardware for the development of research workflows.

CRI\_XCBC is a legacy software project created by the XSEDE program.
It provides a working example of a deployable, extensible HPC stack that used Ansible playbooks to deploy an OpenHPC defined cluster infrastructure @CRIXCBC2024.
We maintain a site-specific fork of CRI\_XCBC that serves as our foundation for software defined cluster environments @Robinson2025.
We extended this code-base to include OOD web services as part of the Ansible-deployed OpenHPC cluster @Tripathi2020.
We further extend this Ansible code-base to support SDHPC in this work.

The "data center as a computer" model facilitates systems development, abstracting complex subsystems as the component in a personal computer @Barroso2019.
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

(sdhpc-cicd)=
## CICD Pipelines

SSH and HTTP application routers direct user traffic to login nodes, OOD instances, and the account app according to site operational goals.
This creates a coherent user experience for HPC access that is consistent across both HTTP and SSH endpoints.
Implementing these services as Infrastructure as Code (IaC) allows continuous development of platform features and control over the introduction of capabilities as they are deployed @Ramos2015.

Our CICD workflows are built using GitLab CI/CD @gitlab-cicd.
We use a dedicated GitLab repository to house the CICD pipelines, following the image factory pattern @Muse2023.
We created CICD pipelines with separate build and deploy phases.
Both phases leverage Ansible to construct relevant artifacts.
The RCS cloud subsystem provides the infrastructure for development, build, and deployment workflows @uabrc2025.
This enables the construction of comprehensive cloud-native workflows and facilates tight integration with campus HPC resources via direct connection to relavent provider networks.

Packer templates are used to construct VM images for the proxies, OOD and account app during the build phase.
These templates include extensions to interface with the OpenStack API and can readily work with other cloud providers.
We execute daily builds of our OOD image.
This ensures image construction remains viable and surfaces problems with build dependencies in a timely fashion.
The daily builds also support developement deploys to review feature improvements against our production HPC system.

For the deployment pipelines, we choose to directly invoke OpenStack CLI commands for implementation convenience.
We have used Terraform for the deploy phase of other system components not featured in this work and plan to migrate SDHPC artifact deployment to Terraform in the future.

The separation of build and deploy steps enables us to construct versioned VM binaries that can be used across development, test, and production environments.
The deploy steps focus on customizations that meet the needs of specific environments, providing late-binding hooks for dev, test, and prod clusters.

(sdhpc-applied)=
# SDHPC for Data and Cluster Migration

The SDHPC framework provides features that enable a variety of operational and service development scenarios.
We introduced SDHPC to maintain stable HTTP and SSH endpoint definitions for users during a multi-phase migration of HPC services in our RCS.

```{figure} images/AB_cluster.png
:label: ab_cluster
:width: 100%
:alt: Schematic showing boxes representing the application routers for HTTP and SSH with lines pointing to additional boxes that represent the login and OOD nodes of distinct clusters A and B.

An arrangement of the Software Defined HPC (SDHPC) framework to route users to distinct cluster environments based on their user identity and group membership. Group A web and ssh connections are routed to the login and OOD nodes of Cluster A and group B connections are routed to the logig and OOD nodes of Cluster B.
```

The SDHPC framework was developed in response to a confluence of events.
We were faced with vendor relationship changes, product licensing changes, product lifecycle transitions, data center power constraints, storage demand growth, and the general financial constraint of operating within annual budget cycles [@carlz-at-us.ibm.com2020; @Sedlmayer2020].

The GPFS high-performance storage for the HPC service needed to be upgraded to the latest supported version.
This required moving petabytes of data to a new GPFS implementation from a new vendor.
The new GPFS implementation tiers inactive files to a capacity tier implmented with Cephfs. 
This cost-effective design provides a unified file namespace for HPC applications, maximizes GPFS performance for active analyses, and maximizes capacity to retain data.

Power constraints in the on-campus data center drove decisions to consolidate all RCS compute resources in newly available high-powered racks at a nearby commercial data center @DcbloxHDcolo.
The compute resources include the HPC batch compute and its associated GPFS performance tier, VM cloud compute services, and container compute services.
The new facility provides sufficient power for planned growth of all RCS compute capacity.
The facility is integrated with the on-campus data center using dark fiber provided by the University of Alabama System Regional Optical Network (UASRON).
This configuration allows the on campus data center to continue hosting RCS Ceph storage and to provide the peering points for campus and R&D networks.

The SDHPC framework helps navigate complex requirements.
[Figure %s](#ab_cluster) highlights the HPC storage and data center migration use-case that drove its development.
Implementing the migration required reconstructing the HPC service in the new data center facility.
In order to avoid extended downtimes for the entire HPC community, we planned the migration around moving isolated subsets of users in batches.
We staged an initial footprint of HPC compute capacity as a separate cluster attached to the new GPFS platform in the new facility.

We designate Cluster A in #fig:ab_cluster as the HPC resources associate with the original storage platform and Cluster B as the HPC resources associate with the new storage platform.
This provides distinct environments onto which users can be mapped depending on their data migration status.
This approach allowes moving users and HPC capacity as their data migrations are completed.
The incremental migration also provided a natural load-testing ramp for the new storage solution.

The SDHPC HTTP and SSH application routers provide stable endpoints for all users, regardless of their migration status.
Moving a user is accomplished by designating their accounts as a member of group A or group B.
This group membership directs their connections to cluster A or cluster B, respectfully, depending on the location of account data.
The assignments are transparent to the end user.

To further simplify the user experience, we provide the same file system namespace and scheduler partitions in both cluster environments.
The file system namespace is made consistent by using apporpriate bind mounts.
The job submission experience is made consistent by using the SLURM job submit plugin to route user jobs to teh correct cluster compute nodes that are connected to the storage platform housing the users data.
The same group memberships are use to modify submitted jobs, tagging them with a feature to select appropriate nodes within a requested partition.

Keeping the user experience consistent during this transition minimizes the cognitive load on users by avoiding changes to their HPC workflows.
The migration workflow seeds user data to the new storage system with regular synchronization runs using dsync from mpiFileUtils @osti_mpifileutils.
When a user's migration from Cluster A to Cluster B is scheduled, their account is place on hold to avoid changes to their data.
A final sync from the source system is performed to capture any changes since the most recent seed point.
Their file namespace is stubbed into the new GPFS performance tier from the Cephfs capacity tier.
Their cluster assignment group is updated and their account hold is released.
From then on their group B membership ensures their HTTP and SSH connections are routed to Cluster B.
Each migrated user effectively experiences a blue-green deployment model [@Fowler2010; @Humble2013].
The user is cut over to the new production system when it is validated as operational for their account.

The RCS cloud service provides an ideal hosting environment for SDHPC operations.
We provision the application routers and cluster B's OOD node to an OpenStack project dedicated to HPC production operations.
The project is authorized to access campus and cluster provider networks made available to the project through the OpenStack SDN services.
This allows the application routers to accept user connections and route them to the appropriate cluster.
Furthermore, this virtualized infrastructure helps frame the functionality of application routers as traffic flow control agents for HPC connections.

The RCS cloud service is also a natural fit for CICD driven workflows.
Developer contributes to the image factory can be validated with integration pipelines.
New releases are delivered to production using the same pipelines used in development.
This cloud-native software development workflow enables rapid feature iteratation during product development cycles and reduces the friction between development and production environments.
Deployment of testable integration environments takes only a few minutes.
These rapid deployments lower testing barriers, increase test frequency, and lead to enhanced service quality.

(ssh-router-perf)=
# SSH Application Router Performance

We consider potential performance impacts of SDHPC through the introduction of cloud-native (VM-based) application routers.
HTTP reverse-proxies are a standard part of the OOD deployment and common for at-scale cloud deployments.
As such, SDHPC HTTP applicaiton router performance did not present any concerns and was not explictly studied.
In order to understand impacts on SSH performance, however, we measured data transfer throughput for a variety of SSH connection scenarios with and without the use of the sshpiper application routing.

:::{table} SSH Throughput Tests. All values in Megabytes per second. Results of SCP transfers of a 2 Gigabyte file to destination hosts, averged over thirty tests.
:label: table-sshperf
:align: center
:width: 100%


| Scenario | SCP Throughput to Target                    | Mean                   | Error Margin      | Confidence |
|----|---------------------------------------------|-----------------------|-------------|-------------|
|A| VM in Project | 369.33 | 7.92 | 361.42 -- 377.25 |
|B| Login node     | 119.80 | 7.11 | 112.69 -- 126.92 |
|C| Proxy via physical node | 98.45  | 4.81 |  93.64 -- 103.26 |
|D| Proxy via VM in Project | 84.44 | 5.32 |  79.13 --  89.76 |
|E| Proxy via sshpiper | 81.47 | 1.52 |  79.95 --  82.99 |
:::

In [](#table-sshperf) we show the throughput of SSH data transfers using SCP to copy a two Gibabyte file (base-ten) to a target node.
All tests originate from a VM in an OpenStack project.
All SSH components, except for the sshpiper application router, are implemented using standard OpenSSH packages provided by vendor distributions.
The results represent the performance of wired clients in different scenarios.
Scenario A, the originating VM is used to connect a VM local to the OpenStack project.
This provides a baseline for the maximum SSH performance achievable by the originating VM.
This is not performance we expect to replicate in other scenarios because all other tests are bound by the performance of the network throughput to the HPC login node's production interface, 10Gbps in the current environment.
Scenario B, is our baseline for the SSH performance to the production interface of the existing HPC login node.
Ideally, the performance of the remaining scenarios  would closely match this baseline.

The remaining scenarios each represent different SSH proxy configurations.
Each of these configurations imply the use of two SSH connections to reach the login node.
These are the results we consider directly comparable for SDHPC SSH application routing.
The proxy scenarios quantify the impact of the double SSH connection required to connect via a proxy to the destination login node.

Scenario C is standard SSH jump host configuration. An SSH connection is establish to the jumphost.
A second SSH connection is then establish from the client to the target login node via the jump host.
This configuration provides a baseline for proxied SSH connections by using SSH services running on physical hardware.
The main advantage of this configuration is the reduction in hypervisor induced SSH server overhead.
We used the production login node as the jump host and a cluster compute node a stand-in for a login node.
This SSH path mirrors the cluster topology that is used to support sshpiper, a front end SSH target directly connected "within" the cluster to a target login node.
We can see that the introduction of the second SSH connection has about an 18% performance impact on throughput.
This indicates that introduction of SSH application routing will impact throughput.

Scenario D represents the same SSH jump host configuration as Scenario C but replaces the physical front-end node with a VM running in the same OpenStack project.
This represents the intended cloud-native deployment for our SDHPC SSH applcation router but without introduction of the new Golang-base SSH implemenation in sshpiper.
This configuration introduces an additional 10% drop in performance over Scenario C.

Finally, Scenario E represents the production sshpiper deployment of the SDHPC SSH application router.
This allows us to understand the performance of the Golang SSH package leveraged by sshpiper to provide the SSH proxy functionality.
Camparing Scenario D and E, we see a slight increase in over head for this implementation.
The sshpiper SSH application router, however, enjoys a signfinicant reduction in the error margin over all other scenarios.
This suggests stable performance for interactive command-line use.
Although the sshpiper-based application router has lower throughput, we considered the stable performance ideal for interactive use. The throughput is acceptable for less demanding data transfer workflows, because we offer high-speed data transfer via Globus endpoints.
Additionally, sshpiper is the only solution that supports per-user routing of SSH connections.

A more detailed study of SSH application router performance is warranted.
It will be informative to understand the origin of performance impacts from pipelined SSH connections.
The sshpiper application router performance is not significantly below traditional SSH jumphost proxy scenarios in the same deployment configuration and could likely be improved with conventional performance tuning, e.g. using physical hardware instead of VMs.
Nonetheless, we can envision the deployment of highly scalable environments given this SDHPC SSH application routing functionality.

(conclusion)=
# Conclusion
<!--- note disaggration of ood and account app -->

We introduced application routing for SSH and HTTP connections based on user identity.
We leveraged on-site cloud computing resources in our RCS to built CICD pipelines for our application routers and OOD services creating a SDHPC framework.
This framework has facilitated a complex data and cluster migration project while minimizing user disruptions.

These new platform features help create a cloud-native HPC user experience.
Policy-based routing of user workloads maintains service availability in the face of significant changes to underlying infrastructure.
SDHPC has facilitated data migration for a major storage upgrade and will support additional improvements to the HPC system in the future.
Using flexible, modern paradigms like CICD creates testable features with reduced development time and reliable deployments.
Providing stable services that can be enhance through continuous development and capacity expansion is the goal of cloud-native platforms.

We are also expanding use of the image factory model to simplify introduction of new HPC services and reduce maintenance overhead.
Creating consistent CICD workflows for development and production 
IaC demands programmatic control over infrastructure.
Automated construction of systems improves their reliability.
Moving service construction to a cloud-native model is necessary to ensure the steps that are built and tested during development are valid for the production environment.
SDHPC is responsive to bug fixes and supports timely deployment of feature enhancements.

Data migrations are complex undertakings.
The longer data sits still the harder it becomes to move.
This migration experience is leading us to explore ways to operationalize data movement.
It is important to ensure that data always resides on the most appropriate storage resources.
The motivation for these ideas is to avoid having data pool on the oldest hardware simply because that is where it originally landed.
This approach should help avoid heavy lifts in the future when platforms are retired.
Effectively developing these data workflows requires automated processes that can implement placement based on site policy.
