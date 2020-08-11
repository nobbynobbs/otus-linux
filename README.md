# Инструкции

* [Как начать Git](git_quick_start.md)
* [Как начать Vagrant](vagrant_quick_start.md)

## otus-linux

Используйте этот [Vagrantfile](Vagrantfile) - для тестового стенда.

## log

1. Исходное состояние
    
    4 диска в рейде, один запасной
    ```
    Personalities : [raid10] 
    md0 : active raid10 sdf[4](S) sde[3] sdd[2] sdc[1] sdb[0]
        507904 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]      
    unused devices: <none>
    ```

1. ломаем один диск

    ```
    mdadm /dev/md0 --fail /dev/sde
    ```

    Получаем:

    ```
    Personalities : [raid10] 
    md0 : active raid10 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/3] [UUU_]
      [================>....]  recovery = 81.2% (207232/253952) finish=0.0min speed=207232K/sec
    unused devices: <none>
    
    Personalities : [raid10] 
    md0 : active raid10 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
        507904 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
        
    unused devices: <none>
    ```

1. Ломаем второй диск
    ```
    mdadm /dev/md0 --fail /dev/sdf
    ```
    
    Получаем:
    
    ```
    Personalities : [raid10] 
    md0 : active raid10 sdf[4](F) sde[3](F) sdd[2] sdc[1] sdb[0]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/3] [UUU_]
      
    unused devices: <none>
    ```

1. ломаем третий диск

    ```
    mdadm /dev/md0 --fail /dev/sdb
    ```

    ```
    Personalities : [raid10] 
    md0 : active raid10 sdf[4](F) sde[3](F) sdd[2] sdc[1] sdb[0](F)
      507904 blocks super 1.2 512K chunks 2 near-copies [4/2] [_UU_]
      
    unused devices: <none>
    ```

1. сломать больше трех я не смог, --force пробовал
    ```
    mdadm /dev/md0 --fail /dev/sdc
    mdadm: set device faulty failed for /dev/sdc:  Device or resource busy
    mdadm /dev/md0 --fail /dev/sdd
    mdadm: set device faulty failed for /dev/sdd:  Device or resource busy
    ```

1. "Перетыкаем" один диск
    ```
    mdadm /dev/md0 --remove /dev/sdb 
    mdadm: hot removed /dev/sdb from /dev/md0
    mdadm /dev/md0 --add /dev/sdb 
    mdadm: added /dev/sdb
    ```
    
    Получем:
    ```
    Personalities : [raid10] 
    md0 : active raid10 sdb[5] sdf[4](F) sde[3](F) sdd[2] sdc[1]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/2] [_UU_]
      [==================>..]  recovery = 93.7% (239040/253952) finish=0.0min speed=239040K/sec
      
    unused devices: <none>

    Personalities : [raid10] 
    md0 : active raid10 sdb[5] sdf[4](F) sde[3](F) sdd[2] sdc[1]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/3] [UUU_]
      
    unused devices: <none>
    ```

1. опять ломаем диск

    ```
    mdadm /dev/md0 --fail --force /dev/sdb
    ```

    ```
    Personalities : [raid10] 
    md0 : active raid10 sdb[5](F) sdf[4](F) sde[3](F) sdd[2] sdc[1]
        507904 blocks super 1.2 512K chunks 2 near-copies [4/2] [_UU_]
        
    unused devices: <none>
    ```

1. опять попробовал сломать больше трех
    ```
    mdadm /dev/md0 --fail --force /dev/sdc
    mdadm: set device faulty failed for /dev/sdc:  Device or resource busy
    mdadm /dev/md0 --fail --force /dev/sdd
    mdadm: set device faulty failed for /dev/sdd:  Device or resource busy
    ```

1. надоело, выдергиваем все сбойные диски

    ```
    mdadm /dev/md0 --remove /dev/sdf /dev/sdb /dev/sde
    ```

    ```
    Personalities : [raid10] 
    md0 : active raid10 sdd[2] sdc[1]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/2] [_UU_]
      
    unused devices: <none>
    ```

2. Втыкаем обратно

    ```
    mdadm /dev/md0 --add /dev/sde /dev/sdb /dev/sdf
    ```

    Получаем:

    ```
    Personalities : [raid10] 
    md0 : active raid10 sdf[6] sdb[5] sde[4](S) sdd[2] sdc[1]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/2] [_UU_]
      [================>....]  recovery = 81.2% (207232/253952) finish=0.0min speed=207232K/sec
      
    unused devices: <none>

    Personalities : [raid10] 
    md0 : active raid10 sdf[6] sdb[5] sde[4](S) sdd[2] sdc[1]
      507904 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]
      
    unused devices: <none>
    ```
    
    ```
    mdadm -D /dev/md0
    ```
    
    ```
    /dev/md0:
            Version : 1.2
        Creation Time : Tue Aug 11 18:36:10 2020
            Raid Level : raid10
            Array Size : 507904 (496.00 MiB 520.09 MB)
        Used Dev Size : 253952 (248.00 MiB 260.05 MB)
        Raid Devices : 4
        Total Devices : 5
        Persistence : Superblock is persistent

        Update Time : Tue Aug 11 18:47:17 2020
                State : clean 
        Active Devices : 4
    Working Devices : 5
        Failed Devices : 0
        Spare Devices : 1

                Layout : near=2
            Chunk Size : 512K

    Consistency Policy : resync

                Name : otuslinux:0  (local to host otuslinux)
                UUID : 65dd0108:d91a3bfb:f061c3b5:ac0bc0cf
                Events : 90

        Number   Major   Minor   RaidDevice State
        6       8       80        0      active sync set-A   /dev/sdf
        1       8       32        1      active sync set-B   /dev/sdc
        2       8       48        2      active sync set-A   /dev/sdd
        5       8       16        3      active sync set-B   /dev/sdb

        4       8       64        -      spare   /dev/sde
    ```
