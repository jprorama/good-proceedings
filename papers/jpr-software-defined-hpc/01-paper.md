---
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
abstract: |
  We introduce a software defined infrastructure for high performance 
  computing (SDHPC) that includes
  SSH and web application routers for user-based connection routing to endpoints based on individual user identities. We detail Continuous Integration
  and Continuous Deployment (CICD) pipelines used
  to build and deploy an infrastructure of application proxies that route user connections to appropriate login and Open OnDemand nodes based on their group membership.
  We demonstrate the utility of this infrastructure to our motivating use case of
  limiting user downtime during maintenance operations.
  We conclude with observations on functionality and highlight future directions.

keywords:
  - High Performance Computing
  - Sofrware Defined
  - Infrastructure as Code
  - CICD
  - Cloud
  - AI Infrastructure
---


# Introduction

Research computing has long focused on providing access to High Performance Computing (HPC) clusters.
The traditional HPC user experience has centered around access to a command line where all system interaction is managed..
Users SSH to a cluster login node from where the user submits batch jobs to a scheduler, conducts workflows, and organizes their data.
Many attempts have been to enhanced the user experience with web-based tooling to reduce learning curves and create a more immersive browser-based user experience.
In this space, Open OnDemand has emerged as the most successful web integration for HPC.
Over the past decade, Open OnDemnd (OOD) has come to dominate the HPC landscape as the de facto web experience and can now be seen as a fundamental component of any HPC cluster.
OOD has elevated HPC to a web-native experience and shaped user expectation for HPC to align with the self-directed experiences of cloud-native applications.

OOD's success stems not from displacing the traditional command shell but by enhancing the user experience to include web-native applications that sit naturally along-side their traditional cluster interaction.
A basic deployment provides access to Juptyer notebooks, R-Studio, and a browser-based VNC desktop capable of presenting any traditional GUI applicaiton, like Matlab or QGIS, within the browser.
All of these web-based applications are easily launched as jobs on cluster compute nodes through simple button clicks in the web browser.

The key aspect of the OOD architecture is transparent mapping of browser actions to per-user web servers that run applications under the native cluster identity of the user.
This allows all actions performed by the user on the cluster, whether via the web or the command shell, to run in the context of their account and adhere to a consistent security model enforced by the operating system across the cluster.
User's can only access cluster resources as dictated by their account permissions.

<!--- ref gridsphere and science gateways that were dedicated tools and that tarun or something that we looked at around 2017 -->

In order to facilitate operation of campus HPC as a cloud-native application, we built front-end services to manage connection routing for SSH and web endpoints.
These services introduce capabilities for horizontal scalability and enable us to affect changes in the environment on miminal subsets of the user community.
The front-end services act as application proxies that route SSH and web connections to appropriate login and OOD nodes based on a user's group membership.
This capability implies that the application routers authenticate users in order to resolve routing rules.
We built a simple Apache-based proxy that manages web single sign-on for users an routes them to their target OOD instance.
We extended ssh.piper, an open-source SSH router built on top of Go ssh, to include group-based user routing. 
<!--- we should show a picture of this here? -->

Operating cloud-native infrastructure at-scale is improved with development processes that leverage continous integration and continous deployment (CICD) methodologies.
To that end, we created a CICD workflow to build and deploy our application routing front-end nodes and the Open On Demand web services.
This workflow ensures that we can deliver features and fix bugs through regular deployments of this software-defined infrastructure.

In the next section we hightlight trends in the design of large-scale systems in industry and research that are driving evolution toward cloud-native HPC infrastructure.
In section [sec-tag] we document the design of our software-defined infrastructure to route user connections to desired endpoints and the CICD workflow used to deploy it.
In section [sec-tag] we discuss the initial use-case for this infrastructure and assess its utility.
We conclude the paper with a discussion of future directions.

# Related Work

