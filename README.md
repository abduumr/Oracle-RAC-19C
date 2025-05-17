# Oracle-RAC-19C

```
vi /etc/hosts
cat /etc/hosts

# Public

192.168.50.67 rac1.localdomain rac1
192.168.50.68 rac2.localdomain rac2

# Private

192.168.51.111 rac1-priv.localdomain rac1-priv
192.168.51.112 rac2-priv.localdomain rac2-priv
# Virtual

192.168.50.31 rac1-vip.localdomain rac1-vip
192.168.50.32 rac2-vip.localdomain rac2-vip

# SCAN

192.168.50.41 rac-scan.localdomain rac-scan
192.168.50.42 rac-scan.localdomain rac-scan
192.168.50.43 rac-scan.localdomain rac-scan
                    
```

```
lsblk
```

```
fdisk /dev/sdb
```

```
pvcreate /dev/sdb1
vgcreate database_vg /dev/sdb1
lvcreate -l 100%FREE -n database_lv database_vg
mkfs.ext4 /dev/database_vg/database_lv
mkdir /u01
mount /dev/database_vg/database_lv /u01
lsblk
```

```
vi /etc/fstab
/dev/mapper/oravg-u01   /u01   ext4  defaults   0 0
```

```
dnf makecache
```

```
dnf install oracle-database-preinstall-19c -y

```

```
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54329 asmadmin

```

```
id oracle | grep asm
usermod -a -G asmdba oracle
id oracle | grep asm

```

```
useradd -u 54331 -g oinstall -G dba,asmdba,asmadmin,asmoper,racdba grid
id grid

```

```
echo password | passwd oracle --stdin 
echo password | passwd grid --stdin

```

```
mkdir -p /u01/19c/oracle/ora_base/db_home
mkdir -p /u01/19c/grid/grid_home
mkdir  -p /u01/19c/grid/grid_base
chown -R oracle:oinstall /u01
chown -R grid:oinstall /u01/19c/grid/
chmod -R 775 /u01

```

```
vim /etc/chrony.conf

server 0.jp.pool.ntp.org iburst
server 1.jp.pool.ntp.org iburst
server 2.jp.pool.ntp.org iburst
server 3.jp.pool.ntp.org iburst

```

```
systemctl restart chronyd
chronyc sources
systemctl status chronyd


```

```
sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config
cat /etc/selinux/config

```

```
setenforce Permissive

cp /etc/security/limits.d/oracle-database-preinstall-19c.conf /etc/security/limits.d/grid-database-preinstall-19c.conf

vim /etc/security/limits.d/grid-database-preinstall-19c.conf

:%s/oracle/grid/g
```

```
systemctl stop firewalld
systemctl disable firewalld

```

```
su - grid
cp .bash_profile .bash_profile.bkp

```

```
cat > /home/grid/.grid_env <<EOF
host=\$(hostname -s)
if [ \$host == "rac1" ]; then 
    ORA_SID=+ASM1
elif [ \$host == "rac2" ]; then
    ORA_SID=+ASM2
else 
 echo "Host not meet the option $host"
fi
 
# User specific environment and startup programs

ORACLE_SID=\$ORA_SID; export ORACLE_SID
ORACLE_BASE=/u01/19c/grid/grid_base; export ORACLE_BASE
ORACLE_HOME=/u01/19c/grid/grid_home; export ORACLE_HOME
ORACLE_TERM=xterm; export ORACLE_TERM
JAVA_HOME=/usr/bin/java; export JAVA_HOME
TNS_ADMIN=\$ORACLE_HOME/network/admin; export TNS_ADMIN

PATH=.:\${JAVA_HOME}/bin:\${PATH}:\$HOME/bin:\$ORACLE_HOME/bin
PATH=\${PATH}:/usr/bin:/bin:/usr/local/bin
export PATH
umask 022
EOF

```

```
echo '. ~/.grid_env' >> /home/grid/.bash_profile

```

```
source .bash_profile
env | grep ORACLE
exit
```

```
su - oracle
cp .bash_profile .bash_profile.bkp

```

