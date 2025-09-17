Metasploitable on Proxmox Guide
Overview

This guide explains how to run Metasploitable (a vulnerable Linux VM used for penetration testing labs) on Proxmox using the .vmdk virtual disk image.

Metasploitable is prebuilt and comes as a VMware VMDK file. Proxmox requires importing this disk and configuring a VM to boot from it.

Requirements

Proxmox VE installed and accessible.

Access to Proxmox root account (via Web GUI or SSH).

Download Metasploitable 2

Official Rapid7 Download Page:

Visit : https://www.rapid7.com/products/metasploit/metasploitable/


Fill out the form to access the download link.

The downloaded file will be a .zip archive containing the .vmdk file.

SourceForge Mirror:

Go to the Metasploitable project on SourceForge
.

Download the .zip file, which includes the .vmdk disk image.

metasploitable.vmdk file uploaded to your Proxmox server (e.g., /root/metasploitable.vmdk).

Step 1: Upload VMDK to Proxmox

Upload your VMDK file to the Proxmox server. For example:

scp metasploitable.vmdk root@<Proxmox-IP>:/root/

Step 2: Create a New VM in Proxmox

Go to Proxmox Web GUI → Create VM.

Name it (e.g., Metasploitable).

OS Type: Linux → Debian 64-bit (or any Linux, version doesn’t matter).

Skip adding a disk during creation (we’ll import the VMDK).

Finish creation with default CPU/Memory settings (1–2 cores, 512–2048 MB RAM).

Step 3: Import VMDK to Proxmox Storage

Identify your storage name (supports VM images):

pvesm status


Import the VMDK into Proxmox:

qm importdisk <VMID> /root/metasploitable.vmdk <storage-name>


Example:

qm importdisk 108 /root/metasploitable.vmdk local-lvm


After this, Proxmox shows the disk as unused0.

Step 4: Attach the Imported Disk

Attach the imported disk to the VM as IDE (recommended for Metasploitable):

qm set 108 -ide0 local-lvm:vm-108-disk-0

Step 5: Configure BIOS and Boot Order

Metasploitable requires Legacy BIOS (SeaBIOS), not UEFI:

qm set 108 -bios seabios
qm set 108 -boot order=ide0


Remove any EFI disk if it exists:

qm set 108 -delete efidisk0

Step 6: Start the VM
qm start 108


Open the Proxmox console in the Web GUI:

Log in with:

username: msfadmin
password: msfadmin

Step 7: Optional — SSH Access

Metasploitable has SSH enabled by default:

ssh msfadmin@<VM-IP>


You can find the VM’s IP from the console:

ifconfig


or

ip addr

Step 8: Notes

Do not use Metasploitable on a public network — it is intentionally vulnerable. Use an isolated lab network.

No graphical interface is needed; use the console or SSH.

You can attach additional disks or network interfaces as needed for testing.

Example CLI Full Sequence (Optional)
# Stop VM
qm stop 108

# Attach imported disk as IDE0
qm set 108 -ide0 local-lvm:vm-108-disk-0

# Switch BIOS to SeaBIOS
qm set 108 -bios seabios

# Remove EFI disk if present
qm set 108 -delete efidisk0

# Set boot order
qm set 108 -boot order=ide0

# Start VM
qm start 108
