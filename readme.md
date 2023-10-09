# build-edk2-gvtd
Build EDK2 with Intel GVT-D Support

## Prerequisites
- git
- docker
- internet access
- your own intel gop driver (which can be extracted with [UEFI-BIOS-Updater](https://winraid.level1techs.com/t/tool-guide-news-uefi-bios-updater-ubu/30357) from your Motherboard's BIOS image, which can be downloaded from [here](https://mega.nz/folder/lLg2GLrA#SnZZd0WjHkULFHg7FESm8g) and [here](https://1drv.ms/u/s!AjMjdZ6bIhRQhIpKHjgH7RBQcwtb2A?e=HyM9oP))

## Usage
```shell
git clone git@github.com:cmd2001/Onekey-GVTd.git
cd Onekey-GVTd
sh ./init_edk2.sh
cp <intel gop driver efi> gop/IntelGopDriver.efi
sudo sh ./build_ovmf.sh
sudo sh ./build_oprom.sh
```

Now you have `OVMF_CODE_4MB.fd`, `OVMF_CODE_4MB.secboot.fd` and `XXX.rom` in `./product`. (where `XXX.rom` is 'B660.rom' as default)

The first two files contains a customized edk2 firmware supporting iGPU passthrough, you can copy them to `/usr/share/pve-edk2-firmware` and overwrite the original files from PVE.

If you are unpleasant about overwritting the edk2 firmware of PVE, you can copy `XXX.rom` to `/usr/share/kvm/` and use this OpRom to achieve iGPU passthrough.

Then edit the configuration file of your VM, now you can boot your VM with integrated graphics passthrough and light your monitor.


## Key Changes in VM Config

### Using customized edk2 firmware
```ini
args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=6 -set device.hostpci0.x-igd-opregion=on
hostpci0: 0000:00:02,legacy-igd=1,romfile=vbios_gvt_uefi.rom
```

vbios_gvt_uefi.rom could be downloaded from [Intel_GVT-g - ArchWiki](https://wiki.archlinux.org/title/Intel_GVT-g) and should be put into `/usr/share/kvm/` on PVE host.

### Using OpRom
```ini
args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=6 -set device.hostpci0.x-igd-opregion=on
hostpci0: 0000:00:02,legacy-igd=1,romfile=XXX.rom
```

Where `XXX.rom` denotes the OpRom generated by `build_oprom.sh` ('B660.rom' as default).

**Remember: OS Type must be set to Linux and CPU Type must be set to HOST, otherwise the Windows guest system will not work after installing the driver from Intel.**

## Note
- If you have global internet access(in another word, you are not in China Mainland), comment out line 6-7 in Dockerfile.ovmf for faster download speed.

## Reference

- [Kethen/edk2-build-intel-gop](https://github.com/Kethen/edk2-build-intel-gop)
- [tianocore/edk2](https://github.com/tianocore/edk2)
- [Kethen/edk2](https://github.com/Kethen/edk2)
- [Enable GPU Passthrough (GVT-d)](https://projectacrn.github.io/latest/tutorials/gpu-passthru.html)
- [Set up Intel IGD opregion to allow for passthrough of Intel integrated graphics devices](https://bugzilla.tianocore.org/show_bug.cgi?id=935)
- [Intel_GVT-g - ArchWiki](https://wiki.archlinux.org/title/Intel_GVT-g)
- [my33love/gk41-pve-ovmf](https://github.com/my33love/gk41-pve-ovmf)
- [intel 11-14代N5105 N100 N305等pve核显直通 测试源码](https://www.bilibili.com/read/cv25876083)
- [KVM Hypervisor](https://eci.intel.com/docs/3.1/components/kvm-hypervisor.html?highlight=igd)