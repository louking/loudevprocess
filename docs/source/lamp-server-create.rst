LAMP Server Create
+++++++++++++++++++++++++++++++

.. warning::

    This is obsolete as we're currently using docker containers, which contain
    the LAMP server.

This uses digitalocean for server and volume creation, but most of this
could be used for any LAMP server.

Create Droplet
==============

-  log into digitalocean.com
-  Create > Droplets <name> 8 GB / 80 GB Disk / 4 Intel vCPUs / Rocky Linux / NYC1 IP

Some Upkeep [root]
==================
::

    dnf install -y vim
    dnf -y update (see /var/log/dnf.log for all updates)
    shutdown -r now


Additional repos
----------------
::

    dnf install -y epel-release

sudo user
=========

Create non-root user and give sudo access [root]
------------------------------------------------
::

    adduser <user>; passwd <user>
    mkdir /home/<user>/.ssh; cp .ssh/authorized_keys /home/<user>/.ssh
    chown -R <user>:<user> /home/<user>/.ssh
    gpasswd -a <user> wheel

Set sudo timeout (minutes)
--------------------------
::

    sudo visudo # replace 'Defaults env_reset' with following
        Defaults env_reset,timestamp_timeout=30

history should display date/time
================================
::

    export HISTTIMEFORMAT="%Y-%m-%d %H:%M " # add this to ~/.bashrc, /root/.bashrc

Set local time and keep in sync
===============================
::

    sudo timedatectl set-timezone America/New_York
    sudo dnf install -y chrony # installs and starts ntp client/servers

Turn off cron information to /var/log/messages
==============================================
::

    sudo vim /etc/rc.d/rc.local # add /usr/bin/systemd-analyze set-log-level notice
    sudo /usr/bin/systemd-analyze set-log-level notice
    sudo chmod +x /etc/rc.d/rc.local

Set /usr/local/lib in library path
===================================
See https://serverfault.com/a/372998

::

    sudo vim /etc/ld.so.conf.d/usrlocal.conf
    > /usr/local/lib
    sudo ldconfig -v

LAMP and Security Stack
=======================

Set up Apache
------------------

See https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-7

::

    sudo dnf install -y httpd
    sudo systemctl start httpd.service
    sudo systemctl enable httpd.service

.. _firewall-access:

Firewall: allow certain access
-------------------------------

::

    sudo systemctl start firewalld
    sudo firewall-cmd --permanent --add-service=ssh
    sudo firewall-cmd --permanent --add-service=http
    sudo firewall-cmd --permanent --add-service=https
    sudo firewall-cmd --permanent --add-service=smtp # is this needed?
    sudo firewall-cmd --reload
    sudo systemctl enable firewalld

Set up HTTPS / certbot
------------------------
requires epel-release

::

    sudo dnf install -y certbot python-certbot-apache

Set up git
--------------

::

    sudo dnf install -y git-all

