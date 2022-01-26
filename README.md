# Usage

This is more a dump of notes than instructions.

## Build instructions

1. Download the iPXE source code.
   ```
   git clone git://git.ipxe.org/ipxe.git
   ```
2. Make sure all the necessary flags and files are uncommented (by applying
   the patches). If you have cloned the ipxe repository directly into the
   tateru-ipxe one:
   ```bash
   cd ipxe
   for p in ../*.patch; do
     patch -p0 <"${p}"
   done
   ```
3. Build `ipxe.efi` and `undionly.kpxe` with:
   ```
   make bin-x86_64-efi/ipxe.efi bin/undionly.kpxe -j$(nproc) EMBED=bootstrap.ipxe
   ```


## Network boot quick start

You need three things: a DHCP server, a TFTP server and a HTTP server.

1. Place the following files in the document root of a TFTP server, do not
   keep the subfolders.
   - `bin-x86_64-efi/ipxe.efi`
   - `bin/undionly.kpxe`
2. Publish the following files on a HTTP server, for example in a `tateru`
   folder right in the document root.
   - `tateru.ipxe`
   - `tateru.png`
3. Edit the `tateru.ipxe` file. You need to update the variables at the top,
   so that they suit your setup.
4. Build the netboot archive from the [tateru-installer](https://github.com/tateru/tateru-installer)
   project (`make netboot`). Extract the `tateru-netboot.tar.gz` archive so
   that the resulting `installer` subfolder is located next to the files
   from the previous step.
5. Configure a DHCP server. Check the next section for a minimal example.


## Example ISC DHCP Config

Tateru iPXE uses DHCP option code 129 instead of requiring looking at the
agent and determining the filename accordingly. This makes it easy to use
on more conventional DHCP servers. Point the option 129 (`tateru-ipxe-cfg`
in the snippet below) to the `ipxe.cfg` file you have copied to the http
server.

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
  option tateru-ipxe-cfg "http://172.16.31.90/tateru/tateru.ipxe";

  next-server 172.16.31.90;
  if option client-arch != 00:00 {
    filename "ipxe.efi";
  } else {
    filename "undionly.kpxe";
  }
}
```
