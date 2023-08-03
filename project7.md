# PROJECT 7 : DEVOPS TOOLING WEBSITE SOLUTION

## STEP 1 – PREPARE NFS SERVER
 Provision a new EC2 instance with RHEL Linux 8 operating system and attach three additional volumes to the server

 ![three additional volumes](./images/2.png)

 1. Partition the three additional disks using gdisk 

 `sudo gdisk /dev/xvdf`

 `sudo gdisk /dev/xvdg`

 `sudo gdisk /dev/xvdh`

 ![disk partition](./images/1.png)

 ![partition setup](./images/4.png)

 Install the lvm tool using the folowing command

 `sudo yum install lvm2 -y`

 ![install lvm tool](./images/3.png)

2.  Create three physical volumes

` sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

![physical volume setup](./images/5.png)

![physical volume setup](./images/6a.png)

3. Create a volume group for the three logical volumes

`sudo vgcreate webdata-vg /dev/xvdh1  /dev/xvdg1 /dev/xvdf1`

![volume group](./images/7.png)

4. Create three logical volumes

`sudo lvcreate -n lv-apps -L 9G webdata-vg`

`sudo lvcreate -n lv-logs -L 9G webdata-vg`

`sudo lvcreate -n lv-opt -L 9G webdata-vg`

![logical volume](./images/8.png)
 
 5. Format the disks into xfs file system

 `sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![logical volume format](./images/9.png)

6.  Create mount points and mount the logical volumes

 `sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

7. Mount the logical volumes

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`

`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`

`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![logical volume mount](./images/10.png)

8. Install NFS server

`sudo yum -y update`

`sudo yum install nfs-utils -y`

9. Start NFS server

`sudo systemctl start nfs-server.service`

10. Enable NFS server to start automatically on reboot

`sudo systemctl enable nfs-server.service`

11. Confirm that NFS server is up and running

`sudo systemctl status nfs-server.service`

![NFS server status](./images/11.png)

12. Set up permission that will allow our Web servers to read, write and execute files on NFS

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

![Grant file permission](./images/12.png)

13. Restart NFS service

`sudo systemctl restart nfs-server.service`

14. Configure access to NFS for clients within the Web servers subnet using vim editor

`sudo vi /etc/exports`

Copy and paste the following commands

`/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

`/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

`/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

Save and exit vim 

`Esc + :wq!`

Export the mounts 

`sudo exportfs -arv`

![Export](./images/13.png)

 In order for NFS server to be accessible from your client, open following ports: TCP 111, UDP 111, UDP 2049 , TCP 2049