Create a2ensite, a2dissite
--------------------------
See http://www.tecmint.com/apache-virtual-hosting-in-centos/
::

   sudo vim /usr/local/bin/a2ensite
        #!/bin/bash
        if test -d /etc/httpd/sites-available && test -d /etc/httpd/sites-enabled  ; then
        echo "-----------------------------------------------"
        else
        mkdir /etc/httpd/sites-available
        mkdir /etc/httpd/sites-enabled
        fi

        avail=/etc/httpd/sites-available/$1.conf
        enabled=/etc/httpd/sites-enabled/
        site=`ls /etc/httpd/sites-available/`

        if [ "$#" != "1" ]; then
                        echo "Use script: a2ensite virtual_site"
                        echo -e "\nAvailable virtual hosts:\n$site"
                        exit 0
        else

        if test -e $avail; then
        sudo ln -s $avail $enabled
        else

        echo -e "$avail virtual host does not exist! Please create one!\n$site"
        exit 0
        fi
        if test -e $enabled/$1.conf; then

        echo "Success!! Now restart Apache server: sudo systemctl restart httpd"
        else
        echo  -e "Virtual host $avail does not exist!\nPlease see available virtual hosts:\n$site"
        exit 0
        fi
        fi

    sudo chmod +x /usr/local/bin/a2ensite

    sudo vim /usr/local/bin/a2dissite
        #!/bin/bash
        avail=/etc/httpd/sites-enabled/$1.conf
        enabled=/etc/httpd/sites-enabled
        site=`ls /etc/httpd/sites-enabled/`

        if [ "$#" != "1" ]; then
                        echo "Use script: a2dissite virtual_site"
                        echo -e "\nAvailable virtual hosts: \n$site"
                        exit 0
        else

        if test -e $avail; then
        sudo rm  $avail
        else
        echo -e "$avail virtual host does not exist! Exiting!"
        exit 0
        fi

        if test -e $enabled/$1.conf; then
        echo "Error!! Could not remove $avail virtual host!"
        else
        echo  -e "Success! $avail has been removed!\nPlease restart Apache: sudo systemctl restart httpd"
        exit 0
        fi
        fi

    sudo mkdir /etc/httpd/sites-available /etc/httpd/sites-enabled
    sudo vim /etc/httpd/conf/httpd.conf
       353a354
       > IncludeOptional sites-enabled/*.conf


Set up mysql
------------------

    sudo dnf -y install mariadb-server mariadb
    sudo systemctl start mariadb
    sudo mysql_secure_installation
    sudo systemctl enable mariadb.service

Set up PHP
-----------------

    sudo dnf -y install php php-mysql
    sudo systemctl restart httpd.service

install additional PHP versions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See https://stackoverflow.com/a/50079574/799921 and
https://blog.remirepo.net/post/2016/04/16/My-PHP-Workstation
::

    # this is done once
    sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -y
    sudo dnf install http://rpms.remirepo.net/enterprise/remi-release-7.rpm -y
    sudo dnf install dnf-utils -y

    # this is done for each new version
    sudo dnf install php74y -y
    sudo dnf install php74-php-fpm -y
    sudo vim /etc/opt/remi/php74/php-fpm.d/www.conf
        listen = 127.0.0.1:9074 # 9000 + 74 for the php version
    sudo dnf install php74-php-mysqlnd -y
    sudo dnf install php74-php-xml -y
    sudo dnf install php74-php-gd -y

    # optimize memory usage
    sudo vim /etc/opt/remi/php74/php.ini
        409c409
        < memory_limit = 128M
        ---
        > memory_limit = 256M
        846c846
        < upload_max_filesize = 2M
        ---
        > upload_max_filesize = 4M
    sudo vim /etc/opt/remi/php74/php-fpm.d/www.conf
        104c104
        < pm = dynamic
        ---
        > pm = ondemand
        115c115
        < pm.max_children = 50
        ---
        > pm.max_children = 25
        141c141
        < ;pm.max_requests = 500
        ---
        > pm.max_requests = 500

    sudo systemctl enable php74-php-fpm
    sudo systemctl start php74-php-fpm

    # this is done for each vhost
    sudo vim /etc/httpd/sites-available/www.steeplechasers.org.conf # match the listen port above
        24c24
        <     SetHandler "proxy:fcgi://127.0.0.1:9073"
        ---
        >     SetHandler "proxy:fcgi://127.0.0.1:9074"
    sudo apachectl restart

.. _install-docker:

Install and start Docker
============================
See https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-rocky-linux-9#step-1-installing-docker

::

    sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo dnf install -y docker-ce docker-ce-cli containerd.io
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo systemctl status docker

Set up VHOST
============

Backups
=======

Create backup volume
--------------------

-  [DO console] Volumes > Add Volume > 200 GB

::

    sudo mkfs.ext4 -F /dev/disk/by-id/<volumename>
    sudo mkdir -p /mnt/backup
    sudo mount -o discard,defaults /dev/disk/by-id/<volumename> /mnt/backup
    echo /dev/disk/by-id/<volumename> /mnt/backup ext4 defaults,nofail,discard 0 0 \| sudo tee -a /etc/fstab

Set up backup
-------------

See https://www.digitalocean.com/community/tutorials/how-to-install-rsnapshot-on-ubuntu-12-04
::

    sudo dnf install -y rsnapshot
    sudo dnf install -y rsnapshot
    sudo vim /etc/rsnapshot.conf
        23c23
        < snapshot_root /.snapshots/
        ---
        > snapshot_root /mnt/backup/snapshots/
        40c40
        < #cmd_cp /usr/bin/cp
        ---
        > cmd_cp /usr/bin/cp
        63c63
        < #cmd_du /usr/bin/du
        ---
        > cmd_du /usr/bin/du
        67c67
        < #cmd_rsnapshot_diff /usr/local/bin/rsnapshot-diff
        ---
        > cmd_rsnapshot_diff /usr/bin/rsnapshot-diff
        93,95c93,95
        < retain alpha 6
        < retain beta 7
        < retain gamma 4
        ---
        > #retain alpha 6
        > #retain beta 7
        > #retain gamma 4
        96a97,100
        > retain hourly 6
        > retain daily 7
        > retain weekly 4
        > retain monthly 3
        120c124
        < #logfile /var/log/rsnapshot
        ---
        > logfile /var/log/rsnapshot
        229c233,234
        < #backup /var/log/rsnapshot localhost/
        ---
        > backup /var/log/rsnapshot localhost/
        > backup /var/www localhost/
    sudo rsnapshot configtest
    sudo rsnapshot -t hourly
    sudo rsnapshot hourly
    sudo vim /etc/cron.d/rsnapshot
    -  These settings will run add a snapshot to the "hourly" directory
          within our "/backup/" directory every four hours, add a daily
          snapshot everyday at 3:30 am, add a weekly snapshot every
          Monday at 3:00 am, and add a monthly snapshot on the first of
          every month at 2:30 am.
    -  It is important to stagger your backups and run larger backup
          intervals first. This means running the monthly backup first
          and progressing to shorter intervals from there in order, as
          we've done in this tutorial. This is necessary so that the
          program does not get caught up trying to do multiple backups at
          the same time, which can cause problems.
        0 \*/4 \* \* \* root /usr/bin/rsnapshot hourly
        30 3 \* \* \* root /usr/bin/rsnapshot daily
        0 3 \* \* 1 root /usr/bin/rsnapshot weekly
        30 2 1 \* \* root /usr/bin/rsnapshot monthly

