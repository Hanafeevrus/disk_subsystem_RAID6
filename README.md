#### Raid6_disk_subsistem
details about the Raid installation can be read [here](https://raid.wiki.kernel.org/index.php/RAID_setup)		
This stand shows the installation of Raid6 with check.		
#### Raid6 assembly description in vagrantfile
   
add disks to Vagrantfile		
```
:sata6 => {
                          :dfile => './sata6.vdi',
                          :size => 250,
                          :port => 6
                  },		
```
install mdadm and smartmontools hdparm gdisk		

`yum install -y mdadm smartmontools hdparm gdisk`		

overwrite zero disk blocks		

`mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}`		

create raid 6, l - type raid		

`mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}`		

create raid config		

`sudo echo "DEVICE partitions" >> /etc/mdadm.conf`		

`sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf`		
    
create markup on Raid		
`sudo parted -s /dev/md0 mklabel gpt`	
    
create partitions:		

```		
sudo parted /dev/md0 mkpart primary ext4 0% 16%		
sudo parted /dev/md0 mkpart primary ext4 16% 33%		
sudo parted /dev/md0 mkpart primary ext4 33% 49%
sudo parted /dev/md0 mkpart primary ext4 49% 65%
sudo parted /dev/md0 mkpart primary ext4 65% 81%
sudo parted /dev/md0 mkpart primary ext4 81% 100%		
```
we create directories for the subsequent installation of partitions		
```
sudo  mkdir -p /raid		
sudo mkdir /raid/part{1,2,3,4,5,6}
```
create a file system		
```
sudo mkfs.ext4 /dev/md0p1
		sudo mkfs.ext4 /dev/md0p2
		sudo mkfs.ext4 /dev/md0p3
		sudo mkfs.ext4 /dev/md0p4
		sudo mkfs.ext4 /dev/md0p5
		sudo mkfs.ext4 /dev/md0p6
```
add auto-mount to fstab			

```
    sudo echo "/dev/md0p1 /raid/part1    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p2 /raid/part2    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p3 /raid/part3    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p4 /raid/part4    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p5 /raid/part5    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p6 /raid/part6    defaults 0 0" >> /etc/fstab
```
raid6 builds on boot
   
#### 2 RAID check, simulation of disk failure		
   
fail block device /dev/sde		

```
mdadm /dev/md0 --fail /dev/sde
>mdadm: set /dev/sde faulty in /dev/md0
```
Check the status of Raid, [UUU_UU] space indicates drive fail		
```
   cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid6 sdg[5] sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
1015808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [6/5] [UUU_UU]
 ```  
Check detail:		
   ```
   mdadm -D /dev/md0
Number   Major   Minor   RaidDevice State
0       8       16        0      active sync   /dev/sdb
1       8       32        1      active sync   /dev/sdc
2       8       48        2      active sync   /dev/sdd
-       0        0        3      removed
4       8       80        4      active sync   /dev/sdf
5       8       96        5      active sync   /dev/sdg
3       8       64        -      faulty   /dev/sde
```
   
Delete the broken disk		
`mdadm /dev/md0 --remove /dev/sde`
`mdadm: hot removed /dev/sde from /dev/md0`		

add the replaced disk to the RAID		
`mdadm /dev/md0 --add /dev/sde`		
`mdadm: added /dev/sde`		
  
Check RAID status, no spaces [UUUUUU] - disks in working condition		
`cat /proc/mdstat`		
```
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid6 sde[6] sdg[5] sdf[4] sdd[2] sdc[1] sdb[0]
1015808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [6/6] [UUUUUU]
```
