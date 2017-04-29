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
   firewall itself. An array of dictionaries:
     - `src`: the source port
     - `dst`: the destination port
     - `proto`: `tcp` or `udp`
   * `firewall_lan_ip`: the IP address of the LAN interface
   * `firewall_lan_width`: bitmask specifier for the LAN IP
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
     - `password`: The password for the user
     - `login`: The username to be used
   * TODO:
     - Rename this role to administrators
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
       * `http`: True if we want http traffic to be served
       * `https`: True if we want to serve https traffic
       * `ssl`: A dictionary for ssl config
         - `cert`: Path to a certificate file
         - `privkey`: Path to the private key
  * TODO:
    - default index.html.j2 does not render any parameters, this might be
      reduced to a single copy.
    - ssl might be not only a path, but a content, so it does not need to
      depend on anyone copying/setting contents of that file.
    - Is there a way to avoid using a handler?
    - the http section for each site seems to be wrong as there is no root
      specified, so if someone wanted to werve something over http it is not
      clear how that would work.
