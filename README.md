# otus-task6

# Размещаем свой RPM в своем репозитории

1. Создать свой RPM пакет (можно взять свое приложение, либо собрать, например, апач с определенными опциями);
2. Создать свой репозиторий и разместить там ранее собранный RPM.

## Создать свой RPM пакет

### Решение

Для примера возьмём пакет NGINX и соберём его с OpenSSL. \

Для выполнения задания будем использовать виртуальную машину с ОС AlmaLinux 9. \
ВМ будем разворачивать с помощью Vagrant. Vagrantfile следющего содержания:
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "almalinux/9"
  config.vm.define "otusrpm"
  config.vm.hostname = "otusrpm"
  
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 1
  end

end
```

С помощью команды vagrant up запускаем нашу виртуалку. \
С помощью команды vagrant ssh подключаемся к нашей виртуальной машине.

После подключения к ВМ устанавливаем необходимые пакеты, которые будут нужны для сборки и создания собственного репозитория. \ 
```

[vagrant@otusrpm ~]$ sudo yum install wget nano rpmdevtools rpm-build createrepo yum-utils gcc git
AlmaLinux 9 - AppStream                                                                                                                                                            451 kB/s | 8.1 MB     00:18    
AlmaLinux 9 - BaseOS                                                                                                                                                               986 kB/s | 3.5 MB     00:03    
AlmaLinux 9 - Extras                                                                                                                                                                14 kB/s |  17 kB     00:01    
Dependencies resolved.
===================================================================================================================================================================================================================
 Package                                                     Architecture                              Version                                                  Repository                                    Size
===================================================================================================================================================================================================================
Installing:
 createrepo_c                                                x86_64                                    0.20.1-2.el9                                             appstream                                     73 k
 gcc                                                         x86_64                                    11.4.1-2.1.el9.alma                                      appstream                                     32 M
 git                                                         x86_64                                    2.39.3-1.el9_2                                           appstream                                     61 k
 nano                                                        x86_64                                    5.6.1-5.el9                                              baseos                                       690 k
 rpm-build                                                   x86_64                                    4.16.1.3-25.el9                                          appstream                                     59 k
 rpmdevtools                                                 noarch                                    9.5-1.el9                                                appstream                                     75 k
 wget                                                        x86_64                                    1.21.1-7.el9                                             appstream                                    772 k
 yum-utils                                                   noarch                                    4.3.0-11.el9_3.alma.1                                    baseos                                        35 k
Upgrading:
 dnf-plugins-core                                            noarch                                    4.3.0-11.el9_3.alma.1                                    baseos                                        36 k
 python3-dnf-plugins-core                                    noarch                                    4.3.0-11.el9_3.alma.1                                    baseos                                       246 k
Installing dependencies:
 annobin                                                     x86_64                                    12.12-1.el9                                              appstream                                    976 k
 createrepo_c-libs                                           x86_64                                    0.20.1-2.el9                                             appstream                                     99 k
 debugedit                                                   x86_64                                    5.0-4.el9                                                appstream                                     75 k
 ed                                                          x86_64                                    1.14.2-12.el9                                            baseos                                        74 k
 elfutils                                                    x86_64                                    0.189-3.el9                                              baseos                                       526 k
 emacs-filesystem                                            noarch                                    1:27.2-9.el9                                             appstream                                    7.8 k
 gcc-plugin-annobin                                          x86_64                                    11.4.1-2.1.el9.alma                                      appstream                                     45 k
 gdb-minimal                                                 x86_64                                    10.2-11.el9                                              appstream                                    3.5 M
 git-core                                                    x86_64                                    2.39.3-1.el9_2                                           appstream                                    4.2 M
 git-core-doc                                                noarch                                    2.39.3-1.el9_2                                           appstream                                    2.6 M
 glibc-devel                                                 x86_64                                    2.34-83.el9_3.7                                          appstream                                     49 k
 info                                                        x86_64                                    6.7-15.el9                                               baseos                                       224 k
 kernel-headers                                              x86_64                                    5.14.0-362.13.1.el9_3                                    appstream                                    6.5 M
 libxcrypt-devel                                             x86_64                                    4.4.18-3.el9                                             appstream                                     28 k
 make                                                        x86_64                                    1:4.3-7.el9                                              baseos                                       531 k
 patch                                                       x86_64                                    2.7.6-16.el9                                             appstream                                    127 k
 perl-Error                                                  noarch                                    1:0.17029-7.el9                                          appstream                                     41 k
 perl-Git                                                    noarch                                    2.39.3-1.el9_2                                           appstream                                     37 k
 python3-argcomplete                                         noarch                                    1.12.0-5.el9                                             appstream                                     61 k
 python3-chardet                                             noarch                                    4.0.0-5.el9                                              baseos                                       209 k
 python3-idna                                                noarch                                    2.10-7.el9                                               baseos                                        92 k
 python3-pysocks                                             noarch                                    1.7.1-12.el9                                             baseos                                        34 k
 python3-requests                                            noarch                                    2.25.1-7.el9_2                                           baseos                                       113 k
 python3-urllib3                                             noarch                                    1.26.5-3.el9                                             baseos                                       188 k
 zstd                                                        x86_64                                    1.5.1-2.el9                                              baseos                                       546 k

