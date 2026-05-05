# **Disk Management**

**_Disk management_** involves administrative tasks like adding, removing, and formatting storage volumes. This is critical when applications exhaust existing storage with logs or when large software packages require more overhead. one needs to add more storage to the existing instance(AWS EC2). In AWS, this typically involves **_Elastic Block Store (EBS)_**.

**Disk Monitoring\[Checking Existing Storage and Partitions\]**

Before adding storage, administrators must identify where space is being consumed.

- `df -h`: Displays disk utilization across all **mounted** file systems in human-readable format (GB/MB).
- `du -sh [folder]`: Summarizes the disk usage of a specific directory, helping locate specific "space-hogging" log files. You can also find the size of a directory:

## Storage Integration Workflow **_WITHOUT_** new disk Partition

Best for simple storage expansion where the entire volume is used for one purpose.

**1) Creation and Attachment of volume to existing EC2 instance:**

- **Provision:** Create an EBS volume in the **same Availability Zone** as your EC2 instance.
- **Attach:** Use the AWS Console to attach the volume(block storage) to your instance.
- **Verify:** Run `lsblk` to see the new block device.

  > _Note: It may appear as_ `/dev/xvdf` _on older instances or_ `/dev/nvme1n1` _on newer ones._

**2) Formatting:** Raw block storage cannot be used directly by applications. It must be formatted with a file system (like **ext4** or **xfs**) before use.

- **Check for existing data:** `sudo file -s /dev/xvdf`
  - If it returns "data", it is empty. If it returns a file system type, **do not format it** or you will lose data.

- **Format:** `sudo mkfs -t ext4 /dev/xvdf`

**3) Mounting:** A formatted volume must be "mapped" to a directory in your Linux tree.

- **Create Mount Point:** `sudo mkdir -p /mnt/data_volume`
- **Mount:** `sudo mount /dev/xvdf /mnt/data_volume`
- **Verify:** Run `df -h` to confirm the new space is listed and accessible.

#### 4) Ensuring Persistence (The "Reboot" Step)

By default, manual mounts disappear after a reboot. To make them permanent:

1.  Find the UUID of your drive: `sudo blkid`
2.  Add an entry to the `/etc/fstab` file: `UUID=your-uuid-here /mnt/data_volume ext4 defaults,nofail 0 2`

## Storage Integration Workflow **_WITH_** new disk Partition

Best practice for larger volumes or when dividing a disk for different uses (e.g., Logs vs. Apps).

1\. Creation and Attachment

- **Provision:** Create an EBS volume in the **same Availability Zone** as your EC2 instance.
- **Attach:** Use the AWS Console to attach the volume.
- **Verify:** Run `lsblk` to see the new block device (e.g., `/dev/xvdf` or `/dev/nvme1n1`).

#### 2\. Partitioning (Optional)

If you want to divide your disk into smaller logical units or prepare it for specific boot requirements, use `fdisk`.

1.  Run `sudo fdisk /dev/xvdf`.
2.  Type `n` for a new partition.
3.  Select `p` for primary.
4.  Press **Enter** to accept default sector values (to use the whole disk).
5.  Type `w` to write the changes and exit. _Your device will now appear in_ `lsblk` _as_ `/dev/xvdf1` _(the partition)._

#### 3\. Formatting

Raw devices or partitions must be formatted with a file system before use.

- **Check for existing data:** `sudo file -s /dev/xvdf1`
- **Format:** `sudo mkfs -t ext4 /dev/xvdf1` (Use `ext4` or `xfs` for Linux).

#### 4\. Mounting

- **Create Mount Point:** `sudo mkdir -p /mnt/data_volume`
- **Mount:** `sudo mount /dev/xvdf1 /mnt/data_volume`
- **Verify:** Run `df -h` to confirm the new space is listed.

**Key Disk Management Commands**

<table style="min-width: 222px;"><colgroup><col style="width: 197px;"><col style="min-width: 25px;"></colgroup><tbody><tr><td colspan="1" rowspan="1" colwidth="197"><p><strong>Command</strong></p></td><td colspan="1" rowspan="1"><p><strong>Purpose</strong></p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>lsblk</code></p></td><td colspan="1" rowspan="1"><p>Lists all block devices (disks and partitions) attached to the instance. (you can ignore virtual "loop" disks, which are used for snapshots)</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>sudo fdisk -l</code></p></td><td colspan="1" rowspan="1"><p>While this returns output similar to <code>lsblk</code>but it provides detailed information about partitions and file system levels.</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>sudo fdisk [device]</code></p></td><td colspan="1" rowspan="1"><p>Opens the partition table editor for a specific disk.</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>mkfs -t [type]</code></p></td><td colspan="1" rowspan="1"><p>Formats a partition with a specific file system (e.g., ext4, xfs).</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>mount [dev] [dir]</code></p></td><td colspan="1" rowspan="1"><p>Attaches a formatted disk to a specific folder.</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>umount [dir]</code></p></td><td colspan="1" rowspan="1"><p>Safely detaches a file system.</p></td></tr><tr><td colspan="1" rowspan="1" colwidth="197"><p><code>blkid</code></p></td><td colspan="1" rowspan="1"><p>Displays the unique UUID for block devices (essential for <code>/etc/fstab</code>).</p></td></tr></tbody></table>

