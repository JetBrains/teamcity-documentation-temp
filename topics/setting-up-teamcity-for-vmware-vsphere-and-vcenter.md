[//]: # (title: Setting Up TeamCity for VMware vSphere and vCenter)
[//]: # (auxiliary-id: Setting Up TeamCity for VMware vSphere and vCenter)

TeamCity vSphere integration allows using TeamCity [agent cloud features](teamcity-integration-with-cloud-solutions.md) with VMware __vSphere__ and __vCenter__ installation. It requires configuring TeamCity with your VMware vSphere/vCenter account and then handles automatic creation/starting/stopping/deleting of the virtual machines with TeamCity agents on-demand, based on the queued builds.

The functionality is implemented as a plugin bundled with TeamCity.

## Requirements

<note>

 The integration requires a write access for vSphere/vCenter via API, which is available in the licensed versions; the API support of free versions is limited to read-only  access.
</note>

## Features

The TeamCity vSphere integration allows you to:
* Select the type of behavior:
  * Start/stop an existing virtual machine.
  * Clone a virtual machine or a template, and either delete the clone once it becomes idle (when the build is finished or an idle timeout elapses, depending on the profile settings) or preserve it after stopping.
* Select a virtual machine snapshot to start.
* Specify the directory for clones and the resource pool where your machine will be allocated.
* Configure the maximum number of started instances.

## Usage

The following steps are required to set up TeamCity-VMware vSphere agent cloud integration:

1. Create a virtual machine where your builds will run. Refer to the VMware vSphere website for details on creating [virtual machines](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-F559CE9C-2D8F-4F69-A846-56A1F4FC8529.html).
2. Make sure VMware Tools or Open VM Tools are installed. See [VMware documentation](https://docs.vmware.com/en/VMware-Tools/10.1.0/com.vmware.vsphere.vmwaretools.doc/GUID-28C39A00-743B-4222-B697-6632E94A8E72.html).
3. Install a TeamCity agent, configure and test the machine as described in [this section](teamcity-integration-with-cloud-solutions.md#Preparing+Virtual+Machine).
   * If you want TeamCity to start/stop this machine on demand or to clone it, proceed to configuring the VMware [agent cloud profile](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images) on the TeamCity server.  When the profile is modified, TeamCity detects the changes immediately, and forces shutdown of the agents started prior to these changes once the agents finish the current build.
   * If you want to create a template of this machine and clone it, refer to the VMware vSphere website for details on creating [templates](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.hostclient.doc/GUID-F40130B0-0194-4A41-91FA-1A967721924B.html) and proceed to configuring the VMware [cloud profile](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images) on the TeamCity server. Make sure to specify the valid [vCenter SDK URL](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/vmc-aws-performance/GUID-02CB4E53-2039-4ED7-BAB0-CFE30FB1C6F0.html).

<note>

When configuring the TeamCity build agent, be sure to specify the valid TeamCity server URL in the build agent properties.
</note>

### Notes on Configuring Agent Cloud Profile

* You can limit a number of instances across all images and/or set the limit per image. 
* It is possible to specify unique hostnames for cloud agents: when adding an image, choose a customization spec from the corresponding field. The option is available for Windows and Linux virtual machines.
* When using the same image in different cloud profiles, to avoid possible conflicts, it is possible to use a custom agent image name when configuring a cloud profile in TeamCity. This can be also useful with naming patterns for agents. When a custom agent image name is specified, the names of cloud agent instances cloned from the image will be based on this name.


### Clone Virtual Machine to Reuse Agent

To clone a virtual machine or template and preserve the clone so TeamCity can reuse it, select the respective option in the _Behavior_ configuration field of the __Add Image/Edit Image__ dialog:

<img src="vCenter-add-image-behavior.png" alt="vCenter Add Image Behavior" width="706"/>

Note that if you use `<Current State>` in the _Snapshot name_ field, TeamCity will always create a new clone and delete the previous one. To preserve the cloned virtual machine:
* __For virtual machines__:  
  1. Select the virtual machine that you want to use in the _Agent image_ field.
  2. [Create a snapshot of your virtual machine](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.hostclient.doc/GUID-A0D8E8E7-629B-466D-A50C-38705ACA7613.html) and use its name in the _Snapshot name_ field.
* __For templates:__  
  1. [Convert a template to a virtual machine](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.hostclient.doc/GUID-79C2F2A0-D268-4F4A-88CC-8B87144E5E46.html), create a snapshot of this virtual machine, and then [convert the virtual machine back to a template](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.hostclient.doc/GUID-846238E4-A1E3-4A28-B230-33BDD1D57454.html). 
  2. In TeamCity, select the template in the _Agent image_ field and use the snapshot you have just created in the _Snapshot name_ field .