Transaction Summary
===================================================================================================================================================================================================================
Install  33 Packages
Upgrade   2 Packages

Total download size: 55 M
Is this ok [y/N]: y
Downloading Packages:
(1/35): createrepo_c-libs-0.20.1-2.el9.x86_64.rpm                                                                                                                                  223 kB/s |  99 kB     00:00    
(2/35): createrepo_c-0.20.1-2.el9.x86_64.rpm                                                                                                                                       147 kB/s |  73 kB     00:00    
(3/35): emacs-filesystem-27.2-9.el9.noarch.rpm                                                                                                                                      40 kB/s | 7.8 kB     00:00    
(4/35): debugedit-5.0-4.el9.x86_64.rpm                                                                                                                                             255 kB/s |  75 kB     00:00    
(5/35): gcc-plugin-annobin-11.4.1-2.1.el9.alma.x86_64.rpm                                                                                                                          343 kB/s |  45 kB     00:00    
(6/35): annobin-12.12-1.el9.x86_64.rpm                                                                                                                                             374 kB/s | 976 kB     00:02    
(7/35): git-2.39.3-1.el9_2.x86_64.rpm                                                                                                                                              495 kB/s |  61 kB     00:00    
(8/35): gdb-minimal-10.2-11.el9.x86_64.rpm                                                                                                                                         1.1 MB/s | 3.5 MB     00:03    
(9/35): git-core-doc-2.39.3-1.el9_2.noarch.rpm                                                                                                                                     1.2 MB/s | 2.6 MB     00:02    
(10/35): glibc-devel-2.34-83.el9_3.7.x86_64.rpm                                                                                                                                    367 kB/s |  49 kB     00:00    
(11/35): git-core-2.39.3-1.el9_2.x86_64.rpm                                                                                                                                        888 kB/s | 4.2 MB     00:04    
(12/35): libxcrypt-devel-4.4.18-3.el9.x86_64.rpm                                                                                                                                   118 kB/s |  28 kB     00:00    
(13/35): patch-2.7.6-16.el9.x86_64.rpm                                                                                                                                             508 kB/s | 127 kB     00:00    
(14/35): perl-Error-0.17029-7.el9.noarch.rpm                                                                                                                                       112 kB/s |  41 kB     00:00    
(15/35): perl-Git-2.39.3-1.el9_2.noarch.rpm                                                                                                                                        128 kB/s |  37 kB     00:00    
(16/35): python3-argcomplete-1.12.0-5.el9.noarch.rpm                                                                                                                               172 kB/s |  61 kB     00:00    
(17/35): rpm-build-4.16.1.3-25.el9.x86_64.rpm                                                                                                                                      197 kB/s |  59 kB     00:00    
(18/35): rpmdevtools-9.5-1.el9.noarch.rpm                                                                                                                                          274 kB/s |  75 kB     00:00    
(19/35): kernel-headers-5.14.0-362.13.1.el9_3.x86_64.rpm                                                                                                                           1.5 MB/s | 6.5 MB     00:04    
(20/35): wget-1.21.1-7.el9.x86_64.rpm                                                                                                                                              820 kB/s | 772 kB     00:00    
(21/35): ed-1.14.2-12.el9.x86_64.rpm                                                                                                                                               383 kB/s |  74 kB     00:00    
(22/35): info-6.7-15.el9.x86_64.rpm                                                                                                                                                633 kB/s | 224 kB     00:00    
(23/35): elfutils-0.189-3.el9.x86_64.rpm                                                                                                                                           975 kB/s | 526 kB     00:00    
(24/35): make-4.3-7.el9.x86_64.rpm                                                                                                                                                 863 kB/s | 531 kB     00:00    
(25/35): nano-5.6.1-5.el9.x86_64.rpm                                                                                                                                               920 kB/s | 690 kB     00:00    
(26/35): python3-chardet-4.0.0-5.el9.noarch.rpm                                                                                                                                    573 kB/s | 209 kB     00:00    
(27/35): python3-idna-2.10-7.el9.noarch.rpm                                                                                                                                        292 kB/s |  92 kB     00:00    
(28/35): python3-pysocks-1.7.1-12.el9.noarch.rpm                                                                                                                                   140 kB/s |  34 kB     00:00    
(29/35): python3-requests-2.25.1-7.el9_2.noarch.rpm                                                                                                                                440 kB/s | 113 kB     00:00    
(30/35): python3-urllib3-1.26.5-3.el9.noarch.rpm                                                                                                                                   602 kB/s | 188 kB     00:00    
(31/35): yum-utils-4.3.0-11.el9_3.alma.1.noarch.rpm                                                                                                                                119 kB/s |  35 kB     00:00    
(32/35): dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch.rpm                                                                                                                          89 kB/s |  36 kB     00:00    
(33/35): zstd-1.5.1-2.el9.x86_64.rpm                                                                                                                                               791 kB/s | 546 kB     00:00    
(34/35): python3-dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch.rpm                                                                                                                 664 kB/s | 246 kB     00:00    
(35/35): gcc-11.4.1-2.1.el9.alma.x86_64.rpm                                                                                                                                        1.8 MB/s |  32 MB     00:17    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                              2.7 MB/s |  55 MB     00:20     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                           1/1 
  Installing       : python3-idna-2.10-7.el9.noarch                                                                                                                                                           1/37 
  Installing       : elfutils-0.189-3.el9.x86_64                                                                                                                                                              2/37 
  Installing       : git-core-2.39.3-1.el9_2.x86_64                                                                                                                                                           3/37 
  Installing       : gdb-minimal-10.2-11.el9.x86_64                                                                                                                                                           4/37 
  Installing       : emacs-filesystem-1:27.2-9.el9.noarch                                                                                                                                                     5/37 
  Installing       : debugedit-5.0-4.el9.x86_64                                                                                                                                                               6/37 
  Installing       : git-core-doc-2.39.3-1.el9_2.noarch                                                                                                                                                       7/37 
  Upgrading        : python3-dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                    8/37 
  Upgrading        : dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                            9/37 
  Installing       : zstd-1.5.1-2.el9.x86_64                                                                                                                                                                 10/37 
  Installing       : python3-pysocks-1.7.1-12.el9.noarch                                                                                                                                                     11/37 
  Installing       : python3-urllib3-1.26.5-3.el9.noarch                                                                                                                                                     12/37 
  Installing       : python3-chardet-4.0.0-5.el9.noarch                                                                                                                                                      13/37 
  Installing       : python3-requests-2.25.1-7.el9_2.noarch                                                                                                                                                  14/37 
  Installing       : make-1:4.3-7.el9.x86_64                                                                                                                                                                 15/37 
  Installing       : info-6.7-15.el9.x86_64                                                                                                                                                                  16/37 
  Installing       : ed-1.14.2-12.el9.x86_64                                                                                                                                                                 17/37 
  Installing       : patch-2.7.6-16.el9.x86_64                                                                                                                                                               18/37 
  Installing       : rpm-build-4.16.1.3-25.el9.x86_64                                                                                                                                                        19/37 
  Installing       : python3-argcomplete-1.12.0-5.el9.noarch                                                                                                                                                 20/37 
  Installing       : perl-Error-1:0.17029-7.el9.noarch                                                                                                                                                       21/37 
  Installing       : perl-Git-2.39.3-1.el9_2.noarch                                                                                                                                                          22/37 
  Installing       : git-2.39.3-1.el9_2.x86_64                                                                                                                                                               23/37 
  Installing       : kernel-headers-5.14.0-362.13.1.el9_3.x86_64                                                                                                                                             24/37 
  Installing       : libxcrypt-devel-4.4.18-3.el9.x86_64                                                                                                                                                     25/37 
  Installing       : glibc-devel-2.34-83.el9_3.7.x86_64                                                                                                                                                      26/37 
  Installing       : gcc-11.4.1-2.1.el9.alma.x86_64                                                                                                                                                          27/37 
  Running scriptlet: gcc-11.4.1-2.1.el9.alma.x86_64                                                                                                                                                          27/37 
  Installing       : createrepo_c-libs-0.20.1-2.el9.x86_64                                                                                                                                                   28/37 
  Installing       : createrepo_c-0.20.1-2.el9.x86_64                                                                                                                                                        29/37 
  Installing       : annobin-12.12-1.el9.x86_64                                                                                                                                                              30/37 
  Running scriptlet: annobin-12.12-1.el9.x86_64                                                                                                                                                              30/37 
  Installing       : gcc-plugin-annobin-11.4.1-2.1.el9.alma.x86_64                                                                                                                                           31/37 
  Running scriptlet: gcc-plugin-annobin-11.4.1-2.1.el9.alma.x86_64                                                                                                                                           31/37 
  Installing       : rpmdevtools-9.5-1.el9.noarch                                                                                                                                                            32/37 
  Installing       : yum-utils-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                                  33/37 
  Installing       : nano-5.6.1-5.el9.x86_64                                                                                                                                                                 34/37 
  Installing       : wget-1.21.1-7.el9.x86_64                                                                                                                                                                35/37 
  Cleanup          : dnf-plugins-core-4.3.0-11.el9_3.noarch                                                                                                                                                  36/37 
  Cleanup          : python3-dnf-plugins-core-4.3.0-11.el9_3.noarch                                                                                                                                          37/37 
  Running scriptlet: python3-dnf-plugins-core-4.3.0-11.el9_3.noarch                                                                                                                                          37/37 
  Verifying        : annobin-12.12-1.el9.x86_64                                                                                                                                                               1/37 
  Verifying        : createrepo_c-0.20.1-2.el9.x86_64                                                                                                                                                         2/37 
  Verifying        : createrepo_c-libs-0.20.1-2.el9.x86_64                                                                                                                                                    3/37 
  Verifying        : debugedit-5.0-4.el9.x86_64                                                                                                                                                               4/37 
  Verifying        : emacs-filesystem-1:27.2-9.el9.noarch                                                                                                                                                     5/37 
  Verifying        : gcc-11.4.1-2.1.el9.alma.x86_64                                                                                                                                                           6/37 
  Verifying        : gcc-plugin-annobin-11.4.1-2.1.el9.alma.x86_64                                                                                                                                            7/37 
  Verifying        : gdb-minimal-10.2-11.el9.x86_64                                                                                                                                                           8/37 
  Verifying        : git-2.39.3-1.el9_2.x86_64                                                                                                                                                                9/37 
  Verifying        : git-core-2.39.3-1.el9_2.x86_64                                                                                                                                                          10/37 
  Verifying        : git-core-doc-2.39.3-1.el9_2.noarch                                                                                                                                                      11/37 
  Verifying        : glibc-devel-2.34-83.el9_3.7.x86_64                                                                                                                                                      12/37 
  Verifying        : kernel-headers-5.14.0-362.13.1.el9_3.x86_64                                                                                                                                             13/37 
  Verifying        : libxcrypt-devel-4.4.18-3.el9.x86_64                                                                                                                                                     14/37 
  Verifying        : patch-2.7.6-16.el9.x86_64                                                                                                                                                               15/37 
  Verifying        : perl-Error-1:0.17029-7.el9.noarch                                                                                                                                                       16/37 
  Verifying        : perl-Git-2.39.3-1.el9_2.noarch                                                                                                                                                          17/37 
  Verifying        : python3-argcomplete-1.12.0-5.el9.noarch                                                                                                                                                 18/37 
  Verifying        : rpm-build-4.16.1.3-25.el9.x86_64                                                                                                                                                        19/37 
  Verifying        : rpmdevtools-9.5-1.el9.noarch                                                                                                                                                            20/37 
  Verifying        : wget-1.21.1-7.el9.x86_64                                                                                                                                                                21/37 
  Verifying        : ed-1.14.2-12.el9.x86_64                                                                                                                                                                 22/37 
  Verifying        : elfutils-0.189-3.el9.x86_64                                                                                                                                                             23/37 
  Verifying        : info-6.7-15.el9.x86_64                                                                                                                                                                  24/37 
  Verifying        : make-1:4.3-7.el9.x86_64                                                                                                                                                                 25/37 
  Verifying        : nano-5.6.1-5.el9.x86_64                                                                                                                                                                 26/37 
  Verifying        : python3-chardet-4.0.0-5.el9.noarch                                                                                                                                                      27/37 
  Verifying        : python3-idna-2.10-7.el9.noarch                                                                                                                                                          28/37 
  Verifying        : python3-pysocks-1.7.1-12.el9.noarch                                                                                                                                                     29/37 
  Verifying        : python3-requests-2.25.1-7.el9_2.noarch                                                                                                                                                  30/37 
  Verifying        : python3-urllib3-1.26.5-3.el9.noarch                                                                                                                                                     31/37 
  Verifying        : yum-utils-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                                  32/37 
  Verifying        : zstd-1.5.1-2.el9.x86_64                                                                                                                                                                 33/37 
  Verifying        : dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                           34/37 
  Verifying        : dnf-plugins-core-4.3.0-11.el9_3.noarch                                                                                                                                                  35/37 
  Verifying        : python3-dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                                                                                                   36/37 
  Verifying        : python3-dnf-plugins-core-4.3.0-11.el9_3.noarch                                                                                                                                          37/37 

