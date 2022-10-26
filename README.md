# DZ3.4

================1====================
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0

● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-10-26 11:51:59 UTC; 4s ago
   Main PID: 3456 (node_exporter)
      Tasks: 5 (limit: 9291)
     Memory: 2.8M
     CGroup: /system.slice/node_exporter.service
             └─3456 /home/vlad/node_exporter/node_exporter

root@ubnt2004:/home/vlad/node_exporter# cat /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter

[Service]
ExecStart=/home/vlad/node_exporter/node_exporter - расположение
EnvironmentFile=/etc/default/node_exporter_conf/node_exporter.conf - предусмотренная возможность добавления опций к запускаемому процессу через внешний файл

[Install]
WantedBy=default.target

================2====================

CPU:
node_cpu_seconds_total{cpu="0",mode="idle"} 2238.49
 node_cpu_seconds_total{cpu="0",mode="system"} 16.72
node_cpu_seconds_total{cpu="0",mode="user"} 6.86
process_cpu_seconds_total
    
Memory:
node_memory_MemAvailable_bytes 
node_memory_MemFree_bytes
    
Disk(если несколько дисков то для каждого):
node_disk_io_time_seconds_total{device="sda"} 
node_disk_read_bytes_total{device="sda"} 
node_disk_read_time_seconds_total{device="sda"} 
node_disk_write_time_seconds_total{device="sda"}

Network(так же для каждого активного адаптера):
node_network_receive_errs_total{device="eth0"} 
node_network_receive_bytes_total{device="eth0"} 
node_network_transmit_bytes_total{device="eth0"}
node_network_transmit_errs_total{device="eth0"}

================3====================

Работу выполнял на своей виртуальной машине в HyperV по этому чуток отличается ссылка.

================4====================

Можно
[    0.040252] Hyper-V: PV spinlocks enabled
[    0.040256] PV qspinlock hash table entries: 512 (order: 1, 8192 bytes, linear)
[    0.040266] Built 1 zonelists, mobility grouping on.  Total pages: 2039545
[    0.040266] Policy zone: Normal

[    0.089966] Hyper-V: Using IPI hypercalls

[    0.761842] ata_piix 0000:00:07.1: Hyper-V Virtual Machine detected, ATA device ignore set

[    5.675925] hv_vmbus: Vmbus version:5.0

[   68.768973] hv_balloon: Max. dynamic memory size: 8096 MB

================5====================

vlad@ubnt2004:~$ /sbin/sysctl -n fs.nr_open
1048576

Это максимальное число открытых дескрипторов для ядра (системы), для пользователя задать больше этого числа нельзя (если не менять). 
Число задается кратное 1024, в данном случае =1024*1024. 
Другой лимит:
vlad@ubnt2004:~$ cat /proc/sys/fs/file-max
9223372036854775807

================6====================

root@ubnt2004:~# ps -e | grep sleep
   8680 pts/3    00:00:00 sleep
root@ubnt2004:~# nsenter --target 8680 --pid --mount
root@ubnt2004:/# ps
    PID TTY          TIME CMD
   8696 pts/3    00:00:00 sudo
   8697 pts/3    00:00:00 bash
   8742 pts/3    00:00:00 nsenter
   8743 pts/3    00:00:00 bash
   8756 pts/3    00:00:00 ps
root@ubnt2004:/#

================7====================

Это функция внутри "{}", судя по всему с именем ":" , которая после определения в строке запускает саму себя, порождает два фоновых процесса самой себя, получается этакое бинарное дерево плодящихся процессов 

А функционал судя по всему этот:
[ 3099.973235] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-4.scope
[ 3103.171819] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-11.scope

Судя по всему, система на основании этих файлов в пользовательской зоне ресурсов имеет определенное ограничение на создаваемые ресурсы 
и соответственно при превышении начинает блокировать создание числа 

Если установить ulimit -u 50 - число процессов будет ограничено 50 для пользователя. 