### Practical Application

If your root partition (`/`) is at 95% capacity due to logs, you can provision a new 20GB EBS volume, format it as `ext4`, and mount it to `/var/log`. This offloads the heaviest data-generator to a separate disk, ensuring the OS remains stable even if the log disk fills up.

## **Quick Summary**

### **Check Available Disks**

Before creating or mounting anything, always check what block devices exist:

```plaintext
lsblk
```

### **Example output:**

| **NAME** | **MAJ:MIN** | **RM** | **SIZE** | **RO** | **TYPE** | **MOUNTPOINT** |
| -------- | ----------- | ------ | -------- | ------ | -------- | -------------- |
| sda      | 8:0         | 0      | 100G     | 0      | disk     |                |
| ├─sda1   | 8:1         | 0      | 96G      | 0      | part     | /              |
| └─sda2   | 8:2         | 0      | 4G       | 0      | part     | \[SWAP\]       |
| sdb      | 8:16        | 0      | 20G      | 0      | disk     |                |

`sda` → existing disk (already partitioned)

`sdb` → new disk, no partitions yet

### **When to use** `fdisk`

Use `fdisk` when:

- The disk is brand new and has no partitions and you want to create `/dev/sdb1`, `/dev/sdb2`, etc.
- View partition details:

  ```plaintext
  fdisk -l
  ```

- **Creating a Partition with** `fdisk`

  ```plaintext
  fdisk /dev/sdX
  ```

  Follow the interactive prompts to create a partition.

- **Formatting a Partition**

  Format as ext4:

  ```plaintext
  mkfs.ext4 /dev/sdX1
  ```

  Format as XFS:

  ```plaintext
  mkfs.xfs /dev/sdX1
  ```

### **When to Use** `mount`

Use `mount` when: The partition already exists and is formatted You just want to make it accessible

```plaintext
sudo mkdir /mnt/mydisk
sudo mount /dev/sdb1 /mnt/mydisk
```

Now your disk is available at `/mnt/mydisk`.

### **When to Use fdisk + mount (Full Setup)**

Use `fdisk + mkfs + mount` when: The disk is completely new You need to partition → format → mount it

```shell
# 1. Check available disks
lsblk
# 2. Create partition
sudo fdisk /dev/sdb
# 3. Format the partition
sudo mkfs.ext4 /dev/sdb1
# 4. Mount it
sudo mkdir /data
sudo mount /dev/sdb1 /data
```

## **Mounting and Unmounting**

### **Mount a Partition**

```plaintext
mount /dev/sdX1 /mnt
```

### **Unmount a Partition**

```plaintext
umount /mnt
```

**NOTE:**

1.  If you mount a new disk to an existing directory like `/var/log` that already contains data, the old data will be "hidden" by the new mount. Always move the existing logs to the new disk _before_ the final mount to ensure no data is lost and space is actually cleared from the root partition.
2.  **XFS vs Ext4:** You mentioned both. In the AWS world (specifically Amazon Linux 2 and AL2023), **XFS** is the default. For XFS, the command to grow a partition later is `xfs_growfs`, whereas for Ext4 it is `resize2fs`.

## Index of Commands Covered

### Viewing Disk Information

- `lsblk` – Display block devices
- `fdisk -l` – List disk partitions
- `blkid` – Show UUIDs of devices
- `df -h` – Check disk space usage
- `du -sh /path` – Show size of a directory

### Partition Management

- `fdisk /dev/sdX` – Create and manage partitions
- `parted /dev/sdX` – Alternative to `fdisk` for GPT disks
- `mkfs.ext4 /dev/sdX1` – Format a partition as ext4
- `mkfs.xfs /dev/sdX1` – Format a partition as XFS

### Mounting and Unmounting

- `mount /dev/sdX1 /mnt` – Mount a partition
- `umount /mnt` – Unmount a partition
- `mount -o remount,rw /mnt` – Remount a partition as read-write

### Logical Volume Management (LVM)

- `pvcreate /dev/sdX` – Create a physical volume
- `vgcreate vg_name /dev/sdX` – Create a volume group
- `lvcreate -L 10G -n lv_name vg_name` – Create a logical volume
- `mkfs.ext4 /dev/vg_name/lv_name` – Format an LVM partition
- `mount /dev/vg_name/lv_name /mnt` – Mount an LVM partition

### Swap Management

- `mkswap /dev/sdX` – Create a swap partition
- `swapon /dev/sdX` – Enable swap space
- `swapoff /dev/sdX` – Disable swap space