Upgraded:
  dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                        python3-dnf-plugins-core-4.3.0-11.el9_3.alma.1.noarch                                                       
Installed:
  annobin-12.12-1.el9.x86_64                    createrepo_c-0.20.1-2.el9.x86_64         createrepo_c-libs-0.20.1-2.el9.x86_64     debugedit-5.0-4.el9.x86_64                      ed-1.14.2-12.el9.x86_64         
  elfutils-0.189-3.el9.x86_64                   emacs-filesystem-1:27.2-9.el9.noarch     gcc-11.4.1-2.1.el9.alma.x86_64            gcc-plugin-annobin-11.4.1-2.1.el9.alma.x86_64   gdb-minimal-10.2-11.el9.x86_64  
  git-2.39.3-1.el9_2.x86_64                     git-core-2.39.3-1.el9_2.x86_64           git-core-doc-2.39.3-1.el9_2.noarch        glibc-devel-2.34-83.el9_3.7.x86_64              info-6.7-15.el9.x86_64          
  kernel-headers-5.14.0-362.13.1.el9_3.x86_64   libxcrypt-devel-4.4.18-3.el9.x86_64      make-1:4.3-7.el9.x86_64                   nano-5.6.1-5.el9.x86_64                         patch-2.7.6-16.el9.x86_64       
  perl-Error-1:0.17029-7.el9.noarch             perl-Git-2.39.3-1.el9_2.noarch           python3-argcomplete-1.12.0-5.el9.noarch   python3-chardet-4.0.0-5.el9.noarch              python3-idna-2.10-7.el9.noarch  
  python3-pysocks-1.7.1-12.el9.noarch           python3-requests-2.25.1-7.el9_2.noarch   python3-urllib3-1.26.5-3.el9.noarch       rpm-build-4.16.1.3-25.el9.x86_64                rpmdevtools-9.5-1.el9.noarch    
  wget-1.21.1-7.el9.x86_64                      yum-utils-4.3.0-11.el9_3.alma.1.noarch   zstd-1.5.1-2.el9.x86_64                  

