A common point of confusion for DevOps engineers is deciding which commands to use when an EC2 instance runs out of space. Does one use `resize2fs` or `mkfs` ? The answer depends on your goal.

> Key takeaway: Scenario 1 is about Resizing (extending what you have). Scenario 2 is about Formatting (preparing something new).

### Recommendation

- If your **Docker images** are failing because the **root (/)** is full: Use **Scenario 1**.
- If you want to move your **Docker data** to a **separate dedicated disk**: Use **Scenario 2** to prepare the disk, then configure Docker to use that new mount point as its data-root.

## Scenario 1: Volume Expansion (Resizing Root)

Choose this when your primary drive (where the OS and Docker live) is full. You increase the size in the AWS Console, but the Linux OS needs to be told to use that extra space.

### **The Workflow:**

**Step 1 (AWS):** Change Volume Size from 8GB to 30GB.  
**Step 2 (Linux):** Expand the Partition.

```shell
sudo growpart /dev/xvda 1
```

**Step 3 (Linux):** Expand the File System.

```shell
sudo resize2fs /dev/xvda1
```

\*Note: This preserves your existing data. No formatting required.

## Scenario 2: Adding a New Volume (Raw Block Storage)

In this scenario, your instance already has its primary storage (e.g., `/dev/xvda`). You are adding a **new, independent** block of storage to handle specific data(logs, backups, or specific application data)

### The Workflow:

#### Step 1: AWS Console Actions

1.  **Create Volume:** Go to **EC2 > EBS > Volumes** and click "Create Volume." Ensure it is in the same **Availability Zone** as your EC2 instance.
2.  **Attach Volume:** Select the new volume, click **Actions > Attach Volume**, and select your running instance. Take note of the device name suggested (e.g., `/dev/sdf`).

#### Step 2: Linux Terminal Actions

- **Verify the new device:** Use `lsblk` to see the new disk. It will likely appear as `xvdf` or `nvme1n1` with no partitions.

  > _Look for the disk with no "MOUNTPOINT" and the size you just created (e.g.,_ `nvme1n1` _or_ `xvdf`_)._

- **Check if it has data:** (Safety first! Never format a disk that already has a file system).

  Bash

  ```plaintext
  sudo file -s /dev/xvdf
  ```

  _If it says "data", it's empty. If it says "ext4" or "XFS", skip formatting!_

- **Format the raw storage/blank disk:** Create the filesystem.

  ```shell
  sudo mkfs -t ext4 /dev/xvdf
  ```

  > **Warning:** Only do this for the _new_ device. Doing this to your existing partition will result in data loss.

- **Mounting:** Create a directory and link the hardware to it.

  ```shell
  sudo mkdir /mnt/new_data
  sudo mount /dev/xvdf /mnt/new_data
  ```

> CRITICAL WARNING: If you run `mkfs` on an existing disk (Scenario 1), you will ERASE all your data. Use it only for brand-new, empty volumes.

**Step 3: Make it Permanent (The Missing Step)**

In this **Scenario 2,** if you reboot the server, the disk will **disappear**. To make it stay, you must add it to `/etc/fstab`:

1.  Get the UUID: `sudo blkid`
2.  Add a line to `/etc/fstab`: `UUID=your-uuid-here /data ext4 defaults,nofail 0 2`

---

## Disk Management Comparison

<table style="min-width: 75px;"><colgroup><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"></colgroup><tbody><tr><td colspan="1" rowspan="1"><p><strong>Feature</strong></p></td><td colspan="1" rowspan="1"><p><strong>Scenario 1: Resizing</strong></p></td><td colspan="1" rowspan="1"><p><strong>Scenario 2: New Disk</strong></p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Hardware</strong></p></td><td colspan="1" rowspan="1"><p>Existing Disk (Expanding a partition)</p></td><td colspan="1" rowspan="1"><p>Newly Attached Disk (Physical or Virtual)</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Data Status</strong></p></td><td colspan="1" rowspan="1"><p><strong>Preserves existing files</strong></p></td><td colspan="1" rowspan="1"><p><strong>Wipes/Initializes Disk</strong> (Starts empty)</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Common Commands</strong></p></td><td colspan="1" rowspan="1"><p><code>growpart</code>, <code>resize2fs</code>, <code>xfs_growfs</code></p></td><td colspan="1" rowspan="1"><p><code>fdisk</code>, <code>mkfs</code>, <code>mount</code>, <code>blkid</code></p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Best For</strong></p></td><td colspan="1" rowspan="1"><p>Fixing "No Space Left on Device" on root</p></td><td colspan="1" rowspan="1"><p>Adding dedicated storage for apps/logs</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Complexity</strong></p></td><td colspan="1" rowspan="1"><p>Medium (Risk to existing data if interrupted)</p></td><td colspan="1" rowspan="1"><p>Low (Clean slate, no risk to existing data)</p></td></tr></tbody></table>

### Conclusion:

Always run `lsblk` first. If the disk size is larger than the partition size, you need Scenario 1. If you see a disk with no partitions or mount points, you need Scenario 2.
