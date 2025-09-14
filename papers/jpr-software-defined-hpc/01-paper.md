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


<!--- 

related work is data center as computer ideas
jetstream2
and maybe micro services?
maybe ssh.piper
cri_xcbc?

-->

OpenStack for on-site cloud-native infrastructure.
- it is a software defined data center infrastructure
- advanced software defined infrastructure across compute, storage, and network
- in a typical deploy, it overlays SDN capabilities across an L2 fabric and bridges the internal SDN stack with traditional site networks.
- advanced deploys can integrate the SDN stack provided by neutron and ovn to control physical devices via openVSwitch and controller devices? can't recall words
- supports L3 to the host

Jetstream2 for at-scale cloud operations
- advanced OpenStack cloud available through NFS access.
- demonstrates at-scale operations and provides direct access to advanced hardware through access allocations

CRI_XCBC
- legacy software fabric of the XSEDE project as an early working deployement of a full HPC stack using Ansible
- provides our foundation for working with cluster environments
the CRI_XCBC project's Ansible code repository.
This reposity was built to deploy an OpenHPC cluster with enhancements intended to simplify workload migration between local HPC cluster and national resources.

Datacenter as the computer


# Software Defined HPC

HPC clusters have long aligned with the principles of software defined infrastucture.
The original Beowulf cluster design deployed fleets of identical compute nodes under the control of a head node that also supplied the core infrastructure for HPC operations.
Today's infrastructure leverages these same approaches to provide scalable compute, storage, and network capacty.


## Research Computing System

Buidlding a cloud native experience.
In order to align campus research computing infrasture with scalable, cloud-native solutions, we adpopted the data-center-as-the-computer model to create a coherent infrastructure delivering compute, storage, and networking to research applications.
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

HPC batch computing is built upon a traditional, stand-alone compute cluster.
At our site this is implemented with Bright Cluster Manager managed compute nodes connected to a GPFS parallel file system via a dedicated Infiniband network.
The move to this RCS architecture positions the HPC infrastructure along-side platforms that provide VM and container abstractions to applications.
The VM ompute capacity is built upon OpenStack which provides a comprehensive software defined infrastructure that enables direct control over the compute, storage and network resources  need by research applications.
We use this infrastructure to build and operate the application routers, Open OnDemand web services and CICD workflows describe the in following subsections.
Container compute capacity for RCS is provided by Kubernetes.
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
We define simple states like good_standing, certification_required, and account_hold to facilate operations.
Only users with accounts in good_standing are allowed to interact with OOD.
Other states force the user's web connections to the account app so they address any requirements to restablish their good_standing state.
The account_hold state provides direct control to support personal over individual user access to HPC resources for service events or other user engagements.

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

With the creation of SSH and HTTP application routers, we are able to direct user traffic flows according to our site operational goals.
This allows us to create a coherent user expereince for HPC access that is consistent across both HTTP and SSH endpoints.
Implementing these routers as software defined infrastructure allows continuous development of platform features and control over the user experience as new capabilities are deployed.

### Core Infrastructure Builds

We build our automated OOD deployments on VM images constructed using the legacy CRI_XCBC Anisble repository that provides an Infrastructure as Code (IaC) framework for site HPC deployments.
The motivation for continuing construction on this foundation was to provide developer instantiated platforms that support operation of OOD.
An OOD deployment is difficult to learn, test, and extend without an HPC cluster with which it integrates.
DevOps workflows are easier to construct when development and production infrastructure are isolated from each other.
Developers need to be able to iterrate repeatedly over the build and deploy pipelines.
In order to write systems applications, developers must be granted complete authority over the software stack that includes the HPC system stack, batch scheduler, and other core platform services.
Providing developer access to production resources is undersireable and works against the goals to create repeatable builds and deployments.