Complete!
```

Далее загружаем SRPM пакет NGINX
```

[vagrant@otusrpm ~]$ wget http://nginx.org/packages/rhel/9/SRPMS/nginx-1.24.0-1.el9.ngx.src.rpm
--2024-01-05 13:27:43--  http://nginx.org/packages/rhel/9/SRPMS/nginx-1.24.0-1.el9.ngx.src.rpm
Resolving nginx.org (nginx.org)... 3.125.197.172, 52.58.199.22, 2a05:d014:edb:5702::6, ...
Connecting to nginx.org (nginx.org)|3.125.197.172|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1129518 (1.1M) [application/x-redhat-package-manager]
Saving to: ‘nginx-1.24.0-1.el9.ngx.src.rpm’

nginx-1.24.0-1.el9.ngx.src.rpm                       100%[=====================================================================================================================>]   1.08M  1.48MB/s    in 0.7s    

2024-01-05 13:27:45 (1.48 MB/s) - ‘nginx-1.24.0-1.el9.ngx.src.rpm’ saved [1129518/1129518]
```

Устанавливаем этот пакет
```

[vagrant@otusrpm ~]$ rpm -i nginx-1.24.0-1.el9.ngx.src.rpm
```
После установки у нас появляется папка rpmbuild содержимое которой будет использоваться для сборки пакет NGINX.
```

