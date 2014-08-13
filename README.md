ansible-vsphere
===============

Ansible module to automate the vsphere api.

This module uses the official VMWare python library https://github.com/vmware/pyvmomi.
Current supported features:
1. Power on, Guest Shutdown, Power off VMs
2. Create, Delete, Clone VMs
3. Upgrade VMWare Tools
4. Create, Delete, Revert VM Snapshots
5. Using the Guest Operations Manager

The module takes a special spec object that will be converted to the
correct vsphere object during processing of the task. The spec
object has two top level parameters:
type: The type of spec ( such as VirtualMachineCloneSpec )
      In some cases this could be the name of a function
      that will be called.
value: A representation of the values that fill out the spec
       object. In the case of the spec being used to specify
       a function being called, the values should be the
       parameters that get passed to the function.

In order to fill spec objects with other spec objects, the values
will be put through a recursive update, that will convert any key
that is an attribute of the `vim` module of `pyVmomi`. The
update process can even substitute Managed Object References
through the special syntax.
`{ "ManagedObjectReference" : { "type": "MOR TYPE", "name" : "MOR NAME" } }`

For example below is how to clone a VM. The type VirtualMachineCloneSpec
corresponds directly to the definition at
http://pubs.vmware.com/vsphere-55/index.jsp?topic=%2Fcom.vmware.wssdk.apiref.doc%2Fvim.vm.CloneSpec.html


```
- name: Clone VM
  local_action:
    module: vsphere
    host: "{{ vcenter_host }}"
    login: "{{ vcenter_login }}"
    password: "{{ vcenter_password }}"
    timeout: 60
    guest:
      name: "{{ deleteme }}"
      state: present
      folder: "{{ folder }}"
      clone_from: base-tmpl
    spec:
      type: VirtualMachineCloneSpec
      value:
        config:
          VirtualMachineConfigSpec:
            name: "{{ deleteme }}"
            memoryMB: 2048
            numCPUs: 1
            deviceChange: []
        location:
          VirtualMachineRelocateSpec:
            pool:
              ManagedObjectReference:
                type: ResourcePool
                name: Resources
        powerOn: False
        template: False
```