Resize backup volume (only if necessary)
----------------------------------------

See https://www.digitalocean.com/community/tutorials/how-to-increase-the-size-of-a-digitalocean-block-storage-volume

-  droplet must be switched off to resize an attached volume

::

    sudo shutdown -h now

-  [DO console] Droplet loutility-server-digitalocean > Volumes > backup > More > Resize Volume > 40GB
-  [DO console] Switch On droplet
-  determine name of volume

::

    ls -l /dev/disk/by-id

    total 0
    lrwxrwxrwx 1 root root 9 Sep 21 05:47 scsi-0DO_Volume_backup -> ../../sdc
    lrwxrwxrwx 1 root root 9 Sep 21 05:44 scsi-0DO_Volume_loutility-server-backup -> ../../sdb
    lrwxrwxrwx 1 root root 9 Sep 21 05:44 scsi-0DO_Volume_loutility-server-swap -> ../../sda

-  determine filesystem type

::

    sudo lsblk --fs /dev/disk/by-id/scsi-0DO_Volume_backup

    NAME FSTYPE LABEL UUID MOUNTPOINT
    sdc ext4 0b21852e-dee8-4828-97b1-92e66d877b2d /mnt/backup

-  resize unpartitioned ext4 volume

::

    sudo resize2fs /dev/disk/by-id/scsi-0DO_Volume_backup

Set up swap volume
==================

See https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-swap-adding.html

-  [DO console] Volumes > Add Volume > 10 GB / swapspace

::

    sudo mkswap /dev/disk/by-id/<volumename>
    sudo vim /etc/fstab # add following line
        /dev/disk/by-id/scsi-0DO_Volume_swapspace swap swap defaults 0 0
    sudo swapon -va

Security
========

Set up server level security
----------------------------

-  https://www.digitalocean.com/community/tutorials/an-introduction-to-securing-your-linux-vps

   -  https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-6

   -  http://stuffphilwrites.com/2013/03/permanently-ban-repeat-offenders-fail2ban/

::

        sudo dnf install -y fail2ban
        sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
        sudo vim /etc/fail2ban/jail.local
        -  set ignoreip to your personal ip address
        -  set destemail to your personal email address
        -  set enabled to true (for desired jails)
        -  set bantime to 3600 (globally)
        sudo systemctl enable fail2ban
        sudo systemctl start fail2ban

.
   -  https://www.digitalocean.com/community/tutorials/how-to-install-aide-on-a-digitalocean-vps
