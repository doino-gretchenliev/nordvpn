[![logo](http://fc05.deviantart.net/fs70/i/2013/010/6/5/openvpn_icon_by_archeinre-d5r1nls.png)](https://openvpn.net/)

# OpenVPN

OpenVPN client docker container

# What is OpenVPN?

OpenVPN is an open-source software application that implements virtual private
network (VPN) techniques for creating secure point-to-point or site-to-site
connections in routed or bridged configurations and remote access facilities.
It uses a custom security protocol that utilizes SSL/TLS for key exchange. It is
capable of traversing network address translators (NATs) and firewalls.

# How to use this image

This OpenVPN container was designed to be started first to provide a connection
to other containers (using `--net=container:openvpn`, see below).

**NOTE**: More than the basic privileges are needed for OpenVPN. With docker 1.2
or newer you can use the `--cap-add=NET_ADMIN` and `--device /dev/net/tun`
options. Earlier versions, or with fig, and you'll have to run it in privileged
mode.

## Hosting an OpenVPN client instance

    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -d dperson/openvpn \
                -v "vpn.server.name;username;password"
    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker restart openvpn

Once it's up other containers can be started using it's network connection:

    sudo docker run --net=container:openvpn -d some/docker-container

## Configuration

    sudo docker run -it --rm dperson/openvpn -h

    Usage: openvpn.sh [-opt] [command]
    Options (fields in '[]' are optional, '<>' are required):
        -h          This help
        -f          Set firewall rules so that only the VPN and DNS are allowed to
                    send traffic to the internet (IE if VPN is down it's offline)
        -t ""       Configure timezone
                    possible arg: "[timezone]" - zoneinfo timezone for container
        -v "<server;user;password>" Configure OpenVPN
                    required arg: "<server>;<user>;<password>"
                    <server> to connect to
                    <user> to authenticate as
                    <password> to authenticate with

    The 'command' (if provided and valid) will be run instead of openvpn

ENVIROMENT VARIABLES (only available with `docker run`)

 * `TZ` - As above, set a zoneinfo timezone, IE `EST5EDT`
 * `VPN` - As above, setup a VPN connection

## Examples

Any of the commands can be run at creation with `docker run` or later with
`docker exec openvpn.sh` (as of version 1.3 of docker).

### Setting the Timezone

    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -d dperson/openvpn -t EST5EDT \
                -v "vpn.server.name;username;password"

OR using `environment variables`

    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -e TZ=EST5EDT -d dperson/openvpn \
                -v "vpn.server.name;username;password"

Will get you the same settings as:

    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -d dperson/openvpn \
                -v "vpn.server.name;username;password"
    sudo docker exec openvpn openvpn.sh -t EST5EDT ls -AlF /etc/localtime
    sudo docker restart openvpn

### VPN configuration

In order to work you must provide VPN configuration and the certificate. You can
use external storage for `/vpn`:

    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -d dperson/openvpn \
                -v "vpn.server.name;username;password"
    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker restart openvpn

Or you can store it in the container:

    cat /path/to/vpn.crt | sudo docker run -i --cap-add=NET_ADMIN \
                --device /dev/net/tun --name openvpn -d dperson/openvpn \
                -v "vpn.server.name;username;password" tee /vpn/vpn-ca.crt \
                >/dev/null
    sudo docker restart openvpn

### Firewall

It's just a simple command line argument (`-f`) to turn on the firewall, and
block all outbound traffic if the VPN is down.

    sudo docker run --cap-add=NET_ADMIN --device /dev/net/tun --name openvpn \
                -v /some/path:/vpn -d dperson/openvpn -f \
                -v "vpn.server.name;username;password"
    sudo cp /path/to/vpn.crt /some/path/vpn-ca.crt
    sudo docker restart openvpn

# User Feedback

## Issues

If you have any problems with or questions about this image, please contact me
through a [GitHub issue](https://github.com/dperson/openvpn/issues).
