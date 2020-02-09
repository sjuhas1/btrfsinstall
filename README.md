# btrfsinstall
```
sgdisk --zap-all /dev/nvme0n1
sgdisk --clear \
  --new=1:0:+1G --typecode=1:ef00 --change-name=1:EFI \
  --new=2:0:+1G --typecode=2:8300 --change-name=2:boot \
  --new=3:0:+16G --typecode=3:8200 --change-name=3:swap \
  --new=4:0:+0 --typecode=4:8300 --change-name=4:system \
  /dev/nvme0n1

sgdisk --print /dev/nvme0n1
```
