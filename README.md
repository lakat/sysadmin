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