Extending CRI_XCBC to include OOD inherently provided an HPC platform with which OOD can integrates.
It also provided all necessary components to develop a web-based, self-service Account app that provides onboarding for users and invokes native HPC account creation services.
Over time we have abondonded the use of the OpenHPC IaC elements and instead rely on a stand-alone Heat-based deployment of a cluster using Bright Cluster Manager and Bright's (now NVIDIA's) Easy8 developer focused cluster solution.
This was ultimately necessary to support construction of Bright-specific drivers for our RabbitMQ based account creation services.

While still rooted in CRI_XCBC, OOD and the Account app have become the only components we build out of this IaC framework.
We run the OOD build and deploy pipelines each day in order to maintain confidence in our ability to construct a complex VM image with many dependencies.
We use the daily builds of OOD as the foundation for multiple deployments.
We provide live deployment of the current development head to allow feature and bug fixes to be explored.

### Deployment of HPC Services
<!---  this is the interface for modern hpc + globus, we do not address globus routing in this work. -->

This is how it's deployed

- the app routers are built directly in gitlab ci as artificts contstructed directly on the openstack cloud infrastructure
- show construction of cicd pipelines and how the produce their artifacts

for our ssh proxy we use ssh-piper
our https applications are proxied with a simple Apache proxy with integrated SSO.
the SSO lets us embed identity into applications by default

the ssh proxy is responsible for routing ssh connections across an infrastructure of user login nodes to the batch computing system
user SSH remote-shell connections are routed to an available login node based on the users identity and system settings.

likewise the https proxy is responsible for routing web application across an infrastructure of user owned webservices.
the http connections are routed based on the user's identity and systems settings

we built a continuous integration and continous deployment (CICD) workflow to facilite maintaining avaialbility of current software releases
the initial focus was moving our deployment of the ondemand web appliction to the cloud VM abstraction
the baseline infrastructure model of a virtual machine running on a cloud platform that gives developer access to control infrastructure
this access is the foundation for maintaining releases with infrastructure as code practices.

we built workflows to maintain the key parts of our user interface
we started with ondemand because web native apps can be made to flexibility define the user interface
our first workflow did a nightly build and deploy of an ondemand instance that incorporated the latest commits of infrastructure codebase
this nightly instance is operational and available to users who prefer or depend upon our latest updates.
the nightly instances serves as a canary deploy for ondmeand
build or deploy failures of the nightly instance provide timely feedback on issues with our infrastructure codebase.
a complex application stack has many dependencies all of which can lead to unexpected errors
frequent deploys help surface issues with the applicaiton build and the runtime environment
the ability to observe development changes in a production context provides timely feedback on newly added  application features to ensure they effectively address user need.

also like many sites, we continue to offer SSH access to the batch system via a login node that offers full control over their batch workflows
-- a login node is the classic HPC interface where the user can arrange t their work and schedule th
ssh connections are hard-wired to specific systems in order to retain access to the process state of the user's shell
due to their tight integration with the cluster's compute environment, login nodes are often deployed as a minor varient of a compute node that doesn't run batch jobs on hardware identical to the other compute nodes.
this is a practicle approach that simplifies job development by ensure the user environment during interactive use is identical to the environment during batch use.

this node-specific deploy can make ssh infrastructure feel very static compared to the flexible routing inherent to web architectures,

placing the HPC ssh service on an equal footing with web applications facilitates a uniform approach for maintaining the HPC interface.
we use ssh.piper to route traffic....

web application and login nodes
the adoption of the cloud VM abstraction for the entire user interface
this

the knightly effort gave us a cicd
then we built a richer cicd to deploy a number of different systems
the key additional systems were http and ssh proxy
they gave us the ability to do canary deployes of all parts of our cluster, not just the ondemand

we did this by separating the physical, compute-derived-login nodes from their function of terminating user SSH connections
introducing an SSH application proxy that, like the web application proxy, is identity aware and can route connections based on user-specific settings.
specific users can be transparently routed to sepecifc login nodes
this transparent ssh proxy is implemented with ssh-pipe a flexible proxy with based on go-ssh with frequent releases that address bugs and feature requests.

