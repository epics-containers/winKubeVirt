# Notes on using KubeVirt to make a Windows build server

First follow this most excellent guide:
[git@github.com:gilesknap/winKubeVirt.git](https://medium.com/adessoturkey/create-a-windows-vm-in-kubernetes-using-kubevirt-b5f54fb10ffd)

Then remove the installation media and add a network interface by applying the VM.yaml.

Now we need to get some network drivers.

Follow

https://kubevirt.io/user-guide/virtual_machines/windows_virtio_drivers/#which-drivers-i-need-to-install
https://chrisbrown.au/techblog/install-windows-drivers-from-folder-using-powershell/

NOTE: this part could have been done at the beginning of the install when adding the virtual disk drivers - that would speed things up.

Screen shot:
[](netdriver.png)

Now you can go back to SConfig and download OS updates.

Add a service so that you can connect to the server using Remmina.

Add RDP service:
```bash
kubectl virt expose vmi iso-win10 --port=3389 --name=rdp --type=NodePort
```

For some reason you must initially connect with the kubectls vnc client, otherwise Remmina connects and drops out. But Remmina is much more fun because it supports cut and paste, different screen sizes and does not grab the mouse.

To connect with kubectl:
```bash
kubectl virt vnc iso-win10-2
```

Set up a shared PVC for copying MSIs etc in.
Could not get this to work - probably filesystem issues. But rcp from a container in the cluster works great. ([See how to rcp here](BUILDSERVER.md) )