```
cat > /home/oracle/.db19_env <<EOF
host=\$(hostname -s)
if [ \$host == "rac1" ]; then 
    ORA_SID=PROD1
elif [ \$host == "rac2" ]; then
    ORA_SID=PROD2
else 
 echo "Host not meet the option $host"
fi

ORACLE_HOSTNAME=\$HOSTNAME; export ORACLE_HOSTNAME
ORACLE_SID=\$ORA_SID; export ORACLE_SID
ORACLE_UNQNAME=prod; export ORACLE_UNQNAME
ORACLE_BASE=/u01/19c/oracle/ora_base; export ORACLE_BASE
ORACLE_HOME=/u01/19c/oracle/ora_base/db_home; export ORACLE_HOME
ORACLE_TERM=xterm; export ORACLE_TERM

JAVA_HOME=/usr/bin/java; export JAVA_HOME
NLS_DATE_FORMAT="DD-MON-YYYY HH24:MI:SS"; export NLS_DATE_FORMAT
TNS_ADMIN=\$ORACLE_HOME/network/admin; export TNS_ADMIN
PATH=.:\${JAVA_HOME}/bin:\${PATH}:\$HOME/bin:\$ORACLE_HOME/bin
PATH=\${PATH}:/usr/bin:/bin:/usr/local/bin
export PATH

LD_LIBRARY_PATH=\$ORACLE_HOME/lib
LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:\$ORACLE_HOME/oracm/lib
LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:/lib:/usr/lib:/usr/local/lib
export LD_LIBRARY_PATH

CLASSPATH=\$ORACLE_HOME/JRE:\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib:\$ORACLE_HOME/network/jlib
export CLASSPATH

TEMP=/tmp ;export TMP
TMPDIR=\$tmp ; export TMPDIR
EOF

```

```
cat  >> /home/oracle/.bash_profile <<EOF
source ~/.db19_env
alias db19_env='. ~/.db19_env'
EOF

```

```
source /home/oracle/.bash_profile
env | grep ORACLE_
exit

```

```
systemctl stop tuned.service ktune.service
systemctl stop firewalld.service
systemctl stop postfix.service
systemctl stop avahi-daemon.service
systemctl stop avahi-daemon.socket
systemctl stop atd.service
systemctl stop bluetooth.service
systemctl stop wpa_supplicant.service
systemctl stop accounts-daemon.service
systemctl stop ModemManager.service
systemctl stop debug-shell.service
systemctl stop rtkit-daemon.service
systemctl stop rpcbind.service
systemctl stop rpcbind.socket
systemctl stop rngd.service
systemctl stop upower.service
systemctl stop rhsmcertd.service
systemctl stop colord.service
systemctl stop libstoragemgmt.service
systemctl stop ksmtuned.service
systemctl stop brltty.service
systemctl disable tuned.service ktune.service
systemctl disable firewalld.service
systemctl disable postfix.service
systemctl disable avahi-daemon.socket
systemctl disable avahi-daemon.service
systemctl disable bluetooth.service
systemctl disable wpa_supplicant.service
systemctl disable accounts-daemon.service
systemctl disable atd.service cups.service
systemctl disable ModemManager.service
systemctl disable debug-shell.service
systemctl disable rpcbind.service
systemctl disable rpcbind.socket
systemctl disable rngd.service
systemctl disable upower.service
systemctl disable rhsmcertd.service
systemctl disable rtkit-daemon.service
systemctl disable mcelog.service
systemctl disable colord.service
systemctl disable libstoragemgmt.service
systemctl disable ksmtuned.service
systemctl disable brltty.service


```

```
systemctl stop avahi-daemon.socket 
systemctl disable avahi-daemon.socket 
systemctl status avahi-daemon.socket


```

```
dnf install bind bind-utils -y

```

```
systemctl stop named
systemctl disable named 
cp /etc/named.conf /etc/named.conf.bkp

```

