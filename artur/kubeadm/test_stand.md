# Тестовый стенд.

В моём стенде будет 8 виртуальных машин.

* **master.k.erushnikov.ru** (RAM 1GB, CPU 1) - вспомогательная машина. 
  * Кеширующий DNS сервер для машин тестового стенда.
  * Поддержка зоны k.erushnikov.ru.
* **control{1,2,3}.k.erushnikov.ru** (RAM 4GB, CPU 4) - control nodes кластера kubernetes.
* **worker{1,2,3}.k.erushnikov.ru** (RAM 8GB, CPU 6) - общие worker nodes кластера kubernetes.
* **db1.k.erushnikov.ru (RAM 4GB, CPU 4)** - дополнительная worker node.

На всех машинах, кроме master, установлен AlmaLinux 8.6.

Маршрут по умолчанию на всех машинах идёт на мой локальный роутер 192.168.128.1.

Клиенты DNS настроены на машину master.

## DNS сервер

На машине master установлен DNS server BIND.

В файл `/etc/named.conf` добавлена поддержка двух зон: `k.erushnikov.ru` и `218.168.192.in-addr.arpa`.

```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };
        recursion yes;
        dnssec-enable no;
        dnssec-validation no;
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        include "/etc/crypto-policies/back-ends/bind.config";
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
zone "." IN {
        type hint;
        file "named.ca";
};
zone "k.erushnikov.ru" IN {
        type master;
        file "k.erushnikov.ru";
};
zone "218.168.192.in-addr.arpa" IN {
        type master;
        file "218";
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Файлы описания зон находятся в директории `/var/named`.

k.erushnikov.ru:

```
$TTL 86400
@ IN SOA master.k.erushnikov.ru. artur.kryukov.biz. (
                                                2022120100 ;Serial
                                                3600 ;Refresh
                                                1800 ;Retry
                                                604800 ;Expire
                                                86400 ;Minimum TTL
)

@ IN NS master

master          IN      A       192.168.218.170
control1        IN      A       192.168.218.171
control2        IN      A       192.168.218.172
control3        IN      A       192.168.218.173
worker1         IN      A       192.168.218.174
worker2         IN      A       192.168.218.175
worker3         IN      A       192.168.218.176
db1             IN      A       192.168.218.177

metallb         IN      A       192.168.218.180
argocd          IN      CNAME   metallb
keycloak        IN      CNAME   metallb

kubeapi         IN      A       192.168.218.189
```

218:

```
$TTL 86400
@ IN SOA master.k.erushnikov.ru. artur.kryukov.biz. (
                                            2022120100 ;Serial
                                            3600 ;Refresh
                                            1800 ;Retry
                                            604800 ;Expire
                                            86400 ;Minimum TTL
)

@ IN NS master.k.erushnikov.ru.

170     IN      PTR master.k.erushnikov.ru.
171     IN      PTR     control1.k.erushnikov.ru.
172     IN      PTR     control2.k.erushnikov.ru.
173     IN      PTR     control3.k.erushnikov.ru.
174     IN      PTR     worker1.k.erushnikov.ru.
175     IN      PTR     worker2.k.erushnikov.ru.
176     IN      PTR     worker3.k.erushnikov.ru.
177     IN      PTR     db1.k.erushnikov.ru.

180     IN      PTR     metallb.k.erushnikov.ru.
189     IN      PTR     kubeapi.k.erushnikov.ru.
```

## NFS сервер

Сервер раздаёт по сети директорию `/var/nfs-disk`.

Конфигурационный файл `/etc/exports`:

```
/var/nfs-disk 192.168.218.0/24(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)
```

## Рабочая станция Windows

Рабочая машина на Windows. Поскольку она использует DNS сервер провайдера, для удобства все машины тестового стенда 
описаны в файле `C:\Windows\System32\drivers\etc\hosts`:

```
192.168.218.170 master.k.erushnikov.ru
192.168.218.171 control1.k.erushnikov.ru
192.168.218.172 control2.k.erushnikov.ru
192.168.218.173 control3.k.erushnikov.ru
192.168.218.174 worker1.k.erushnikov.ru
192.168.218.175 worker2.k.erushnikov.ru
192.168.218.176 worker3.k.erushnikov.ru
192.168.218.177 db1.k.erushnikov.ru
192.168.218.180 metallb.k.erushnikov.ru
192.168.218.189 kubeapi.k.erushnikov.ru
```

Включен wsl2, в котором установлен дистрибутив Ubuntu 22.04.1 LTS.

В Ubuntu установлен ansible:

```shell
apt install python3.10-venv
python3 -m pip install ansible
```

Ansible можно ставить в venv.

```shell
apt install python3.10-venv
python3 -m venv venv
. ~/venv/bin/activate
python3 -m pip install ansible
```

Все файлы проекта будут находиться в домашней директории пользователя в Ubuntu.