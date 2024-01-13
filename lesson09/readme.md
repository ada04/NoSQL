root@ubuntu2204:~# grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
SSE 4.2 supported
root@ubuntu2204:~# sudo apt-get install -y apt-transport-https ca-certificates dirmngr
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20230311ubuntu0.22.04.1).
ca-certificates set to manually installed.
dirmngr is already the newest version (2.2.27-3ubuntu2.1).
dirmngr set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,510 B of archives.
After this operation, 169 kB of additional disk space will be used.
Err:1 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe amd64 apt-transport-https all 2.4.10
  404  Not Found [IP: 147.75.80.249 443]
E: Failed to fetch https://mirrors.edge.kernel.org/ubuntu/pool/universe/a/apt/apt-transport-https_2.4.10_all.deb  404  Not Found [IP: 147.75.80.249 443]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
root@ubuntu2204:~# sudo apt-get install -y apt-transport-https ca-certificates dirmngr
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20230311ubuntu0.22.04.1).
ca-certificates set to manually installed.
dirmngr is already the newest version (2.2.27-3ubuntu2.1).
dirmngr set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,510 B of archives.
After this operation, 169 kB of additional disk space will be used.
Err:1 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe amd64 apt-transport-https all 2.4.10
  404  Not Found [IP: 147.75.80.249 443]
