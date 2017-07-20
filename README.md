# dmpdockerfiles

from ubuntu:16.04
add sources.list sources.list
run mv sources.list /etc/apt/ && apt-get update && apt-get install maas-rack-controller net-tools -y ; \
(cd /lib/systemd/system/sysinit.target.wants/; ls ; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*; \
rm -f /etc/systemd/system/*.wants/*; \
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*; \
rm -f /lib/systemd/system/anaconda.target.wants/*; \
/bin/systemctl enable maas-rackd
cmd ["/sbin/init"]
expose 5248
expose 69
expose 123
expose 53


#!/bin/bash

/usr/sbin/rsyslogd

export LOGFILE=/var/log/maas/rackd.log

rm -f /var/lib/maas/dhcpd.sock
rm -f /var/lib/maas/dhcpd.conf
rm -f /var/lib/maas/dhcpd6.conf

exec /usr/bin/authbind --deep /usr/bin/twistd3 --nodaemon --pidfile=  --logger=provisioningserver.logger.EventLogger maas-rackd 2>&1 | tee -a $LOGFILE


# deb cdrom:[Ubuntu-Server 16.04.2 LTS _Xenial Xerus_ - Release amd64 (20170215.8)]/ xenial main restricted

#deb cdrom:[Ubuntu-Server 16.04.2 LTS _Xenial Xerus_ - Release amd64 (20170215.8)]/ xenial main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://pl.archive.ubuntu.com/ubuntu/ xenial main restricted
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://pl.archive.ubuntu.com/ubuntu/ xenial-updates main restricted
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://pl.archive.ubuntu.com/ubuntu/ xenial universe
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial universe
deb http://pl.archive.ubuntu.com/ubuntu/ xenial-updates universe
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://pl.archive.ubuntu.com/ubuntu/ xenial multiverse
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial multiverse
deb http://pl.archive.ubuntu.com/ubuntu/ xenial-updates multiverse
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://pl.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src http://pl.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu xenial partner
# deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://security.ubuntu.com/ubuntu xenial-security main restricted
# deb-src http://security.ubuntu.com/ubuntu xenial-security main restricted
deb http://security.ubuntu.com/ubuntu xenial-security universe
# deb-src http://security.ubuntu.com/ubuntu xenial-security universe
deb http://security.ubuntu.com/ubuntu xenial-security multiverse
# deb-src http://security.ubuntu.com/ubuntu xenial-security multiverse



from ubuntu:16.04
add sources.list sources.list
add prep.sh /usr/local/sbin
add entrypoint.sh /entrypoint.sh
run mv sources.list /etc/apt/ && apt-get update && apt-get install maas-region-controller net-tools -y sudo && apt-get install maas-rack-controller -y ; \
(cd /lib/systemd/system/sysinit.target.wants/; ls ; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*; \
rm -f /etc/systemd/system/*.wants/*; \
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*; \
rm -f /lib/systemd/system/anaconda.target.wants/*; \
systemctl disable maas-rackd ; \
systemctl enable postgresql bind9 ntp maas-proxy maas-regiond ; \
echo "[Unit]\nDescription=Prepare region controller\nAfter=maas-regiond.service\nRequires=maas-regiond.service\n\n[Service]\nExecStart=/bin/sleep 40\nExecStart=/usr/local/sbin/prep.sh\nType=oneshot\n\n[Install]\nWantedBy=multi-user.target" >> /etc/systemd/system/prep.service && systemctl enable prep.service ; touch /tmp/env
env allinone=1 maas_admin_login=admin maas_admin_password=admin maas_admin_email=root@localhost external_postgres=0 external_postgres_host=0.0.0.0 external_postgres_port=0 external_postgres_username=none externel_postgres_password=none
cmd ["/entrypoint.sh"]
expose 5240



#!/bin/bash

#save docker envs in file. After we prepare container they will be removed anyway

if [ -f /tmp/env ]
then
        echo "export allinone=$allinone" >> /tmp/env
        echo "export maas_admin_login=$maas_admin_login" >> /tmp/env
        echo "export maas_admin_password=$maas_admin_password" >> /tmp/env
        echo "export maas_admin_email=$maas_admin_email" >> /tmp/env
        echo "export external_postgres=$external_postgres" >> /tmp/env
fi

exec /sbin/init


#!/bin/bash

source /tmp/env

if [ $allinone -ne 0 ]
then
        systemctl enable maas-rackd
fi

maas-region createadmin --username "$maas_admin_login" --password "$maas_admin_password" --email "$maas_admin_email"

#if [ $external_postgres -ne 0 ]
#then

#fi

#rm -f /tmp/env

#systemctl mask prep.service
