# ansible-nvidia-pve-lxc-passthrough

Ansible playbook to automate NVIDIA GPU passthrough on Proxmox VE (PVE) nodes for use within LXC containers. This playbook:

- Detects NVIDIA GPUs
- Skips nodes that are already correctly configured
- Removes old NVIDIA/CUDA drivers
- Installs NVIDIA drivers directly from NVIDIA
- Sets up IOMMU and VFIO modules
- Adds device access lines to all LXC container configs

---

## Requirements

- Proxmox VE nodes with NVIDIA GPUs
- Passwordless SSH access from control node to Proxmox nodes
- Ansible 2.10+
- Python installed on target hosts

---

## Inventory Example

Create an `inventory.ini` file:

```ini
[proxmox_nodes]
10.0.0.10
10.0.0.11
10.0.0.12
...
```

---

## Usage

Run the playbook with:

```bash
ansible-playbook -i inventory.ini gpu_passthrough_proxmox.yml -l proxmox_nodes --become --become-method=su
```

If your nodes need a password for `su`, add:

```bash
--ask-become-pass
```

---

## What It Does

### 1. Detect GPU Hardware

- Installs `pciutils`
- Verifies presence of NVIDIA VGA devices via `lspci`
- Skips nodes if no GPU is found

### 2. Check Existing Driver

- Runs `nvidia-smi -L`
- Skips node if the output confirms a working NVIDIA driver

### 3. Configure Proxmox

- Installs required packages
- Updates GRUB to enable IOMMU passthrough
- Blacklists default GPU drivers
- Adds VFIO kernel modules
- Updates initramfs
- Reboots the node

### 4. Driver Management

- Removes existing NVIDIA/CUDA packages
- Downloads and installs official NVIDIA `.run` driver (headless/silent)

### 5. Post Validation

- Runs `nvidia-smi -L` and verifies GPU visibility

### 6. LXC Config Update

- Parses `/dev/nvidia*` for major/minor numbers
- Appends `lxc.cgroup2.devices.allow` lines to all container configs in `/etc/pve/lxc/*.conf`

---

## Notes

- Tested with Proxmox VE 7.x and 8.x
- Designed for LXC containers, not full VM passthrough
- The `.run` driver must support the target GPU (update `nvidia_driver_url` if needed)

---

## License

MIT
