Docker Based Server Setup
+++++++++++++++++++++++++++++++
This document describes the steps to create a Docker based server
environment on a Rocky Linux 9 server.

Some initial housekeeping
=================================

::

    sudo dnf install vim -y
    sudo visudo
        > Defaults    env_reset,timestamp_timeout=30
    sudo timedatectl set-timezone America/New_York
    sudo dnf install -y chrony
    sudo dnf -y update > dnf-update-[date].txt
    sudo shutdown -r now

Set up firewall
=================================
see :ref:`firewall-access`

Install and start docker
=================================
see :ref:`install-docker`

Create application user and set them up for docker
======================================================

::

    sudo adduser -u 5678 appuser
    sudo vi /home/appuser/.bashrc
        > export HISTTIMEFORMAT="%Y-%m-%d %H:%M "
    (appuser) mkdir .ssh
    (appuser) touch .ssh/authorized_keys
    (appuser) chmod 600 .ssh/authorized_keys
    sudo usermod -aG docker appuser

Set up rsnapshot backups
=================================
Create the following files on the server, in /home/appuser/rsnapshot. 

Note the backups for /etc, /root, /home, and /usr/local are stored in
directories annotated with -host. This allows the backup of the host without
overwriting the container folders of the same name.

In this example, the backups are stored at ``/mnt/backup/snapshots``. Be sure to
chmod this directory to 700 so that only root can read the backup data.

``docker-compose.yml``

.. literalinclude:: examples/rsnapshot/docker-compose.yml

``config/crontabs/root``

.. literalinclude:: examples/rsnapshot/config/crontabs/root

``config/rsnapshot.conf``

.. literalinclude:: examples/rsnapshot/config/rsnapshot.conf

``custom-init/rsnapshot-init``

.. literalinclude:: examples/rsnapshot/custom-init/rsnapshot-init


Set up caddy
===============================
Create the following files on the server, in /home/appuser/caddy

Note in ``Caddyfile``, logging can't use ``{common_log}`` because of the reverse
proxy setup with Cloudflare. Instead, a custom log format is used that checks
for the ``X-Forwarded-For`` header first. (see
https://github.com/caddyserver/transform-encoder,
https://caddyserver.com/docs/caddyfile/matchers, and
https://developers.cloudflare.com/support/troubleshooting/restoring-visitor-ips/restoring-original-visitor-ips/)

Note in ``Dockerfile``, caddy must be built with caddy-dns/cloudflare to be able
to update Cloudflare DNS records for automatic https certificate management, and
with WeidiDeng/caddy-cloudflare-ip to periodically update the list of Cloudflare
IPs for correct client IP logging behind the Cloudflare reverse proxy. (see
https://github.com/caddy-dns/cloudflare,
https://github.com/WeidiDeng/caddy-cloudflare-ip,
https://roelofjanelsinga.com/articles/using-caddy-ssl-with-cloudflare/, and
https://caddy.community/t/trusted-proxies-with-cloudflare-my-solution/16124/6 )

Note in ``docker-compose.yml`` the caddy-backend-network is shared with the
webapp containers to allow caddy to reverse proxy for the webapps.

Note in ``Caddyfile``, the reverse_proxy port must match the webapp port.

``.env``

::

    APP_VER=[version number]
    CF_API_TOKEN=[cloudflare api token]

``docker-compose.yml``

.. literalinclude:: examples/caddy/docker-compose.yml

``Dockerfile``

.. literalinclude:: examples/caddy/Dockerfile

``config/Caddyfile``

.. literalinclude:: examples/caddy/config/Caddyfile

Set up web site 
===============================
Create the following files on the server, in /home/appuser/[website]

In this example, static website files are stored in ``/home/appuser/webapp/html``

template files can be stored in ``/home/appuser/webapp/templates``. See https://hub.docker.com/_/nginx for details on using nginx with templates.

``docker-compose.yml``

.. literalinclude:: examples/webapp/docker-compose.yml

``nginx.conf``

.. literalinclude:: examples/webapp/nginx.conf

``html/index.html``

.. literalinclude:: examples/webapp/html/index.html
