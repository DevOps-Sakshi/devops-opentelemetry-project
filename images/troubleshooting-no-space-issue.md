# Issue: “No space left on device” during docker compose up
While running the microservices using:

`sudo docker compose up`
The execution stopped halfway and Docker showed this error:

`failed to prepare extraction snapshot ... no space left on device`

This means the EC2 instance ran out of disk storage, preventing Docker from pulling more images or creating layer snapshots.

## Root Cause
- The default EC2 volume size (8GB) was too small for all the microservices and images used in the project. As the Docker images downloaded, the root volume filled up completely, causing the snapshot creation to fail.

## Solution: Increase EC2 Volume + Resize Filesystem
### Step 1: Increase the EBS Volume Size
In AWS Console:
- EC2 → Elastic Block Store → Volumes → Select Volume → Modify Volume
- Old size: 8GB
- New size: 30GB
- After modifying, AWS will show the volume as “in-use – optimizing”, but it is immediately usable.

## Step 2: Verify Current Disk Usage
- Run:
`df -h`

- This showed that even after increasing the EBS volume, the filesystem did not automatically expand.

## Step 3: Check Block Devices
- To confirm the correct partition:

`lsblk`
- Output showed:
```
nvme0n1     (main disk)
├─ nvme0n1p1  (root partition)
```
- We must extend partition 1 (nvme0n1p1).

## Step 4: Grow the Partition
- Use growpart:

`sudo growpart /dev/nvme0n1 1`
- This increases the partition size, but not yet the filesystem.

## Step 5: Resize the Filesystem
- Run:

`sudo resize2fs /dev/nvme0n1p1`
- This expands the file system so the OS can use the new disk space.

## Step 6: Verify the New Size
`df -h`
Now the root filesystem shows the full 30GB available.

## Final Result
- After resizing the disk and filesystem, running:

`sudo docker compose up`
completed successfully without any space issues.