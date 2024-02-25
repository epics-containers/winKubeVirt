# How Windows IOCs will fit into epics-containers

## Overview

A windows build server is required. This should be set up using the method described in [Build Server](BUILDSERVER.md).

The build server will also require the SDKs from OEMs that we build IOCs for. It will not require access to any shared filesystems because we will build everything from source on this server as per linux epics-containers Dockerfiles.

The build server could be one of:

1. a VM running in KubeVirt in the Argus (or other) Cluster. Created and managed by Beamline Controls.
1. a standalone HW or virtual machine managed by SciComp
1. it could also be the target runtime machine (i.e. different for each IOC instance). This would be a simple approach to maintaining compatibility between build and runtime SDKs.

I prefer 1. because:
- the servers are easy to create in around 10 mins
- they need not be in any DLS domain or require access to any DLS resources.
  - this makes them
    - quick and easy to set up
    - more lightweight
    - less of a security risk as they can be isolated as much as required
- they can be spun up and down really quickly as part of the Generic IOC CI
- we could maintain several versions if there were incompatible requirements between different OEMs SDKs

## Generic IOCs

### developer stage container build
- should look as similar as possible to linux generic IOCs, still using ibek-support to download and configure support modules
- will probably only run on our GitLab CI - but GitHub is not out of the question, any solution is likely to include an additional cloud service to host the Windows VM
  - e.g. https://learn.microsoft.com/en-us/azure/developer/github/build-vm-image?tabs=userlevel%2Cprincipal.
  - But what about IRIS?
- will in addition to linux GIOCs require an ssh address for a build server and a private key to authenticate that connection (using a sealed secret so it can be stored in the repo)
- Will execute a container build and farm off the compilation to the build server
  - How? well here is one idea:
  - The full-fat-crazy-cool option:
    - modify `ibek support compile` to be:
      - scp the current folder to the build server
      - ssh make on the build server
      - scp the built assets back into the calling container
    - this means that we get build-cache support in a windows build!!!!
    - It requires a step to decide on the temporary build folder name on the server and record that into the container filesystem.
    - it means that the ibek-support folder for the support modules are unchanged for compilation on windows or linux. Just the `ibek` `compile` command behaves differently for windows targets
    - NOTE: this would be relatively eay to get working at develop and debug time in a developer container

### runtime stage container build
- The runtime phase would be FROM a standard windows GIOC container image and get the built assets copied in from the developer stage just like linux does. This is pretty much the same thing as RTEMS will be doing

### runtime
- When the GIOC runtime is launched it will:
  - connect to the target windows runtime server
  - scp the built assets out of the container filesystem to the target server
  - scp the Instance config into the config folder of the GIOC
  - use ssh to launch the IOC instance and keep it running with its stdio acting as the console to the IOC.

## IOC Instance

The IOC instance folder in the beamline repo will look almost identical to that for a linux IOC instance. It will have:

- A values file to indicate which GIOC to launch in the cluster
  - the values file will also hold details of the target runtime server, so these will get baked into the deployment via the ioc-instance helm-chart
- a config folder with a standard ioc.yaml to tell the GIOC what instance configuration to use


