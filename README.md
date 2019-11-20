## Project Goal

This project is focused on making several improvements to the cloud based application [OpenStudio-server](https://github.com/NREL/OpenStudio-server) that is used by building energy modelers.

## Application background

OpenStudio is a cross-platform (Windows, Mac, and Linux) collection of software tools to support whole building energy modeling using [EnergyPlus](https://github.com/NREL/EnergyPlus) and advanced daylight analysis using [Radiance](https://github.com/NREL/Radiance/). OpenStudio is an open source project to facilitate community development, extension, and private sector adoption. OpenStudio includes graphical interfaces along with a Software Development Kit (SDK)

OpenStudio-server is an energy modeling framework that uses OpenStudio and EnergyPlus to run whole distributed building energy modeling simulations in the cloud. Currently, there are some challenges associated with the setup as discussed below. This project is focused on addresses these by optimizing and incorporating concepts and open-sourced tools learned from CSCI-7000 DataCenter and Cloud Computing course @ CU Boulder. 

## Challenges


1 ) The current framework runs in a containerized platform - Docker Swarm. The cluster setup is manually intensive as many SDN  and IaaC steps are hard-wired into ruby client code to configure, provision, a deploy the cluster setup for Docker Swarm.  Furthermore, the current Swarm arch only allows a fixed number of nodes making it difficult for users to determine the optimal cluster size to balance costs and processing time. 

2 ) The current application back-end HTTP API suffers from TCP traffic bottlenecks from the worker node responses. Each worker node returns a large data set, and since many work nodes can send responses simultaneously, it can saturate the link and create very high latent response times and even induce hung processes.

3 ) Due to the issue discussed on #2, raw, high-resolution model output was never designed to be included as part of the output as this data is several times larger than the coarser model output and too intensive to be integrated for end-user post -processing. Although this would be highly beneficial to provide access to the high-resolution model output, up until now, there has been no way to address latency issues to make this as usable feature. 

## Potential Solutions

1 ) Upgrade the current dockerized deployment using Kubernetes orchestration framework. Since the application currently is using Ruby, use a Ruby Kubernetes client to handle the provisioning, deployment, and tear-down of the cluster. Kubernetes offers auto-scaling features that can be triggered by CPU utilization. This should be implemented at the worker nodes which can then auto-scale dynamically based on the model runs.

  
2 ) Several enhancements could be made to optimize the API. The goal is to evaluate using an overlay by-pass framework (Slim or FreeFlow) to determine the efficiency gains in terms of network throughput and latency. Simulate model runs on the existing Kubernetes framework that implements a container overlay network vs FreeFlow that by-passes the overlay network stack.
  
## Dataset or tools
  
Deploying FreeFlow would require RDMA enabled NICs, so having a cloud environment testbed such as CloudLab would be important to test and measure the performance gains. In addition to having a suitable cloud hardware, several container network overlay by-pass frameworks exist:

[https://github.com/danyangz/Slim](https://github.com/danyangz/Slim)

[https://github.com/microsoft/Freeflow](https://github.com/microsoft/Freeflow)

## Timelime 

Now that the OpenStudio-server deployment configs have been updated to run within a k8s environment, I can begin testing out RDMA and Slim in attempt to improve network I/O. Within the semester ends I should have enough time to get one of the two options (Freeflow and/or Slim) to evaluate performance  improvements it relates to network latency, throughput and resource (cpu) consumption.

## Concerns 

The main concern is finding a cloud provider that offers RDMA enabled NICs to run implement FreeFlow.  Slim requires access to the underlining VM, which I believe can be done using k8s provisioned VMs and then making modifications.  If that doesn't work I might have to use custom VMs and install k8s manually to implement Slim.


## Update 2019-11-19


Created a [check list](https://github.com/CU-CSCI7000-Fall2019/final-project-csci7000-container-network-op/issues/1) for progress

The OpenStudio-server deployment configs have been successfully converted to use Kubernetes. Currently, these are statically defined in .yaml configs but initial work
has been done to use k8s deployment manager - Helm.  The static defined configs have been tested on AWS EKS and run successfully. Now that the groundwork has been laid, the next steps are to experiment with specialized cloud hardware that can support RDMA to use FreeFlow and/or use Slim be modifying the VM host containers that run the k8s cluster.