we created a build and deploy pipeline for the ssh proxy to validate its infrastructure code using the same model in use for ondemand.
a cloud VM is built and deployed facilitating review of the current state of the proxy in the production context
with the latest release in place and validated, we follow a blue-green deployment strategy to direct ssh traffic to the new instance.
as a proxy, it continues to route users to their designated login nodes.

the final build and deploy pipeline is used to constrcut the ondemand applicaiton proxy
moving the https termination and user authentication of ondemand to the application proxy allows us to also route specific users to sepcific ondemand instances

with the automated build and deploy of our CICD pipelines, we ensure we can reliably and continuously release the latest improvements to the user experience.

# Experimental setup

this is the experiment
this doesn't need to cover how the data is moved
that's really opaque to the deployment
we can just say we can place accounts on hold, do final sync, and route the user to their new space
conclusion can mention the complexity of the data movement as separate work
this does acheive our ability to fluidly move people and projects
conclusion can note the side effect is more autonomy of science engagement which is helping drive our goal for end-user managed infrastructure.

our motivation for this infrastucture is to facilitate the forward migration of our user community as our infrastructure evoles and adapts to emerging requirements.
our rcs has subsystems that provide rich abstractions over infrastruture out of the box
the VM and container compute systems are very mature.
they provide user access to infrastructure that operates like mainstream cloud native platforms.

our hpc batch compute system is also very adept at evolving with application demand
we maintain multiple  generations of hardware that are made available to the batch system.
the batch computing systems strenth is that access to these resource is coordinated via the batch system
users request the specific hardware they need to run their applications and on the systems with that hardware
our most demanding evolationary application domain frequently  demands GPU capabilities
and we invest in systems to address that compute demand

we used the identity based routing capabilities to facilate migrating user data across a version and vendor upgrade of our primary HPC storage subsystem.
we created a new cluster environment configured to use the new storage subsystem.
both the existing and new cluster environment has a dedicated login node for SSH sessions and dedicated ondemand node for web applications that integrated with the the existing and new storage subsystem, respectively.

each cluster environment is tied to specific storage subsystems, therefor users need to be routed the existing or new cluster environment depending on where their data is located.
the user data migration worfkow begins with all user data located on the existing storage subsystem and all user ssh and web connections routing to the existing cluster environment
once a user's data is migrated to the new storage system that user's ssh and web connections must route to the new compute environment.
this is accomplished by updating the migration state for the user
when the ssh and web application proxies receive a connection request for the migrated user they look up the user's migration status based on their identity.
accordingly, the migrated user is routed to the login node and ondmand node belonging to the new cluster environment.

these CICD built and deployed application proxies have been successfully used in production for a number of months.
the CICD framework was crucial to the development of the application proxies and has facilited maintaining ondemand services for the distinct cluster environments
this automated deployment workflow enbles deploying new features and bug fixes for actively developed services into our production environment with confidence that the systems remain consistent across builds through a curated infrastructure as code framework.

# Conclusion

infrastructure as code demands programatic control over infrastructure.
not only do we need to script the constructuion of specfic nodes so they contain the appropriate components and configurations to build an application
we also need to control the instanciation of node and ensure they are properly integrated their to the production network
such control is common during development phases.
many production envrionments are manually configuration, they creates a disconnect between the desired automation of CICD pipelines and the reality of the statically configured last mile of connectivity.

moving services to a cloud native model is necessary to ensure the steps that are built and tested during development are valid for the production environment.
by relocating user-aware connection routing to our application proxies for ssh and web applications we were able to focus the CICD pipeline on nodes that can be readily hosted on a cloud platform that supports the VM abstraction.
we provide an on-site cloud computing environment with on OpenStack.
we used this environment as our cloud-native target for the CICD pipelines.
by exposing or cluster-specific network as provider network to our production environment, we are able to control the development and production using identical infrastructure code for both development and production deployments.
this ensures we don't introduce untested code just to address bespoke configuration in production.
the infrastructure code remains the same in all environments and only the configuration values change based on dev or prod targets.
this software controlled infrastructure is critical for a building reliable automation pipelines that are responsive to bug fixes and new application features.
