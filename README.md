# elastix-mt-gui - Elastix MT GUI

![ScrennShot](https://raw.githubusercontent.com/lordbasex/elastix-mt-gui/master/logo/elastixmtgui.png)


This code is distributed under the GNU LGPL v2.0 license.


## Introduction


## Installation

Install the git package and follow the instructions on a CentOS 6.


```bash

#Disable Selinux
sed -i 's/enforcing/disabled/g' /etc/selinux/config

#Activating the interface eth0
sed -i "s/^\(ONBOOT=\).*$/\1yes/g" /etc/sysconfig/network-scripts/ifcfg-eth0

#disable iptables
service iptables save
service iptables stop
chkconfig iptables off

#Yum Update
wget https://raw.githubusercontent.com/lordbasex/elastix-mt-gui/master/elastix-framework/additionals/etc/yum.repos.d/elastix.repo -O /etc/yum.repos.d/elastix.repo
yum -y update
yum -y install vim wget

cd /usr/src

#System packages
yum -y install system-config-date system-config-firewall-base system-config-keyboard system-config-language system-config-network-tui system-config-users
#Packages for this implementation.
yum -y install dialog vim mc screen nmap wget mlocate mailx
#Packages for Development
yum -y groupinstall "Development Tools" 
yum -y install gcc gcc-c++ make openssl openssl-devel newt-devel ncurses-devel autoconf automake libpcap-devel
yum -y install rpm-build redhat-rpm-config rpmdevtools
#Packages for web server.
yum -y groupinstall "Web Server"
yum -y install mod_ssl openssl
#Packages to the database.
yum -y install mysql-server mysql-connector-odbc
#Packages for php
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm -O /usr/src/epel-release-6-8.noarch.rpm
rpm -ivh /usr/src/epel-release-6-8.noarch.rpm
yum -y install php-mcrypt
yum -y install php php-cli php-common php-devel php-gd php-imap php-mbstring  php-mysql php-pdo php-pear php-pear-DB php-process php-soap php-xml
#Packages for perl
yum -y install perl-Archive-Tar perl-Archive-Zip perl-CGI perl-Convert-BinHex perl-Crypt-OpenSSL-Bignum perl-Crypt-OpenSSL-RSA perl-Date-Manip perl-Digest-HMAC perl-Digest-SHA perl-Encode-Detect perl-HTML-Parser perl-HTML-TokeParser-Simple perl-HTTP-Response-Encoding perl-IO-Multiplex perl-IO-Socket-INET6 perl-IO-Socket-SSL perl-IO-stringy perl-MIME-tools perl-Mail-DKIM perl-Mail-IMAPClient perl-Net-IP perl-Net-Server perl-Net-Telnet perl-NetAddr-IP perl-String-CRC32 perl-URI perl-Unix-Syslog perl-WWW-Mechanize perl-XML-Parser  perl-suidperl

#Git 2.0.4
yum -y remove git
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum -y install  gcc perl-ExtUtils-MakeMaker

cd /usr/src
wget https://www.kernel.org/pub/software/scm/git/git-2.0.4.tar.gz
tar xzf git-2.0.4.tar.gz
cd git-2.0.4
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc

#Install Asterisk
yum -y install asterisk
/usr/sbin/adduser asterisk
/usr/sbin/usermod -c "Asterisk VoIP PBX" -g asterisk -s /bin/bash -d /var/lib/asterisk asterisk


#Cloning repository
cd /usr/src
git clone https://github.com/lordbasex/elastix-mt-gui.git

#Creating RPM for installation
rpmdev-setuptree
rm -fr /root/rpmbuild/
ln -s /usr/src/elastix-mt-gui/rpmbuild /root/
rpmbuild -ba /root/rpmbuild/SPECS/elastix-agenda.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-email_admin.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-fax.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-firstboot.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-framework.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-pbx.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-portknock.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-reports.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-security.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix.spec
rpmbuild -ba /root/rpmbuild/SPECS/elastix-system.spec

#Start Services
chkconfig --level 345 ntpd on
chkconfig --level 345 mysqld on
chkconfig --level 345 httpd on

/etc/init.d/htpd start
/etc/init.d/mysqld start
/etc/init.d/httpd start

#Reconfigure Mysql
echo "SET PASSWORD FOR 'root'@'localhost' = PASSWORD('eLaStIx.2oo7');" | mysql -u root

## INSTALL RPM ##
#Install elastix-firstboot
rpm -i --nodeps /root/rpmbuild/RPMS/noarch/elastix-firstboot-3.0.0-6.noarch.rpm

#Install elastix-framework 
rpm -i /root/rpmbuild/RPMS/noarch/elastix-framework-3.0.0-12.noarch.rpm

#Install elastix-agenda
rpm -i /root/rpmbuild/RPMS/noarch/elastix-agenda-3.0.0-8.noarch.rpm

#Install elastix-email_admin
yum -y install cyrus-imapd mailman spamassassin
rpm -i /root/rpmbuild/RPMS/noarch/elastix-email_admin-3.0.0-7.noarch.rpm

#Install elastix-fax
yum -y install ghostscript hylafax iaxmodem
rpm -i /root/rpmbuild/RPMS/noarch/elastix-fax-3.0.0-8.noarch.rpm

#Install elastix-system
yum -y install dahdi dhcp
rpm -i /root/rpmbuild/RPMS/noarch/elastix-system-3.0.0-8.noarch.rpm

#Install elastix-pbx
yum -y install asterisk-chan-allogsm asterisk-chan-extra asterisk-dahdi asterisk-festival asterisk-odbc asterisk-voicemail asterisk-perl festival kamailio kamailio-mysql kamailio-presence kamailio-unixodbc kamailio-utils tftp-server vsftpd uuid-perl
rpm -i /root/rpmbuild/RPMS/noarch/elastix-pbx-3.0.0-13.noarch.rpm
source /etc/elastix.conf
echo "GRANT ALL PRIVILEGES ON elxpbx.* TO 'asteriskuser'@'localhost' IDENTIFIED BY '${mysqlrootpwd}' WITH GRANT OPTION;" | mysql -u root -p${mysqlrootpwd}
echo "GRANT ALL PRIVILEGES ON asteriskcdrdb.* TO 'asteriskuser'@'localhost' IDENTIFIED BY '${mysqlrootpwd}' WITH GRANT OPTION;" | mysql -u root -p${mysqlrootpwd}
echo "GRANT ALL PRIVILEGES ON kamailio.* TO 'asteriskuser'@'localhost' IDENTIFIED BY '${mysqlrootpwd}' WITH GRANT OPTION;" | mysql -u root -p${mysqlrootpwd}


#Install elastix-reports
rpm -i /root/rpmbuild/RPMS/noarch/elastix-reports-3.0.0-9.noarch.rpm

#Install elastix-portknock
rpm -i /root/rpmbuild/RPMS/x86_64/elastix-portknock-0.0.1-0.x86_64.rpm

#Install elastix-security
rpm -i /root/rpmbuild/RPMS/noarch/elastix-security-3.0.0-5.noarch.rpm
```
