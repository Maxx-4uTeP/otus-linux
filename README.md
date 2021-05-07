# Домашка 2 "Дисковая подсистема"
Vagrant файл: <https://github.com/Maxx-4uTeP/otus-linux/blob/master/Vagrantfile>

В файл добавлены несколько HDD и уменьшен их размер с 250 до 100Мб:

    :sata6 => {
    :dfile => './sata6.vdi',
    :size => 100, # Megabytes
    :port => 6
    }

Так же в файл добавлен код сборки рейда и монтирования каталогов:

    mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}
    mkdir /etc/mdadm
    touch /etc/mdadm/mdadm.conf
    echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
    mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
    parted -s /dev/md0 mklabel gpt
    parted /dev/md0 mkpart primary ext4 0% 20%
    parted /dev/md0 mkpart primary ext4 20% 40%
    parted /dev/md0 mkpart primary ext4 40% 60%
    parted /dev/md0 mkpart primary ext4 60% 80%
    parted /dev/md0 mkpart primary ext4 80% 100%
    for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
    mkdir -p /raid/part{1,2,3,4,5}
    for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

в конце добавлен вывод смонтированных каталогов с рейда:

    echo "ls /raid/"
    ls /raid/

Итоговый результат:

    otuslinux: part1
    otuslinux: part2
    otuslinux: part3
    otuslinux: part4
    otuslinux: part5


Для монтрования после перезапуска добавлена строка:

    for i in $(seq 1 5); do echo "/dev/md0p$i /raid/part$i ext4 defaults 0 0" >> /etc/fstab; done