# OpenVPN role for Ansible
Ansible role to install and configure OpenVPN Server.

This role will setup you an OpenVPN Server to access your cloud network (VPC) at no cost.
Just run this role agains empty Ubuntu Bionic instance, and you will have secure OpenVPN server
that does NAT. All configured clients will have .ovpn profile file, that is suitable for most
OpenVPN clients.

This role does not cover all aspects of VPN configuration, rather it was designed just to quickly
provide a working VPN server with reasonable level of security.

# Certificate Authority
This role will generate self-signed CA, and will use it to issue client certificates. Everything
is idempotent, so it is safe to run it again - nothing will be overwritten.

If your CA is compromised, just remove OpenVPN instance and re-create it using this role.

# Example usage
Following configuration is just enough, to create VPN server with one client - `johndoe`. Users profile will be saved
in `/etc/openvpn/client/johndoe.ovpn`, and in users home directory /home/johndoe, if such user exists. It will make easier
for the user to fetch his/her profile using SCP. Following example assumes that 172.30.0.0/16 is the network used in
your VPC.
```
openvpn_clients:
  - johndoe

openvpn_routes:
  - "172.30.0.0 255.255.0.0"

openvpn_nat_networks:
  - "172.30.0.0/16"

openvpn_server_network: "172.30.128.0 255.255.255.0"
```

# Full list of configurable options
```
---

# You probably should override following values. This values will go to CA.
openvpn_ca_organization_name: "Private Cloud"
openvpn_ca_organization_unitname: "Infrastructure"
openvpn_ca_locality: "Berlin"
openvpn_ca_state: "Berlin"
openvpn_ca_country: "DE"

# List of names of clients for which .ovpn profiles will be generated.
# If there is a system user with same name, profile will be copied to users
# home directory.
openvpn_clients:
  - client1

# Address of OpenVPN server that will be configured in .ovpn profile.
# If there is no DNS entry, it can be changed to ip address:
# openvpn_server_address: "{{ ansible_default_ipv4.address }}"
openvpn_server_address: "{{ inventory_hostname }}"

# This is internal identifier for OpenSSL, does not used in CA itself.
# Does not really need to be changed.
openvpn_ca_name: "OpenVPN"

# These are reasonable defaults, not likely to be changed.
openvpn_server_common_name: "{{ inventory_hostname }}"
openvpn_ca_directory: "/etc/ssl/openvpn"
openvpn_config_directory: "/etc/openvpn"

# Certificate lifetime and key length
openvpn_ca_lifetime: 3650
openvpn_client_cert_lifetime: 365
openvpn_ca_crl_lifetime: 30
openvpn_ca_bits: 4096

# These routes are going to be pushed to client.
openvpn_routes:
  - "10.0.0.0 255.0.0.0"
  - "172.16.0.0 255.240.0.0"
  - "192.168.0.0 255.255.0.0"

# This is the list of destinations, for which NAT should be configured.
# Very likely to reflect networks from openvpn_routes list.
openvpn_nat_networks:
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16

# This is the network, from which clients will get addresses.
openvpn_server_network: "10.255.255.0 255.255.255.0"

# VPN transport protocol and port number
openvpn_server_transport: udp
openvpn_server_port: 1194
```