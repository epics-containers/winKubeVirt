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

