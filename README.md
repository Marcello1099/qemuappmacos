# App-Like QEMU with 3D Graphic Acceleration on MacOS

This guide is mostly focused on the M1 Macs, but it can be adapted for Intel models easily. This is an implementation of an app-like experience through Automator, whereas QEMU is from user knazarov's [GitHub Repo](https://github.com/knazarov/homebrew-qemu-virgl#readme) please follow the steps for QEMU installation and setup.

This setup allows to move the app to an external drive or be used locally as well, it behaves like your average MacOS app in the user end.

## Installation

### Files needed

From the QEMU [guide](https://github.com/knazarov/homebrew-qemu-virgl#readme) you will have 3 files ".raw" for storage, "edk2-arm-vars.fd", and "edk2-aarch64-code.fd". Move these three files into a new folder called "main".

### Enviroment

Open **Automator** and go to `File > New > Application`

### Getting the Path

Then, in actions select and add `Run AppleScript` and add:

```
  set my_POSIX_path to POSIX path of ((path to me as text) & "::")
```

### Starting the VM

Once again in actions select and add `Run Shell Script`, in **"Shell:"** select `/bin/zsh`, then in **"Pass input:"** select `as arguments`, and add:

**For shared clipboard and mouse integration:**

```
  /opt/homebrew/bin/qemu-system-aarch64 \
         -machine virt,accel=hvf,highmem=off \
         -cpu cortex-a72 -smp 4 -m 4G \
         -device intel-hda -device hda-output \
         -device qemu-xhci \
         -device virtio-gpu-gl-pci \
         -device usb-kbd \
         -device usb-tablet \
         -device virtio-net-pci,netdev=net \
         -chardev qemu-vdagent,id=spice,name=vdagent,clipboard=on \
         -device virtio-serial-pci \
         -device virtserialport,chardev=spice,name=com.redhat.spice.0 \
         -netdev user,id=net,ipv6=off \
         -display cocoa,gl=es \
         -drive "if=pflash,format=raw,file=$1/Fedora.app/Contents/main/edk2-aarch64-code.fd,readonly=on" \
         -drive "if=pflash,format=raw,file=$1/Fedora.app/Contents/main/edk2-arm-vars.fd,discard=on" \
         -drive "if=virtio,format=raw,file=$1/Fedora.app/Contents/main/hdd.raw,discard=on"
```

**For default settings:**

```
  /opt/homebrew/bin/qemu-system-aarch64 \
         -machine virt,accel=hvf,highmem=off \
         -cpu cortex-a72 -smp 4 -m 4G \
         -device intel-hda -device hda-output \
         -device qemu-xhci \
         -device virtio-gpu-gl-pci \
         -device usb-kbd \
         -device virtio-net-pci,netdev=net \
         -device virtio-mouse-pci \
         -display cocoa,gl=es \
         -netdev user,id=net,ipv6=off \
         -drive "if=pflash,format=raw,file=$1/Fedora.app/Contents/main/edk2-aarch64-code.fd,readonly=on" \
         -drive "if=pflash,format=raw,file=$1/Fedora.app/Contents/main/edk2-arm-vars.fd,discard=on" \
         -drive "if=virtio,format=raw,file=$1/Fedora.app/Contents/main/hdd.raw,discard=on"
```

-smp are the CPU cores you desire, I suggest max of 4 in M1

-m is the amount of RAM you desire, I suggest max half of your RAM

Notice it states in the drive options **"Fedora.app"** this needs to be equal to the name given to the application when doing the next step:

`File > Save...` and make sure **"File Format:"** is set to **"Application"** 

### Storing the VM

After saving the application in Automator a new app (Default: Fedora) will be added to your Applications. Go ahead and navigate to your `Applications` folder and find your newly created app. Right click on it and select **"Show Package Contents"**, navigate to `Contents` and move your previously created folder **"main"**.

### Icon

For changing the icon I suggest the [guide from Apple](https://support.apple.com/guide/mac-help/change-icons-for-files-or-folders-on-mac-mchlp2313/mac).

For Fedora I recommend [this icon](https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.svg), but you can use whatever you desire.