Research generates data that must be processed, analyzed, and stored in order to derive scientific insight.
Information Technology (IT) infrastructure is fundamental to the creation of research applications that process data.
Access to infrastructure determines the success or failure of research workflows that depend on computing.
Cloud computing increases access to infrastructure by provide software defined abstractions for the compute, storage, and network resources needed to build applications.
OpenStack is an Open Source Software (OSS) platfrom that allows a site to encapsulate its physical IT hardware and present cloud-native abstractions for compute, storage, and network resources via software defind infrastructure.
This allows platform users to access software defined IT infrastructure to build applications using cloud-native development paradigms.
IT staff can guard access to the physical systems running OpenStack to ensure secure operations while democratizing access to resource abstractions that enable developers to build applications.
Deploying OpenStack to provide infrastructure for research applications provides autonomy for IT operations and research software development.

The NSF-funded Jetstream2 project is an at-scale cloud computing environment available to researchers through the NSF ACCESS program.
The NFS ACCESS program is the successor to the XSEDE program which nurtured the development of national HPC resources to support research workflows.
Jetstream2 is built using OpenStack to provide cloud-native abstract research applications that need compute, storage, and network resources.
Jetstream2 allocates capacity for CPU, GPU, and storage using ACCESS-granted service unit credits.
Jetstream2 demonstrates at-scale operations of OpenStack clouds and provides autonomous access to advanced hardware for the development of research workflows.

CRI\_XCBC is legacy software project created by the XSEDE program as working example of deployable and exstensible HPC stack that used Ansible playbooks to deploy and OpenHPC defined infrastructure.
We maintain a site-specific fork of CRI\_XCBC to as our foundation for software defined cluster environments.
We extended this code-base to include OOD web services as part of the Ansible-deployed OpenHPC cluster.
We leverage that solution in this work.

Datacenter as a computer is a warehouse scale computing paradigm that treats the data center a massive computer.
While not all computing systems are warehouse scale, the paradigm facilitates systems development by demostrating how computer system abstractions can simplify the organization of complex multi-site and multi-system infrasture deployments.
Using these abstractions allows operators, administrators, and users of the system to communicate and stratigize on specific functions or services without risking a loss of interoperability.
Such layered abstractions have been crucial to the success of global network initiatives.
We adopted this paradigm to organize our research computing infrastructure.

# Software Defined HPC

HPC clusters have long aligned with the principles of software defined infrastucture.
The original Beowulf cluster model deployed fleets of identical compute nodes under the control of a head node that also supplied the core infrastructure for HPC operations.
Today's IT infrastructure leverages these same approaches to provide scalable compute, storage, and network capacty.


## Research Computing System

