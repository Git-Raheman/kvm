**convert and run VMs from Hyper-V, VirtualBox, and QEMU on KVM**:

---
## 1. 🎯 Identify the VM Image Format

### 🔹 Hyper-V

* Exported VM is usually `.vhdx` or `.vhd`.

### 🔹 VirtualBox

* Disk format is usually `.vdi`.

### 🔹 QEMU

* Might be `.qcow2` or `.img`.

---

## 2. 🔄 Convert Disk Format to `qcow2` or `raw` (KVM-compatible)

Use `qemu-img` to convert any format.

### ✅ General Conversion Command

```bash
qemu-img convert -f <input_format> -O qcow2 <input_file> <output_file>
```

---

### 🔹 Convert VHDX to QCOW2 (from Hyper-V)

```bash
qemu-img convert -f vhdx -O qcow2 win10.vhdx win10.qcow2
```

---

### 🔹 Convert VDI to QCOW2 (from VirtualBox)

```bash
qemu-img convert -f vdi -O qcow2 ubuntu.vdi ubuntu.qcow2
```

---

### 🔹 Convert IMG to QCOW2 (QEMU raw)

```bash
qemu-img convert -f raw -O qcow2 centos.img centos.qcow2
```

---

## 3. 🖥️ Create New VM in KVM using the Converted Disk

You can use **virt-manager GUI** or CLI with `virt-install`.

---

### ✅ CLI (Recommended for server-only setup)

```bash
virt-install \
  --name myvm \
  --ram 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/ubuntu.qcow2,format=qcow2 \
  --os-type linux \
  --os-variant ubuntu20.04 \
  --network network=default \
  --graphics none \
  --import
```

Change parameters:

* `--os-variant` use `osinfo-query os` to get correct values.
* `--network` can be `bridge=br0` if using a bridged setup.

---

## 4. ✅ Move Converted Image to KVM Images Directory

```bash
sudo mv win10.qcow2 /var/lib/libvirt/images/
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/win10.qcow2
```

---

## 5. 🧪 Test and Boot VM

You can now boot the VM:

```bash
virsh start myvm
virsh console myvm
```

If it's a desktop VM, use `virt-manager` GUI or `virt-viewer`.

---

## 6. 🌐 Optional: Set up Bridged Networking

To let the VM access LAN directly (like Hyper-V switch):

1. Create a Linux bridge:

   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

2. Example config:

   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eno1:
         dhcp4: no
     bridges:
       br0:
         interfaces: [eno1]
         dhcp4: yes
   ```

3. Apply:

   ```bash
   sudo netplan apply
   ```

4. Then in `virt-install` use:

   ```
   --network bridge=br0
   ```

---

## 🔁 Batch Convert Many VMs

If you have many VMs:

```bash
for i in *.vhdx; do
  name=$(basename "$i" .vhdx)
  qemu-img convert -f vhdx -O qcow2 "$i" "${name}.qcow2"
done
```

---

## ✅ Summary Table

| Source Hypervisor | Disk Format | Convert With              | Command Example             |
| ----------------- | ----------- | ------------------------- | --------------------------- |
| Hyper-V           | `.vhdx`     | `qemu-img`                | `-f vhdx -O qcow2`          |
| VirtualBox        | `.vdi`      | `qemu-img`                | `-f vdi -O qcow2`           |
| QEMU              | `.img`      | Optional (usually native) | `-f raw -O qcow2` if needed |

---


