# lesson2_disk_subsistem
# 1 сборка RAID6
   
   ##### добавляем диски в Vagrantfile ##
        :sata6 => {
                          :dfile => './sata6.vdi',
                          :size => 250,
                          :port => 6
                  },
   ##### устанавливаем mdadm и smartmontools hdparm gdisk
    yum install -y mdadm smartmontools hdparm gdisk
    
   ##### затираем нулевые блоки дисков
    mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
    
   #### создаем raid 6, l - тип raid
    mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}
    
   #### создаем конфиг raid
    sudo echo "DEVICE partitions" >> /etc/mdadm.conf
	  sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf
    
   #### создаем разметку на Raid'e
    sudo parted -s /dev/md0 mklabel gpt
    
   #### создаем партиции
    sudo parted /dev/md0 mkpart primary ext4 0% 16%
		sudo parted /dev/md0 mkpart primary ext4 16% 33%
		sudo parted /dev/md0 mkpart primary ext4 33% 49%
		sudo parted /dev/md0 mkpart primary ext4 49% 65%
		sudo parted /dev/md0 mkpart primary ext4 65% 81%
		sudo parted /dev/md0 mkpart primary ext4 81% 100%
    
   #### создаем директории для последующего монтирования партиций
    sudo  mkdir -p /raid
		sudo mkdir /raid/part{1,2,3,4,5,6}
    
   #### создаем файловую систему
    sudo mkfs.ext4 /dev/md0p1
		sudo mkfs.ext4 /dev/md0p2
		sudo mkfs.ext4 /dev/md0p3
		sudo mkfs.ext4 /dev/md0p4
		sudo mkfs.ext4 /dev/md0p5
		sudo mkfs.ext4 /dev/md0p6
    
   #### добавляем в fstab автомонтирование
    sudo echo "/dev/md0p1 /raid/part1    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p2 /raid/part2    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p3 /raid/part3    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p4 /raid/part4    defaults 0 0" >> /etc/fstab
    sudo echo "/dev/md0p5 /raid/part5    defaults 0 0" >> /etc/fstab
		sudo echo "/dev/md0p6 /raid/part6    defaults 0 0" >> /etc/fstab
    
   ### raid6 собирается при загрузки
   
   # 2 Проверка RAID, иммитация выхода из строя диска
   
   #### зафейлим блочное устройство /dev/sde
   mdadm /dev/md0 --fail /dev/sde
   >mdadm: set /dev/sde faulty in /dev/md0
   
   #### Проверяем статус Raid, [UUU_UU] пробел указывает на fail диска
   cat /proc/mdstat
   > Personalities : [raid6] [raid5] [raid4] 
   >
   > md0 : active raid6 sdg[5] sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
   >
   > 1015808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [6/5] [UUU_UU]
   
   #### Проверяем детали
   mdadm -D /dev/md0
   > Number   Major   Minor   RaidDevice State
   >
   > 0       8       16        0      active sync   /dev/sdb
   > 
   > 1       8       32        1      active sync   /dev/sdc
   >
   > 2       8       48        2      active sync   /dev/sdd
   >
   > -       0        0        3      removed
   >
   > 4       8       80        4      active sync   /dev/sdf
   > 
   > 5       8       96        5      active sync   /dev/sdg
   > 
   > 3       8       64        -      faulty   /dev/sde

   
   #### Удалим сломанный диск
   mdadm /dev/md0 --remove /dev/sde
   > mdadm: hot removed /dev/sde from /dev/md0
   
   #### добавим замененный диск в RAID
   mdadm /dev/md0 --add /dev/sde
   > mdadm: added /dev/sde
   
   #### Проверяем статус RAID, отсутствие пробелов [UUUUUU] - диски в рабочем состояни
   cat /proc/mdstat
   >Personalities : [raid6] [raid5] [raid4] 
   >
   > md0 : active raid6 sde[6] sdg[5] sdf[4] sdd[2] sdc[1] sdb[0]
   >
   >   1015808 blocks super 1.2 level 6, 512k chunk, algorithm 2 [6/6] [UUUUUU]
   