<!--Buidlding a cloud native experience. -->
In order to align our campus research computing infrasture with scalable, cloud-native solutions, we adpopted the data-center-as-the-computer model to create a coherent platform delivering compute, storage, and networking to research applications.
We identify this platform as a Research Computing System (RCS), see [Fig. %s](#rcs_architecture).

```{figure} images/RCS Architecture.png
:label: rcs_architecture
:alt: RCS architicuture following design of warehouse scale machines. Shows batch, vm, and container compute modalities connected to file, block, and object storage modalities via interconnect and peering to external networks.

RCS architicuture following design of warehouse scale machines. Shows batch, vm, and container compute modalities connected to file, block, and object storage modalities via interconnect and peering to external networks.
```

RCS supports HPC batch, virtual machine, and process container compute modelities.
The compute services integrate with storage systems that provide high throughput parallel file systems, block devices, and object storage.
Compute and storage are integrated with a network fabric the provides bandwidth for both east-west and north-south traffic flows.
The RCS interfaces with external networks peering to both campus and R&E network via a Science DMZ segment.

HPC batch computing is implemented via a traditional, stand-alone compute cluster.
At our site this is implemented with Bright Cluster Manager managed compute nodes connected to a GPFS parallel file system via a dedicated Infiniband network.
The move to this RCS architecture positions the HPC infrastructure along-side platforms that provide VM and container abstractions to applications.
The VM compute capacity is implemented via an on-premise OpenStack private cloud which provides a comprehensive software defined infrastructure enabling direct control over compute, storage and network resources needed to build and operate research applications.
We use this infrastructure to host the application routers, Open OnDemand web services and CICD workflows describe the in following subsections.
Container compute capacity for RCS is implmented via Kubernetes.
The current work does not leverage microservices so this subsystem is not discussed further.

In addition to GPFS parallel storage for HPC workloads, our implementation of RCS leverages Ceph storage subsystems to deliver block and object storage services to OpenStack and Kubernetes based workloads.
Ceph additionally provides public S3 endpoints for user data and Cephfs as a tiering backend for GPFS to manage cost.
Research and education (R&E) network peering is provided by a Science DMZ segment (SciDMZ) that leverages Globus as the I/O interface for moving data to GPFS and allows users to manage other storage services via additional Globus connectors.
The campus peering connection provides network access to all other services in the RCS simplifying compliance with campus IT policies.
Enhancements to the RCS network interface are under consideration to expand services on the SciDMZ.

## Application Routers

OOD provides a web-native experience for HPC.
In order to operate the full HPC stack under this model, it is necessary to have control over all aspect of user interaction with the HPC platform.
This includes the traditional, secure shell (SSH) interfaces to the system.
While OOD does provide a web-based solutions for secure shell access via web ssh, many users depend on traditional SSH clients that directly connect to port 22 using the SSH protocol.
In order to provide a unified experience with the HPC platform to all users, it desirable to manage SSH connections with infrastructure components that share the same functionality of HTTP service.
In particular, HTTP connections are readily routed to nodes that provide specific services to the user.
For example, while OOD is delivered as an intgrated single-system solution, it is composed of standard web application components that route authenticated user connections to their own dedicated web server that provides access to HPC resources.

If a user has no corresponding HPC account, OOD can be configured to direct the connection to an endpoint designed to handle this scenario.
Many environments, provide a destination service that enables user account creation.
We implemented an Account web application that allows users with a defined institutional relationship to create their own HPC account.
The account app prompts the user to create the account.
After the user submits the request, the account app interacts with standard HPC system services, exposed as RabbitMQ services, to create the user account.
Once the account is created the user connection is redirected to OOD, this time passing the account check and passing through to their per-user nginx server where they can interact with the OOD services.
The self-directed account creation request typically takes less than fifteen seconds to complete.
This capability ensures authorized users experience little more than a slight account provisioning delay the first time the access the HPC system.

Because user authorization is a standard part of OOD access, the user can also be redicted to their account services page as needed to support HPC operations.
We define simple states like good\_standing, certification\_required, and account\_hold to facilate operations.
Only users with accounts in good\_standing are allowed to interact with OOD.
Other states force the user's web connections to the account app so they address any requirements to restablish their good\_standing state.
The account\_hold state provides direct control to support personal over individual user access to HPC resources for service events or other user engagements.

This request routing is standard fare for web-native application.
It enables the creation of rich user experiences that can be consistently implemented across the platform.
We have further enhanced OOD's implicit account behavior by introducing a dedicated front-end HTTP application router that can control web user access based on additional attributes like group membership.
Just like OOD, this application router requires authenticated connections.
Once authenticated via web SSO, the full spectrum of user account attributes can be used to route connections based on state, group membership, or individual identity according to site policies.
We add dedicated reverse proxy stanzas to the router in order to control the destination.
This enhanced web connection routing enables the creation of rich user experiences that align with site service delivery goals.
Because HTTP routing is long established and the requirements for our enahancements are easily satisfied with standard Apache web server features.

HPC is, however, not an exclusively web-based experience.
The SSH protocol is the dominate user interface to HPC.
In order to provide a consistent user experience accross HTTP and SSH endpoints, it is desirable for both services to respond similarly to site policies based on a users account status.
For example, a user whose account is in a hold state should be notified to access the account app via their web browser and prevent from further SSH interaction with the site.
We have accomplished this with Group Match stanzas in our OpenSSH server configuration.
More sophisticated routing of valid SSH connections is not possible.
To accomplish rule-based routing based on user attributes, we implemented an SSH application router using sshpiper, a reverse proxy for SSH built on Golang's SSH library.
We contributed an enhancement to sshpiper that allows a user's group membership to be use in the routing rules.
This SSH application router enables user connection routing to preferred login nodes based on site policy or operational requirments.
The sshpiper routing works by terminating the client-side SSH connection and establising a second internal connection to the desired login node endpoint.
For password-based authentication, the users credentials are passed to and validated by the login node endpoint.
For key-based authentication, the user authentication is managed using internal cluster keys shared via the file system.
This uses the same infrastructure already in place on the HPC system that allows users to transparently access compute nodes running their jobs without prompting for additional authentication.

We descibe our motivating use-case for these application routers in the Experiment section.

<!--- so we can actually say we have the openstack as a front end to our hpc
the advantage here is that we can use this same front end regardless of where a physical cluster may actually be located
openstack gives us cloud-native tooling and sdn to route traffic to desired destinations -->

## CICD Pipelines

<!--- that should really about working from a defined image and customizing it via the user-data section to connet it to the runtime environment.

this is really where a picture could come in handy
we build images via packer with possible external repo

we build deploys from defined images
add hooks for late binding to target env
this is doen via user-data and direct openstack cli

this infrastructure relies on gitlab runner for packer and other deploy tasks
-->

With the creation of SSH and HTTP application routers, we are able to direct user traffic flows to login nodes, OOD, and the account app according to site operational goals.
This supports creating a coherent user expereince for HPC access that is consistent across both HTTP and SSH endpoints.
Implementing these routers as software defined infrastructure (SDI) allows continuous development of platform features and control over the introduction of capabilities as they are deployed.

Our CICD workflows are built using GitLab CI/CD.
We use a dedicated GitLab repository to house the CICD pipelines, following an image factory pattern.
We created CICD pipelines with separate build and deploy phases based on the requirements of specific use-cases.
Both phases leverage Ansible to construct relavent artifacts.
The RCS cloud subsystem provides the infrastructure for development, build, and deployment workflows.
Using an on-premise cloud enables the construction of comprehensive cloud-native workflows and allows tight integration with campus HPC resources.
Packer templates are used to construct VM images for OOD and account app componentes.
We choose to directly invoke OpenStack CLI commands during deployment as a convenient optimization.
We have used Terraform for the deploy phase of other system components not featured in this work.
The separation of build and deploy steps enables us to construct versioned OOD VM binaries that can be used across development, test, and production environments.
The deploy steps focus on customizations that meet the needs of the specific environments, providing late binding hooks and other customizations for specific clusters.
We descibe the build pipeline in the next subsection and the deploy pipeline in the following.

### Core Infrastructure Builds

We build OOD VM images that include all dependencies of it's software stack and additional compenents for integration with our local HPC systems architecture.
The images are constructed using the legacy CRI\_XCBC Anisble repository.
CRI\_XCBC project was created under the NSF XSEDE initiatve to provide an infrastructure-as-code (IaC) framework for HPC deployments.

We adopted CRI\_XCBC to provide developer instantiated HPC platforms to implement our original OOD deployment.
OOD is challenging for early career developers to learn, test, and extend without an HPC cluster with which it integrates.
In order to write systems applications, developers must be granted complete authority over the software stack that includes the HPC system stack, batch scheduler, and other core platform services.
DevOps workflows are easier to construct when development and production infrastructure are isolated from each other.
Developers need to be able to iterrate repeatedly over the build and deploy pipelines.
Providing developer access to production resources is undesireable and works against the goal of creating repeatable builds and deployments.
Deploying disposable HPC clusters (dev clusters) as part of the development workflow aids construction of SDI and professional growth.

We forked and extended CRI\_XCBC to include Ansible roles to build an OOD image.
Constructing OOD within the CRI\_XCBC framework provided an HPC platform with which OOD integrates.
This also provided the necessary components to develop our web-based Account app that enables self-service HPC onboarding for users.
The account app provides a web UI that invokes native HPC account creation services via a RabbitMQ message bus.
Over time we abandoned use of the OpenHPC IaC elements from the CRI_XCBC framework.
We now rely on stand-alone Heat-based deployment of dev clusters built using Bright Cluster Manager (BCM) and the NVIDIA (formerly Bright Computing) Easy8 developer-focused cluster framework.
We use this cluster management infrastructure on our production cluster.
This move was necessary to support construction of BCM-specific drivers for our RabbitMQ based account creation services.

A dedicated image factory repository uses GitLab CI/CD to construct the OOD and account application images by ingesting external Ansible rules from our CRI\_XCBC fork.
The Ansible framework is executed during the Packer image build.
The images are stored in the OpenStack Glance image repository and made available to developer and production projects.
This approach ensures we can track and test the latest changes to the IaC repositories and povide versioned images for development and production use.
To that end, we execute a daily build of our OOD image to ensure the construction remains viable and surfaces problems with build dependencies in a timely fashion.
Long delays between builds can otherwise lead to unexpected failures when it becomes necessary to update configurations in response to bugs or changes in operation.
The daily build also supports deployment of the development head to review feature improvements against our production HPC system.
We use this approach to enable the introduction of new OOD applications through the addition of an app-specific Ansible role in our CRI\_XCBC repo.

<!--- following may not be necessary 
While still rooted in CRI_XCBC, OOD and the Account app have become the only components we build out of this IaC framework.
We run the OOD build and deploy pipelines each day in order to maintain confidence in our ability to construct a complex VM image with many dependencies.
We use the daily builds of OOD as the foundation for multiple deployments.
We provide live deployment of the current development head to allow feature and bug fixes to be explored.
-->

### Deployment of HPC Services
<!---  this is the interface for modern hpc + globus, we do not address globus routing in this work. -->

We created deploy pipelines for the HTTP and SSH application routers and the OOD and account application web applications.
The application routers are built from stock OS images, currently Alama9, that are customized during deploy with Ansible rules to include the HTTP and SSH servers configured to authenticate the user and then route to the correct backend service.
We deploy the HTTP and SSH on services to separate VMs for ease of construction and to facilitate service scaling if needed.
The HTTP application router is a standard Apache reverse proxy that includes SAML-based WebSSO using Shibboleth service provider components.
Once the user identy is know, an LDAP lookup against the HPC cluster's account database to gather group memberships.
We use membership in specifc groups to choose the target OOD backend.

The SSH application router is built using sshpiper, a Golang SSH proxy server implemented on the Golang SSH library.
The sshpiper server is built from source with Ansible during deploy.
Additional Ansible tasks are used to configure the host to integrate with the HPC cluster system environment.
The key parts of this configuration include system level user account integration to support user account validation for the SSH protocol.
User home directories to be mounted via NFS to support standard user-level ssh configuration, e.g. ordinary management of the authorized_keys files for key-based authentication.
The SSH application router operates much like the familar login node on any cluster with the exception that all SSH connections are passed through the application router.
No user shells are run on the application router.
The SSH router transparently establishes a second ssh connection to the internal termination point for the desired login node, where the user shell is started.
All user SSH based user interaction with the cluster will occur from the login node that hosts the shell process.

We deploy the OOD and account apps from the VM image created during the build phase described above.
Using prebuilt VM images for OOD results in shorted deploy time.
We use the concept of late-binding to the customize the deployed image with the configuration required to interface with a specific cluster environment.
An OOD node has cluster integration expectations similar to login nodes.\
We characterize the late binding as interfacing the node with the clusters name resolution (DNS), account lookup (LDAP), file namespace (NFS), and batch job submission (Slurm).
The Ansible coded deploy steps adjust the pre-built image to interface with these necessary cluster services.

In all cases, we initiate the Ansible deploy steps during node instanciation using cloud-init user-data scripts.
These scripts are injected into the node by the cloud platforma and executed on the node during instance startup.
The deploy pipelines directly execute commands that generate the user data script which is then passed to the invoked openstack CLI commands that instanciate the node.
After the node is intantiated, the pipeline assignes public floating IP address for external and cluster-internal networks to enaable the node to accept user connections and interact with cluster services.

We use a blue-green deployment model where the node for a service is deployed using a non-production endpoint address.
Operation of the newly deployed node is validated in the blue state.
If the service is operating as expected, the services public IP address is moved to the newly deploy node.
This simplifies our migration to newly deployed services and avoids service interruption because the software define network service of the cloud platform provide a clean transtion for traffic flows.

There are many ways to handle this deployment cut of over.
Our current approach use manual IP address cutover after accept test are performed.
More advanced deploy pipelines could include automatic validation modeled after test-driven development.
These automated build and deploy of our CICD pipelines help ensure we can reliably and continuously release the latest improvements to the user experience.

<!---
This is how it's deployed

- the app routers are built directly in gitlab ci as artificts contstructed directly on the openstack cloud infrastructure
- show construction of cicd pipelines and how the produce their artifacts
-->


# SDHPC for Data and Cluster Migration

```{figure} images/AB_cluster.png
:label: ab_cluster
:width: 100%
:alt: Software Defined HPC framework used to manage a two cluster deployment where a single visible entry point leads to two differrent cluster fabrics, transparent to the user experience.

An arrangement of the Software Defined HPC (SDHPC) framework to route users to distinct cluster environments based on their user identity and group membership. Group A web and ssh connections are routed to Cluster A and group B connections are routed to Cluster B.
```

The Software Defined HPC (SDHPC)_framework provides features that enable a variety of service developement and operations workflows.
We introduced SDHPC to maintain front-end application routers for our traditional campus HPC user interfaces to support a multi-phase migration of data and infrastructure in our RCS.
The framework was developed in response to a confluence of events.
We were faced with vendor relationship changes, product licensing changes, product lifecycle transitions, data center power constraints, storage demand growth, and budgetary constraints.
The GPFS-based, high-performance storage tier for the campus HPC system needed to move to the latest supported version.
Due to evolutions in the vendor landscape, this neccessitated moving petabytes of existing data to new hardware from a new vendor.
Power constraints in the on-campus data center drove decisions to consolidate HPC CPU resources alongside other RCS compute resources in newly available high-powered racks at a nearby commercial data center already hosting our HPC GPU resources and the cloud and container components of RCS.
The new facilities provided sufficient power to support anticipated growth of our compute capacity across RCS.
This consolidation would also return HPC operations to a single Infiniband segment serving all HPC compute resources with access to the updated GPFS platform, which includes an NVMe performance tier to support I/O throughput demands for emerging AI workloads.
The compute node consilidate freed on-campus data center space to allow expansion of the capacity tier storage provided by Ceph.
This growth was designed to leveray annual operating budgets through multi-year infrastucture improvement cycles.
The approach allowed us to maximize GPFS performance for HPC applications and maximize storage capacity for large data sets a unified namespace by leveraging policy-based tiering of infrequently used data to a more granular, lower cost storage platform powered by Ceph.

The SDHPC framework supports balancing competing priorities.
[Figure %s](#ab_cluster) shows our initial use-case to migrate the HPC user community from the original GPFS storage platform to the new, tiered solution.
Implementing the storage migration required reconstructing the RCS HPC subsystem in new data center facilities.
In order to avoid extended downtime for the entire HPC community to accommodate data movement and system relocation, we created an initial footprint of HPC nodes as a seperate cluster attached to the new GPFS platform.
We designate Cluster A in the figure as the HPC resource assocate with the original storage platfrom  and Cluster B as the HPC resources associate with the new storage platform.
This approache facilitated incremental movement of user and HPC capacity as data and data center migritations completed.
It also provided a load testing ramp for the new tiered storage solution.

We were able to move subsets of the user community by designating users as members of goup A or group B to direct their connections to cluster A or cluster B, respectfully, depending on the location of their data.
The assignements are transparent to the user.
The SSH and HTTP endpoint names remain the same, with application connections routed to the appropriate servers.
The file system namespace is maintained across storage systems with appropriate bind mounts.
Job submissions remain the same, with resource scheduling modified using SLURM a job submit plugin that routes user jobs using the same group names to compute nodes connected to the appropriate storage platform housing their data.
Keeping the use experience consistent during this transition, minimized the cognitive load on users, avoided requirement to change workflows, and furthered the creation of a managed HPC user experience that facilitates future changes.
Each subset of migrated users effectively experiences a blue-green deployment model.
Their connections are cut over to the new service when their data migration is validated.

The cloud services of the RCS platform provide an ideal hosting environment for this infrastructure.
The comprehensive software defined infrastructure provided by OpenStack enables the creation of cloud-native development and operations workflows.
We provisioned the components for application routing and cluster B's OOD node in an OpenStack project dedicated to HPC operations.
The project is authorized to access campus and cluster provider networks made accessible through OpenStack SDN services.
This allows the services to accept user connections that can be routed to the desired cluster nodes.
We route users of cluster A to it's existing login and OOD services.
We route users of cluster B to a newly provisioned physical login node and VM instance OOD, hosted in the same project.
We chose to retain a physical login node for Cluster B to avoid performance scaling concerns and because of the ease with which these can be deployed using the traditonal cluster managers.
The cloud-based applicaition routers and OOD instance have access all necessary cluster services via the cluster provider network.
We use NFS to share the storage namespace with the SSH application router and OOD VM, as opposed to native GPFS clients as is the case with physical nodes connected to respective InfiniBand fabrics of the cluster environment.
The storage workloads for sshpiper routing and OOD interaction are relatively light and have not presented performance issues using NFS.

The cloud hosting envirnoment is also a natural fit for CICD driven developement and production workflows.
A self-hosted GitLab instance provides CICD tooling to develop and deploy the SDHPC into production using a dedicated hpc-factory project.
Developers contribute improvements to the hpc-factory which are validated with integration pipelines.
New releases are delivered to production in a timely manor using deployment pipelines.

This provides familar software development paradigms that enable contributions from a broader community.
It takes time to learn the full spectrum of HPC operations.
Seasoned professionals are responsible for daily cluster operations and often do not have time for relatively minor updates to non-core services.
Enabling early career developers to focus on more narrowly scoped services and successfully contribute features.

The most significant advantage of this approach is to reduce the friction between development and production environments.
Traditional approaches deploy ancillary HPC services to physical hardware in close proximity to the cluster.
This introduces a technology gap between how production nodes attached to the HPC fabric are managed versus how these services are operated during developement.
That friction frequently leads to delays in updating services like OOD because they are deployed with to dedicated hardware with manual steps excuted by core operations staff.
Reducing this friction increases the velocity of releases and allows the HPC platform to more easily adapt to user requirements. 
Another advantage is that the same services can be deployed to any cloud environment which supports the construction of HPC environments on the most cost-effective platforms.


<!--- 
conclusion can mention the complexity of the data movement as separate work
this does acheive our ability to fluidly move people and projects
conclusion can note the side effect is more autonomy of science engagement which is helping drive our goal for end-user managed infrastructure.
-->

# Conclusion

Infrastructure as code demands programatic control over infrastructure.
Automated constructuion of nodes ensures they reliably contain appropriate components and configurations to implement the service.
Programatic control of infrastructure ensures services are properly integrated with the production environment.
These controls are common during development.
Ensuring the same experience for production reduces deployment friction and leads to improvement in the user experience.
Moving services to a cloud native model is necessary to ensure the steps that are built and tested during development are valid for the production environment.

<!--- note disaggration of ood and account app -->

We introduced user-based application routing for SSH and HTTP connections to the HPC environment.
We leveraged on-site cloud computing resources in our RCS to build CICD pipelines for our applicaiton routers and OOD services creating a SDHPC framework.
This framework has facilated a data and cluster migration project while minimizing user disruptions.
The SDHPC is responsive to bug fixes and supports timely deployment of feature enhancements.

Data migrations are complex undertakings.
The longer data sits still the harder it becomes to move.
This experience is leading us to explore continuous data movement to reduce the dwell time and dependency on specific storage hardware.
We are also expanding use of the image factory model to simplify introduction of new HPC service and reduce maintanence overhead.
By creating consistent CICD workflows between development and production to deploy application routers we have moved our HPC cluster to a cloud-native orientation that provides comprehensive control over the user experience.
