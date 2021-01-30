# Usage

This is more a dump of notes than instructions.

```
make bin-x86_64-efi/ipxe.efi  bin/undionly.kpxe -j$(nproc) EMBED=scriptlet.ipxe
```

Tateru iPXE uses DHCP option code 129 instead of requiring looking at the
agent and determining the filename accordingly. This makes it easy to use
on more conventional DHCP servers.

Example ISC DHCP Config:

```
option domain-name "example.org";
option domain-name-servers 8.8.8.8;

default-lease-time 600;
max-lease-time 7200;

authoritative;

option client-arch code 93 = unsigned integer 16;
option tateru-ipxe-cfg code 129 = string;

subnet 172.16.31.0 netmask 255.255.255.0 {
  range 172.16.31.100 172.16.31.150;
  option routers 172.16.31.1;
  option tateru-ipxe-cfg "http://172.16.31.90/tateru/ipxe.cfg";

  if option client-arch != 00:00 {
    filename "ipxe.efi";
  } else {
    filename "undionly.kpxe";
  }
}
```
