# sysadmin

## Ansible Roles

Ansible roles are located under [roles](ansible/roles) directory. Below is a
short description for each:

 - bootstrap: role that installs python that is required to run ansible and
   also updates the system. This role has no parameters.
 - interfaces: set your `/etc/network/interfaces` file's contents, remove
   resolvconf package, and set a static `/etc/resolv.conf` that points to
   pre-defined dns servers:
   * `interfaces_file_contents`: the contents of `/etc/network/interfaces`
   * `interfaces_dns_servers`: an array specifying dns servers to be set in
      `/etc/resolv.conf`
 - firewall: set up a firewall, using shorewall.
   * `firewall_wan_if`: name of the interface that is connected to wan.
   defaults to `eth0`.
   * `firewall_lan_if`: interface to use for lan traffic. If empty, no NAT
   will happen. Defaults to empty.
   * `firewall_dhcp_ifs`: an array that might contain either wan or lan if.
   Indicates that the specific interface has to let dhcp traffic through.
   * `firewall_enabled_protocols`: an array that lists protocols that are
   enabled from the outside world.
   * `firewall_self_forwards`: port forwardings for services running on the
   firewall itself. The service that is listening on the src port will also
   be accessible on the dst port. An array of dictionaries:
     - `src`: the source port
     - `dst`: the destination port
     - `proto`: `tcp` or `udp`
   * `firewall_lan_ip`: the IP address of the LAN interface
   * `firewall_lan_width`: bitmask specifier for the LAN IP
   * `firewall_local_dnats`: for the local network, accessing src ip will be
   served by dst ip on net.
     - `src`: The source IP, or list of IPs
     - `dst`: The destination IP
 - ips: Set up Intrusion Prevention System, in this case fail2ban. This role
   has no parameters.
 - locale: Set the system's locale to GB
 - hostname: Set the system's hostname and domain name. Please note that a
   system reboot is required for the changes to take effect. The following
   variables can be used:
   * `hostname_hostname`: the host part of the FQDN
   * `hostname_domainname`: the domain part of the FQDN
   * TODO:
     - check if the `hostname` command could be used to avoid the need for
       rebooting the system.
 - ssh: Configure sshd on the system.
   * `ssh_allowed_users`: The list of logins that can log in through ssh.
     Please note that this list is empty by default, which means that you
     will be locked out of the system with the default configuration. Make
     sure you enable desired logins.
 - certificates: Install certificates on your system, and add a dhparam file.
   For generating the dhparam file, use:
   `openssl dhparam -out dhparam.pem 4096`, for finding out why is it needed
   at all, read
   [this article](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html).
   Long story short: "The second threat is that many servers and use the same
   prime numbers for Diffie-Hellman key exchange instead of generating their
   own unique DH parameters"
   * `certificates`:
     - `certs`:
       * `private_key`:
         - `path`: The path for the private key on the system.
         - `content`: Contents of the private key.
       * `certificate`:
         - `path`: The path for the certificate on the system.
         - `content`: Contents of the certificate.
       * `delete`: When true, this certificate will be removed from the system.
     - `dhparam`:
       * `path`: Path for the DHParams file
       * `content`: Contents of the DHparams file
 - mountpoints: Establish mount points for block devices if they exist. The
   usecase is that the system has a block device with persistent data on it,
   and this role will mount that block device to a specific directory. If the
   block device does not exist, the mount point will be created and the error
   will be ignored.
   * `mountpoints`: An array of dictionaries
     - `name`: Name for the mount point
     - `src`: Block device to be used
     - `path`: Target path into which to me mounted to
     - `fstype`: File system type
     - `opts`: Mount options
  * TODO:
     - Ignoring the non-existence of a block device smells. It would be better
       to have an option within the mountpoint record to indicate that the
       non-existence of the block device on the system can be ignored. The
       mechanism to ignore missing block devices was added due to testing, so
       another possible fix would be to make the testing infrastructure more
       production-like by attaching a block device to it.
 - users: Create administrators in the system and install ssh keys for them.
   * `users_administrators`: An array of dictionaries
     - `password`: The password for the user. See
        [the section below](#howtopass) on how to generate the password.
     - `login`: The username to be used
     - `key`: ssh key to be added to ~/.ssh/authorized_keys
   * `users_admin_robots`: An array of dictionaries, each represent a user that
     can log in and run sudo without the need for a password.
     - `login`: Username to be used
     - `key`: ssh key to be added to ~/.ssh/authorized_keys
   * `users_users`: An array of dictionaries, each represent a user that
     can log in to the system with an ssh key. Please do not forget to add the
     user to the allowed_users in your ssh config should you have any.
     - `login`: Username to be used
     - `key`: ssh key to be added to ~/.ssh/authorized_keys
     - `home`: Location for the user's home
     - `shell`: Shell to be used for the user (defaults to `/bin/bash`)
 - logwatch: Add logwatch - no parameters
 - www: http server (nginx) with pluggable python applications - see pyapps
   * `www_default_root`: Default filesystem path for the default server.
   * `www_dhparam_path`: Path for dhparam file. This is needed to make sure
     SSL is set up in a secure way.
   * `www`:
     - `sites`: An array of dictionaries for each site as follows:
       * `name`: Name for the site
       * `root`: root path for the site
       * `domain`: domain name to serve
       * `redirect_http_to_https`: if this is true, a 301 (Moved Permanently)
         will be sent upon http request to redirect clients to https
       * `redirect_to_another_domain`: Specify target domain here, something
         like https://example.com, and all requests arriving to this site's
         http and https endpoint will be redirected to the target site.
       * `http`: True if we want http traffic to be served
       * `https`: True if we want to serve https traffic
       * `serve_on_http`: Serve aliases on http as well
       * `ssl`: A dictionary for ssl config
         - `cert`: Path to a certificate file
         - `privkey`: Path to the private key
       * `serve_root`: If this is true, nginx will serve the `/` point, so you
                       cannot export a pyapp/alias there.
       * `aliases`: Optional array of dictionaries for each alias you want to
                    share
         - `name`: A name to be used to identify the alias. This will be used
                   as a filename, so please use compatible string
         - `directory`: The local directory to be exported. Please note that
                        you need to take care of creating this directory.
         - `mount_point`: Where would you like to mount the directory
         - `htpasswd`: Optional - contents of the htpasswd file. Use a
                       `user:password-hash` format and generate the password
                       hash with `openssl passwd -apr1`. If not specified
                       this will be open to public.
         - `allowed_hosts`: Optional list of hosts that you want to allow
                            access. If not specified, it will be open to public.
         - `noautoindex`: Optional, by default it is False, set it to true to
                          disable autoindex
      * `rewrites`: A list of custom strings. Refer to nginx's documentation on
        the format, see rewrite section. For example having the following value:
        `^/something$ https://something.com` will result in a line:
        `rewrite ^/something$ https://something.com;` to be added to nginx
        configuration.
   * TODO:
     - default index.html.j2 does not render any parameters, this might be
       reduced to a single copy.
     - ssl might be not only a path, but a content, so it does not need to
       depend on anyone copying/setting contents of that file.
     - Is there a way to avoid using a handler?
     - the http section for each site seems to be wrong as there is no root
       specified, so if someone wanted to werve something over http it is not
       clear how that would work.
 - reboot: include this role to reboot your server
   * TODO:
     - we might want to flush handlers as a first task as after the reboot
       it might not be possible to reach the server.
 - time: set timezone of the server
   * `time_timezone`: The timezone to use
 - database: Install postgresql and libraries required for managing it later.
   This role has no parameters.
 - mail: Instal both smtp and imap server
   * `mail_certificate_path`: Path to certificate (used by IMAPS and SMTP)
   * `mail_private_key_path`: Path to private key for the above
   * `mail_virtual_mailbox_maps`: location for virtual mailbox mapping file
   * `mail_virtual_alias_maps`: location for virtual alias maps
   * `mail_aliases_path`: aliases file
   * `mail_vboxes_root`: root for virtual mailboxes
   * `mail_vhomes_root`: root for virtual homes
   * `mail_mailusers_passdb`: location where to put password database
   * `mail_mailusers_userdb`: location for user database
   * `mail_gid`: GID for mail user
   * `mail_uid`: UID for mail user
   * `mail_aliases`: A list of dictionaries for explicit aliases
     - `src`: Source
     - `dst`: Destination
   * `mail`: Dictionary for mail settings:
     - `mailname`: name for your mailing system
     - `admin`: the user who receives e-mail for system accounts
     - `domains`: list of domains to serve
     - `mailboxes`: A list of dictionaries:
       * `user`: username for that mailbox
       * `address`: e-mail address for that mailbox
       * `password`: Password for that user (use `doveadm pw -s SHA512-CRYPT`)
       * `aliases`: A list to define aliases for this mailbox
 - mailutils: Install mailutils, so people can send emails from console. This
   role has no parameters.
 - uwsgi: Install uwsgi to prepare for pyapps
   * `uwsgi_install_dir`: Installation directory for uwsgi
   * `uwsgi_root`: Root for applications
 - pyapps: Python application deployer. This creates a separate user, group,
   database, wsgi config and www mapping for your application.
   * `uwsgi_root`: Root of uwsgi
   * `pyapps`: A list of dictionaries
     - `name`: Name for the application
     - `site`: Name of the site under which you want to set it up
     - `location`: A location where you want to map the application
     - `processes`: Number of processes to run
 - postgres_db_dumper: Dump a postgres database
   * `backup_database`: Name of the database to dump
   * `backup_local_file`: Name for the local file to store the dump
 - postgres_db_restorer: Restore a database from a local dump
   * `restore_local_file`: Local path pointing to a database dump
   * `restore_database`: Database to restore to
   * `restore_user`: User on remote system who will own the database


### <a name="howtopass"></a>How to generate a password

Have python passlib installed, and then:


    python -c "from passlib.hash import sha512_crypt; import getpass; print sha512_crypt.using(rounds=5000).hash(getpass.getpass())"


### How to generate an SSL certificate request

First, generate a private key:

    openssl genrsa -out <domain.key> 2048

Then Create the CSR:

    openssl req -utf8 -new -sha256 -key <domain.key> -out <domain.csr>

On cert generation I used SHA256 - not the FULL CHAIN. Once that's done, needed
to append a cert to the chain:
 - Intermediate CA Certificate: RapidSSL RSA SHA-2 (under SHA-1 Root)

Always do a check with:
https://www.ssllabs.com/ssltest/analyze.html