[vagrant@otusrpm ~]$ ls -1
nginx-1.24.0-1.el9.ngx.src.rpm
rpmbuild
```

Загружаем исходники openssl
```

[vagrant@otusrpm ~]$ git clone git://git.openssl.org/openssl.git
Cloning into 'openssl'...
remote: Enumerating objects: 469790, done.
remote: Counting objects: 100% (11929/11929), done.
remote: Compressing objects: 100% (4943/4943), done.
remote: Total 469790 (delta 8865), reused 9149 (delta 6965), pack-reused 457861
Receiving objects: 100% (469790/469790), 116.11 MiB | 3.36 MiB/s, done.
Resolving deltas: 100% (350972/350972), done.
```


Теперь нам нужно добавить openssl в файл nginx.spec, который используется при сборке пакета. \
Для этого открываем файл
```

nano ~/rpmbuild/SPECS/nginx.spec
```
И в разделе %build в блок ./configure добавляем: --with-openssl=/home/vagrant/openssl \
```

%build
./configure %{BASE_CONFIGURE_ARGS} \
    --with-cc-opt="%{WITH_CC_OPT}" \
    --with-ld-opt="%{WITH_LD_OPT}" \
    --with-openssl=/home/vagrant/openssl \
    --with-debug
make %{?_smp_mflags}
%{__mv} %{bdir}/objs/nginx \
    %{bdir}/objs/nginx-debug
