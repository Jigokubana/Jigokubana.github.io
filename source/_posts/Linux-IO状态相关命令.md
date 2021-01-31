---
title: Linux-IO状态相关命令
date: 2021-01-09 18:53:22
categories: 
- Linux
tags:
- iostat
- df
- du
---

# iostat

```
yum install sysstat -y


[root@fish ~]# iostat
Linux 4.18.0-193.6.3.el8_2.x86_64	01/09/2021 	_x86_64_	(4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.06    0.02    0.08    0.01    0.00   99.84

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.00         1.68         0.00    3520393       3015
sdb               0.16         0.81         2.68    1702968    5612922
dm-0              0.17         0.79         2.78    1656682    5811573
dm-1              0.00         0.00         0.01       4404      10828
dm-2              0.00         0.00         0.00       1193       2209
```

# df

```
[root@fish ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             1.8G     0  1.8G   0% /dev
tmpfs                1.8G   84K  1.8G   1% /dev/shm
tmpfs                1.8G   40M  1.8G   3% /run
tmpfs                1.8G     0  1.8G   0% /sys/fs/cgroup
/dev/mapper/cl-root   19G   13G  6.5G  67% /
/dev/sda             112G   34G   79G  30% /nas1
/dev/mapper/cl-home  4.0G   61M  4.0G   2% /home
/dev/sdb1            2.0G  211M  1.6G  12% /boot
tmpfs                368M     0  368M   0% /run/user/0
```

# du

estimate file space usage

估计文件空间使用量 默认察看当前目录
```
[root@fish ~]# du -h
8.0K	./book
244K	./.halo/logs
72K	./.halo/db
36K	./.halo/templates/themes/anatole/source/plugins/prism/css
152K	./.halo/templates/themes/anatole/source/plugins/prism/js
188K	./.halo/templates/themes/anatole/source/plugins/prism
900K	./.halo/templates/themes/anatole/source/plugins/gallery/fonts
32K	./.halo/templates/themes/anatole/source/plugins/gallery/css/images
104K	./.halo/templates/themes/anatole/source/plugins/gallery/css
56K	./.halo/templates/themes/anatole/source/plugins/gallery/js/ie
180K	./.halo/templates/themes/anatole/source/plugins/gallery/js
1.2M	./.halo/templates/themes/anatole/source/plugins/gallery
1.4M	./.halo/templates/themes/anatole/source/plugins
72K	./.halo/templates/themes/anatole/source/images
1.1M	./.halo/templates/themes/anatole/source/fonts
80K	./.halo/templates/themes/anatole/source/css
356K	./.halo/templates/themes/anatole/source/js
2.9M	./.halo/templates/themes/anatole/source
24K	./.halo/templates/themes/anatole/module
3.1M	./.halo/templates/themes/anatole

```