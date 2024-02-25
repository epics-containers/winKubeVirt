# Notes on using KubeVirt to make a Windows build server

First follow this most excellent guide:
[git@github.com:gilesknap/winKubeVirt.git](https://medium.com/adessoturkey/create-a-windows-vm-in-kubernetes-using-kubevirt-b5f54fb10ffd)

Then remove the installation media and add a network interface by applying the VM.yaml.

Now we need to get some network drivers.

Follow

https://kubevirt.io/user-guide/virtual_machines/windows_virtio_drivers/#which-drivers-i-need-to-install
https://chrisbrown.au/techblog/install-windows-drivers-from-folder-using-powershell/

Screen shot:
[](netdriver.png)

Now you can go back to SConfig and download OS updates!

This may help setting up RDP.
https://redhat-developer-demos.github.io/kubevirt-crc-windows-tutorial/

Add RDP service:
```bash
kubectl virt expose vmi iso-win10 --port=3389 --name=rdp --type=NodePort
```

Set up a shared PVC for copying MSIs etc in.
Could not get this to work - probably filesystem issues.