./configure %{BASE_CONFIGURE_ARGS} \
    --with-cc-opt="%{WITH_CC_OPT}" \
    --with-ld-opt="%{WITH_LD_OPT}"
make %{?_smp_mflags}
```

Устанавливаем зависимости для сборки нашего кастомного пакета.
```

[vagrant@otusrpm ~]$ sudo yum-builddep rpmbuild/SPECS/nginx.spec
Last metadata expiration check: 0:10:36 ago on Fri Jan  5 13:25:28 2024.
Package openssl-devel-1:3.0.7-24.el9.x86_64 is already installed.
Package systemd-252-18.el9.x86_64 is already installed.
Package zlib-devel-1.2.11-40.el9.x86_64 is already installed.
Dependencies resolved.
===================================================================================================================================================================================================================
 Package                                             Architecture                                   Version                                                Repository                                         Size
===================================================================================================================================================================================================================
Installing:
 pcre2-devel                                         x86_64                                         10.40-2.el9                                            appstream                                         474 k
Installing dependencies:
 pcre2-utf16                                         x86_64                                         10.40-2.el9                                            appstream                                         216 k
 pcre2-utf32                                         x86_64                                         10.40-2.el9                                            appstream                                         205 k

Transaction Summary
===================================================================================================================================================================================================================
Install  3 Packages

Total download size: 894 k
Installed size: 3.0 M
Is this ok [y/N]: y
Downloading Packages:
(1/3): pcre2-utf16-10.40-2.el9.x86_64.rpm                                                                                                                                          406 kB/s | 216 kB     00:00    
(2/3): pcre2-utf32-10.40-2.el9.x86_64.rpm                                                                                                                                          358 kB/s | 205 kB     00:00    
(3/3): pcre2-devel-10.40-2.el9.x86_64.rpm                                                                                                                                          703 kB/s | 474 kB     00:00    
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                              534 kB/s | 894 kB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                           1/1 
  Installing       : pcre2-utf32-10.40-2.el9.x86_64                                                                                                                                                            1/3 
  Installing       : pcre2-utf16-10.40-2.el9.x86_64                                                                                                                                                            2/3 
  Installing       : pcre2-devel-10.40-2.el9.x86_64                                                                                                                                                            3/3 
  Running scriptlet: pcre2-devel-10.40-2.el9.x86_64                                                                                                                                                            3/3 
  Verifying        : pcre2-devel-10.40-2.el9.x86_64                                                                                                                                                            1/3 
  Verifying        : pcre2-utf16-10.40-2.el9.x86_64                                                                                                                                                            2/3 
  Verifying        : pcre2-utf32-10.40-2.el9.x86_64                                                                                                                                                            3/3 

