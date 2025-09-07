---
title: Software Defined HPC with Open OnDemand
authors:
  - name: John-Paul Robinson
    affiliations:
      - University of Alabama at Birmingham, Research Computing
    orcid: 0009-0007-4063-2599
    email: jpr@uab.edu
  - name: Louis Chen
  - name: Eesaan Atluri
  - name: Krish Moodbidri
abstract: |
  We introduce a software defined infrastructure for high performance 
  computing (SDHPC) that includes
  SSH and web application routers for user-based connection routing to endpoints based on individual user identities. We detail Continuous Integration
  and Continuous Deployment (CICD) pipelines used
  to build and deploy an infrastructure of application proxies that route users to appropriate login and Open OnDemand nodes based on their group membership.
  We demonstrate the utility of this infrastructure to our motivating use case of
  limiting user downtime during maintenance operations.
  We conclude with observations on functionality and highlight future directions.
---


# Introduction

we built an abstraction layer in front of the system so that we could affect change in the environment by isolating those changes from the entire community.
this is chiefly accomplished through an application proxy
we provide https and ssh user access to the cluster.
so our application focus has been https and ssh
for our ssh proxy we use ssh-piper
our https applications are proxied with a simple Apache proxy with integrated SSO.
the SSO lets us embed identity into applications by default

the ssh proxy is responsible for routing ssh connections across an infrastructure of user login nodes to the batch computing system
user SSH remote-shell connections are routed to an available login node based on the users identity and system settings.

likewise the https proxy is responsible for routing web application across an infrastructure of user owned webservices.
the http connections are routed based on the user's identity and systems settings

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

the beowulf heratage of modern HPC systems in the origin architecture for much of this capabiltiy
this led to comprehensive tooling to manage a varied and robust collection of systems.
today's HPC system intefaces have also adapted to changing user demand.
ondemand provides access a comprehensive web application that povides user access their applicaitons and data from their comfort of their web browser.
ondemand has established a de facto baseline for user expectation and has now been deployed broadly across the HPC landscape
ondemand also includes a web-based command-line shell so the user has access to full control over their batch environment dirctly in the browser.
these new abilities have not supplanted use of the traditional remote command line shell accessed via SSH.
the extension into a rich web user experience has only grown the variety of applications being run on the batch computing system

while HPC systems have become flexible hardware deployment frameworks
their default user interface has remained centered on the traditional remote command line shell accessed via SSH.
will sufficient, any enhancements to the user experience remain the responsibility of the site deploying the batch system.

like many sites, we have deployed ondemand to add web applictions to the HPC user experience at our site.
the strength of web applications is their fluid adapatation to an ever changing
bugs are fixed and new features added continously
a deployment approach that doesn't address this need to release updates can quickly fall behind the latest releases and misses out on new capabilities.
doesn't facilitiate frequent releases ages quickly and falls behind the state of the art.

# Related Work

Research Computing System

OpenStack for on-site cloud-native infrastructure

Jetstream2 for at-scale cloud operations



# CICD Solution

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
