# Manage Azure disks with the Azure CL

## Attach disk at VM creation
```
az group create --name myResourceGroupDisk --location eastus

az vm create \
  --resource-group myResourceGroupDisk \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 128
```

## Attach disk to existing VM
```
az vm disk attach \
    --resource-group myResourceGroupDisk \
    --vm-name myVM \
    --name myDataDisk \
    --size-gb 128 \
    --sku Premium_LRS \
    --new
```

## Prepare data disks
```
ssh azureuser@10.101.10.10

sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%

sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1

sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive

df -h | grep -i "sd"
```
Output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        29G  2.0G   27G   7% /
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        14G   41M   13G   1% /mnt
/dev/sdc1        50G   52M   47G   1% /datadrive
```

```
sudo -i blkid
```
Output:
```
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs"
```
Open the /etc/fstab file in a text editor as follows:
```
sudo nano /etc/fstab
```
Add a line similar to the following to the /etc/fstab file, replacing the UUID value with your own.
```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive  xfs    defaults,nofail   1  2
```

## Create snapshot
```
osdiskid=$(az vm show \
   -g myResourceGroupDisk \
   -n myVM \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)


az snapshot create \
    --resource-group myResourceGroupDisk \
    --source "$osdiskid" \
    --name osDisk-backup
```

## Create disk from snapshot
```
az disk create \
   --resource-group myResourceGroupDisk \
   --name mySnapshotDisk \
   --source osDisk-backup
```

## Restore virtual machine from snapshot
```
az vm delete \
--resource-group myResourceGroupDisk \
--name myVM

az vm create \
    --resource-group myResourceGroupDisk \
    --name myVM \
    --attach-os-disk mySnapshotDisk \
    --os-type linux
```

## Reattach data disk
```
datadisk=$(az disk list \
   -g myResourceGroupDisk \
   --query "[?contains(name,'myVM')].[id]" \
   -o tsv)

az vm disk attach \
   –g myResourceGroupDisk \
   --vm-name myVM \
   --name $datadisk
```

