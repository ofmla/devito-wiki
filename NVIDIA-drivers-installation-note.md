Every time a new kernel comes out you will probably have to manually rebuild the NVIDIA binary driver kernel module. This can be done by booting to the new kernel and then running:
```bash
sudo sh NVIDIA* -K
```
on the previously downloaded NVIDIA installer file.

To avoid this, for newer drivers it is possible to register its module to [DKMS](https://wiki.archlinux.org/index.php/Dynamic_Kernel_Module_Support). To register the driver, first install [DKMS](https://wiki.archlinux.org/index.php/Dynamic_Kernel_Module_Support):
```bash
sudo apt install dkms
```
and then install the driver with the `--dkms` flag:
```bash
sudo sh ./<DRIVER>.run --dkms
```