E: Failed to fetch https://mirrors.edge.kernel.org/ubuntu/pool/universe/a/apt/apt-transport-https_2.4.10_all.deb  404  Not Found [IP: 147.75.80.249 443]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
root@ubuntu2204:~# sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
Executing: /tmp/apt-key-gpghome.AkFM0twwtT/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
gpg: key 8919F6BD2B48D754: public key "ClickHouse Inc. Repositories Key <packages@clickhouse.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
root@ubuntu2204:~# echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
deb https://packages.clickhouse.com/deb stable main
root@ubuntu2204:~# sudo apt-get update
Hit:1 https://mirrors.edge.kernel.org/ubuntu jammy InRelease
Get:2 https://mirrors.edge.kernel.org/ubuntu jammy-updates InRelease [119 kB]
Get:3 https://mirrors.edge.kernel.org/ubuntu jammy-backports InRelease [109 kB]
Get:4 https://mirrors.edge.kernel.org/ubuntu jammy-security InRelease [110 kB]
Get:5 https://mirrors.edge.kernel.org/ubuntu jammy-updates/main amd64 Packages [1,277 kB]
Get:6 https://packages.clickhouse.com/deb stable InRelease [2,490 B]
Get:7 https://mirrors.edge.kernel.org/ubuntu jammy-updates/main Translation-en [261 kB]
Get:8 https://packages.clickhouse.com/deb stable/main amd64 Packages [110 kB]
Get:9 https://mirrors.edge.kernel.org/ubuntu jammy-updates/main amd64 c-n-f Metadata [16.1 kB]
Get:10 https://mirrors.edge.kernel.org/ubuntu jammy-updates/restricted amd64 Packages [1,272 kB]
Get:11 https://mirrors.edge.kernel.org/ubuntu jammy-updates/restricted Translation-en [207 kB]
Get:12 https://mirrors.edge.kernel.org/ubuntu jammy-updates/restricted amd64 c-n-f Metadata [520 B]
Get:13 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe amd64 Packages [1,023 kB]
Get:14 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe Translation-en [228 kB]
Get:15 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe amd64 c-n-f Metadata [22.1 kB]
Get:16 https://mirrors.edge.kernel.org/ubuntu jammy-updates/multiverse amd64 Packages [42.1 kB]
Get:17 https://mirrors.edge.kernel.org/ubuntu jammy-updates/multiverse Translation-en [10.1 kB]
Get:18 https://mirrors.edge.kernel.org/ubuntu jammy-updates/multiverse amd64 c-n-f Metadata [472 B]
Get:19 https://mirrors.edge.kernel.org/ubuntu jammy-backports/main amd64 Packages [41.7 kB]
Get:20 https://mirrors.edge.kernel.org/ubuntu jammy-backports/main amd64 c-n-f Metadata [388 B]
Get:21 https://mirrors.edge.kernel.org/ubuntu jammy-backports/universe amd64 Packages [24.3 kB]
Get:22 https://mirrors.edge.kernel.org/ubuntu jammy-backports/universe Translation-en [16.5 kB]
Get:23 https://mirrors.edge.kernel.org/ubuntu jammy-backports/universe amd64 c-n-f Metadata [644 B]
Get:24 https://mirrors.edge.kernel.org/ubuntu jammy-security/main amd64 Packages [1,062 kB]
Get:25 https://mirrors.edge.kernel.org/ubuntu jammy-security/main Translation-en [201 kB]
Get:26 https://mirrors.edge.kernel.org/ubuntu jammy-security/main amd64 c-n-f Metadata [11.4 kB]
Get:27 https://mirrors.edge.kernel.org/ubuntu jammy-security/restricted amd64 Packages [1,244 kB]
Get:28 https://mirrors.edge.kernel.org/ubuntu jammy-security/restricted Translation-en [203 kB]
Get:29 https://mirrors.edge.kernel.org/ubuntu jammy-security/restricted amd64 c-n-f Metadata [520 B]
Get:30 https://mirrors.edge.kernel.org/ubuntu jammy-security/universe amd64 Packages [826 kB]
Get:31 https://mirrors.edge.kernel.org/ubuntu jammy-security/universe Translation-en [156 kB]
Get:32 https://mirrors.edge.kernel.org/ubuntu jammy-security/universe amd64 c-n-f Metadata [16.8 kB]
Get:33 https://mirrors.edge.kernel.org/ubuntu jammy-security/multiverse amd64 Packages [37.1 kB]
Get:34 https://mirrors.edge.kernel.org/ubuntu jammy-security/multiverse Translation-en [7,476 B]
Get:35 https://mirrors.edge.kernel.org/ubuntu jammy-security/multiverse amd64 c-n-f Metadata [260 B]
Fetched 8,660 kB in 8s (1,062 kB/s)
Reading package lists... Done
W: https://packages.clickhouse.com/deb/dists/stable/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
root@ubuntu2204:~# sudo apt-get install -y apt-transport-https ca-certificates dirmngr
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20230311ubuntu0.22.04.1).
ca-certificates set to manually installed.
dirmngr is already the newest version (2.2.27-3ubuntu2.1).
dirmngr set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 96 not upgraded.
Need to get 1,510 B of archives.
After this operation, 170 kB of additional disk space will be used.
Get:1 https://mirrors.edge.kernel.org/ubuntu jammy-updates/universe amd64 apt-transport-https all 2.4.11 [1,510 B]
Fetched 1,510 B in 0s (7,250 B/s)
Selecting previously unselected package apt-transport-https.
(Reading database ... 76032 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.4.11_all.deb ...
Unpacking apt-transport-https (2.4.11) ...
Setting up apt-transport-https (2.4.11) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@ubuntu2204:~# sudo apt-get install -y clickhouse-server clickhouse-client
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  clickhouse-common-static
Suggested packages:
  clickhouse-common-static-dbg
The following NEW packages will be installed:
  clickhouse-client clickhouse-common-static clickhouse-server
0 upgraded, 3 newly installed, 0 to remove and 96 not upgraded.
Need to get 290 MB of archives.
After this operation, 904 MB of additional disk space will be used.
Get:1 https://packages.clickhouse.com/deb stable/main amd64 clickhouse-common-static amd64 23.12.2.59 [290 MB]
Get:2 https://packages.clickhouse.com/deb stable/main amd64 clickhouse-client amd64 23.12.2.59 [137 kB]
Get:3 https://packages.clickhouse.com/deb stable/main amd64 clickhouse-server amd64 23.12.2.59 [164 kB]
Fetched 290 MB in 24s (12.2 MB/s)
Selecting previously unselected package clickhouse-common-static.
(Reading database ... 76036 files and directories currently installed.)
Preparing to unpack .../clickhouse-common-static_23.12.2.59_amd64.deb ...
Unpacking clickhouse-common-static (23.12.2.59) ...
Selecting previously unselected package clickhouse-client.
Preparing to unpack .../clickhouse-client_23.12.2.59_amd64.deb ...
Unpacking clickhouse-client (23.12.2.59) ...
Selecting previously unselected package clickhouse-server.
Preparing to unpack .../clickhouse-server_23.12.2.59_amd64.deb ...
Unpacking clickhouse-server (23.12.2.59) ...
Setting up clickhouse-common-static (23.12.2.59) ...
Setting up clickhouse-server (23.12.2.59) ...
ClickHouse binary is already located at /usr/bin/clickhouse
Symlink /usr/bin/clickhouse-server already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-server to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-client already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-client to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-local already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-local to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-benchmark already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-benchmark to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-copier already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-copier to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-obfuscator already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-obfuscator to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-git-import to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-compressor already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-compressor to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-format already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-format to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-extract-from-config already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-extract-from-config to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-keeper already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-keeper to /usr/bin/clickhouse.
Symlink /usr/bin/clickhouse-keeper-converter already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-keeper-converter to /usr/bin/clickhouse.
Creating symlink /usr/bin/clickhouse-disks to /usr/bin/clickhouse.
Creating symlink /usr/bin/ch to /usr/bin/clickhouse.
Creating symlink /usr/bin/chl to /usr/bin/clickhouse.
Creating symlink /usr/bin/chc to /usr/bin/clickhouse.
Creating clickhouse group if it does not exist.
 groupadd -r clickhouse
Creating clickhouse user if it does not exist.
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse
Will set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.
Creating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.
Creating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.
Config file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.
/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.
/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.
Users config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.
Creating log directory /var/log/clickhouse-server/.
Creating data directory /var/lib/clickhouse/.
Creating pid directory /var/run/clickhouse-server.
 chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'
 chown -R clickhouse:clickhouse '/var/run/clickhouse-server'
 chown  clickhouse:clickhouse '/var/lib/clickhouse/'
 groupadd -r clickhouse-bridge
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'
Enter password for default user:
Password for default user is saved in file /etc/clickhouse-server/users.d/default-password.xml.
Setting capabilities for clickhouse binary. This is optional.
 chown -R clickhouse:clickhouse '/etc/clickhouse-server'

ClickHouse has been successfully installed.

Start clickhouse-server with:
 sudo clickhouse start

Start clickhouse-client with:
 clickhouse-client --password

Synchronizing state of clickhouse-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable clickhouse-server
Created symlink /etc/systemd/system/multi-user.target.wants/clickhouse-server.service → /lib/systemd/system/clickhouse-server.service.
Setting up clickhouse-client (23.12.2.59) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@ubuntu2204:~# sudo service clickhouse-server start
root@ubuntu2204:~# clickhouse-client
ClickHouse client version 23.12.2.59 (official build).
Connecting to localhost:9000 as user default.
Password for user (default):
Connecting to localhost:9000 as user default.
Code: 516. DB::Exception: Received from localhost:9000. DB::Exception: default: Authentication failed: password is incorrect, or there is no user with such name.

If you have installed ClickHouse and forgot password you can reset it in the configuration file.
The password for default user is typically located at /etc/clickhouse-server/users.d/default-password.xml
and deleting this file will reset the password.
See also /etc/clickhouse-server/users.xml on the server where ClickHouse is installed.

. (AUTHENTICATION_FAILED)

root@ubuntu2204:~# clickhouse-client
ClickHouse client version 23.12.2.59 (official build).
Connecting to localhost:9000 as user default.
Password for user (default):
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.12.2.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. Check /proc/sys/kernel/task_delayacct
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.

ubuntu2204.localdomain :) SELECT 1

SELECT 1

Query id: 5ee5fd3d-fe78-49f5-a4a9-a92552e815a6

┌─1─┐
│ 1 │
└───┘

1 row in set. Elapsed: 0.002 sec.

ubuntu2204.localdomain :) exit
Bye.
root@ubuntu2204:~#