```
vim dns.sh

export DNS_IP="192.168.50.67"
export DNS_DOMAIN="localdomain"
export DNS_NETWORK="192.168.50.0/24"
export DNS_BACKWARD="50.168.192.in-addr.arpa"
export DNS_FORWARD=$DNS_DOMAIN
export DNS_BACKWARD_FILE="backward.$DNS_DOMAIN"
export DNS_FORWARD_FILE="forward.$DNS_DOMAIN"
export DNS_HOSTNAME="rac1"
export DNS_FQDN=$DNS_HOSTNAME.$DNS_DOMAIN

cat > /etc/named.conf <<EOF
options {
        listen-on port 53 { 127.0.0.1; $DNS_IP; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; $DNS_NETWORK; };

        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

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

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

// forward zone
zone "$DNS_DOMAIN" IN {
        type master;
        file "$DNS_FORWARD_FILE";
        allow-update { none; };
        allow-query { any; };
};

// reverse zone
zone "$DNS_BACKWARD" IN {
        type master;
        file "$DNS_BACKWARD_FILE";
        allow-update { none; };
        allow-query { any; };
};
EOF

# Buat file zona forward
cat > /var/named/$DNS_FORWARD_FILE <<EOF
\$TTL 86400
@   IN  SOA $DNS_FQDN. root.$DNS_DOMAIN. (
        2025051401 ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Minimum TTL

    IN  NS  $DNS_FQDN.

rac1          IN A 192.168.50.67
rac2          IN A 192.168.50.68
rac1-priv     IN A 192.168.51.111
rac2-priv     IN A 192.168.51.112
rac1-vip      IN A 192.168.50.31
rac2-vip      IN A 192.168.50.32
rac-scan      IN A 192.168.50.41
rac-scan      IN A 192.168.50.42
rac-scan      IN A 192.168.50.43
EOF

# Buat file zona reverse
cat > /var/named/$DNS_BACKWARD_FILE <<EOF
\$TTL 86400
@   IN  SOA $DNS_FQDN. root.$DNS_DOMAIN. (
        2025051401 ; Serial
        3600       ; Refresh
        1800       ; Retry
        604800     ; Expire
        86400 )    ; Minimum TTL

    IN  NS  $DNS_FQDN.

67    IN PTR rac1.$DNS_DOMAIN.
68    IN PTR rac2.$DNS_DOMAIN.
31    IN PTR rac1-vip.$DNS_DOMAIN.
32    IN PTR rac2-vip.$DNS_DOMAIN.
41    IN PTR rac-scan.$DNS_DOMAIN.
42    IN PTR rac-scan.$DNS_DOMAIN.
43    IN PTR rac-scan.$DNS_DOMAIN.
EOF


```

```
chmod +x dns.sh
ls -l
./dns.sh
```

```
ls -al /var/named/

chown named:named /var/named/forward.localdomain
chown named:named /var/named/backward.localdomain

ls -al /var/named/

```

```
# check the configurations 
named-checkconf
named-checkzone localdomain /var/named/forward.localdomain
named-checkzone 192.168.50.67 /var/named/backward.localdomain
systemctl start named
systemctl enable named 
systemctl status named


```


```
sudo dnf install -y xorg-x11-server-utils
```

```
nslookup rac1
```

```
cat > /etc/resolv.conf <<EOF
search localdomain
nameserver 192.168.50.67
EOF

```


```
nslookup rac1
```

```
fdisk /dev/sdc
```

```
blkid /dev/sdc1
blkid /dev/sdc2
blkid /dev/sdc3

```

```
vim /etc/udev/rules.d/99-asm-disks.rules

KERNEL=="sd*", SUBSYSTEM=="block", ENV{DEVTYPE}=="partition", ENV{ID_PART_ENTRY_UUID}=="a6479004-01", SYMLINK+="oracleasm/CRS", OWNER="grid", GROUP="asmadmin", MODE="0660"

KERNEL=="sd*", SUBSYSTEM=="block", ENV{DEVTYPE}=="partition", ENV{ID_PART_ENTRY_UUID}=="a6479004-02", SYMLINK+="oracleasm/DATA", OWNER="grid", GROUP="asmadmin", MODE="0660"

KERNEL=="sd*", SUBSYSTEM=="block", ENV{DEVTYPE}=="partition", ENV{ID_PART_ENTRY_UUID}=="a6479004-03", SYMLINK+="oracleasm/FRA", OWNER="grid", GROUP="asmadmin", MODE="0660"

```

```
partx -u /dev/sdc1
partx -u /dev/sdc2
partx -u /dev/sdc3

udevadm control --reload-rules && udevadm trigger --action=add

```

```
unzip LINUX.X64_193000_grid_home.zip -d $ORACLE_HOME
```

```
cd $ORACLE_HOME
mv $ORACLE_HOME/OPatch/ $ORACLE_HOME/Opatch_BKP
unzip /u01/p6880880_210000_Linux-x86-64.zip -d $ORACLE_HOME

```

```
 $ORACLE_HOME/OPatch/opatch version
```

```
cd /u01
unzip p33803476_190000_Linux-x86-64.zip -d /u01

```

```
timedatectl
```

```
source /home/grid/.grid_env
export ORACLE_BASE=/tmp
asmcmd afd_label CRS /dev/sdc1 --init
asmcmd afd_lslbl /dev/sdc1

```

```
export DISPLAY=192.168.242.59:0.0
xhost +
export CV_ASSUME_DISTID=OEL7.9
./gridSetup.sh -applyPSU /u01/33803476/

```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

```
isi
```

