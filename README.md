# otus-linux-les-2
## Работа с mdadm
### Зануление суперблоков 
`mdadm --zero-superblock --force /dev/sd{b,c,d,e}`  
### Создание RAID 5 на 4 дисках 
`mdadm --create --verbose /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}`  
### Создание mdadm.conf 
`mkdir /etc/mdadm`  
`echo "DEVICE partitions" > /etc/mdadm/mdadm.conf`  
`mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf`  
 
### Создание GPT раздела 
`parted -s /dev/md0 mklabel gpt`  
 
### Создание 5 партиций 
`parted /dev/md0 mkpart primary ext4 0% 20%`  
`parted /dev/md0 mkpart primary ext4 20% 40%`  
`parted /dev/md0 mkpart primary ext4 40% 60%`  
`parted /dev/md0 mkpart primary ext4 60% 80%`  
`parted /dev/md0 mkpart primary ext4 80% 100%`  

### Создание ФС на партициях 
`for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done`  
 
### Монтирование по каталогам 
`mkdir -p /raid/part{1,2,3,4,5}`  
`for i in $(seq 1 5); do sudo echo "/dev/md0p1 /raid/part1 ext4 rw,relatime,seclabel,stripe=1536,data=ordered 0 0\n" > /etc/fstab; done`  

описанные выше команды добавлены в блок `box.vm.provision "shell", inline: <<-SHELL` в Vagrantfile чтобы рейд собирался при загрузке 
 

## Сломать/починить RAID 
 
### Сломать диск в массиве 
`mdadm /dev/md0 --fail /dev/sde`  
 
### Инфо 
`cat /proc/mdstat`  
`mdadm -D /dev/md0`  
 
### Удалить диск из массива 
`mdadm /dev/md0 --remove /dev/sde`  
 
### Добавить диск в массив 
`mdadm /dev/md0 --add /dev/sde`  
 
### Инфо 
`cat /proc/mdstat`  
`mdadm -D /dev/md0`  