Installed:
  pcre2-devel-10.40-2.el9.x86_64                                        pcre2-utf16-10.40-2.el9.x86_64                                        pcre2-utf32-10.40-2.el9.x86_64                                       

Complete!
```


Запускаем сборку пакета
```

rpmbuild -bb rpmbuild/SPECS/nginx.spec
```
Процесс сборки довольно долгий, в моём случае занимает порядка 11 минут. Ждём окончания сборки. 
```

Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.pnAWXk
+ umask 022
+ cd /home/vagrant/rpmbuild/BUILD
+ cd nginx-1.24.0
+ /usr/bin/rm -rf /home/vagrant/rpmbuild/BUILDROOT/nginx-1.24.0-1.el9.ngx.x86_64
+ RPM_EC=0
++ jobs -p
+ exit 0
```

После проверяем наличие собранного пакета:
```

[vagrant@otusrpm ~]$ ls -1 rpmbuild/RPMS/x86_64/
nginx-1.24.0-1.el9.ngx.x86_64.rpm
nginx-debuginfo-1.24.0-1.el9.ngx.x86_64.rpm
```

Попробуем установить наш пакет.
```

[vagrant@otusrpm ~]$ sudo yum localinstall rpmbuild/RPMS/x86_64/nginx-1.24.0-1.el9.ngx.x86_64.rpm
Last metadata expiration check: 0:26:10 ago on Fri Jan  5 13:25:28 2024.
Dependencies resolved.
===================================================================================================================================================================================================================
 Package                                      Architecture                                  Version                                                      Repository                                           Size
===================================================================================================================================================================================================================
Installing:
 nginx                                        x86_64                                        1:1.24.0-1.el9.ngx                                           @commandline                                        2.8 M

Transaction Summary
===================================================================================================================================================================================================================
Install  1 Package

Total size: 2.8 M
Installed size: 8.8 M
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                           1/1 
  Running scriptlet: nginx-1:1.24.0-1.el9.ngx.x86_64                                                                                                                                                           1/1 
  Installing       : nginx-1:1.24.0-1.el9.ngx.x86_64                                                                                                                                                           1/1 
  Running scriptlet: nginx-1:1.24.0-1.el9.ngx.x86_64                                                                                                                                                           1/1 
----------------------------------------------------------------------

Thanks for using nginx!

Please find the official documentation for nginx here:
* https://nginx.org/en/docs/

Please subscribe to nginx-announce mailing list to get
the most important news about nginx:
* https://nginx.org/en/support.html

Commercial subscriptions for nginx are available on:
* https://nginx.com/products/

----------------------------------------------------------------------

  Verifying        : nginx-1:1.24.0-1.el9.ngx.x86_64                                                                                                                                                           1/1 

Installed:
  nginx-1:1.24.0-1.el9.ngx.x86_64                                                                                                                                                                                  

Complete!
```

Запускаем nginx и проверяем его статус.
```

[vagrant@otusrpm ~]$ sudo systemctl start nginx
[vagrant@otusrpm ~]$ systemctl status nginx
● nginx.service - nginx - high performance web server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Fri 2024-01-05 13:53:00 UTC; 19s ago
       Docs: http://nginx.org/en/docs/
    Process: 32563 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
   Main PID: 32564 (nginx)
      Tasks: 2 (limit: 11928)
     Memory: 1.9M
        CPU: 6ms
     CGroup: /system.slice/nginx.service
             ├─32564 "nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf"
             └─32565 "nginx: worker process"
```
Пакет успешно собран и установлен. Сервис запущен.


## Создать свой репозиторий и разместить там ранее собранный RPM.

### Решение


