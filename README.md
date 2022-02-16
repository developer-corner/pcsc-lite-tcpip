# pcsc-lite-tcpip
TCP/IP patch for PCSCLite project - pcscd listens on UNIX domain socket as well as TCP/IP socket

The tiny project aims to add support for TCP/IP networking to the PCSCLite project. Two patches will be developed:
* TCP/IP-only patch - done!
* TCP/IP with TLSv1.3 support patch (OpenSSL) - TODO!

The first patch introduces three new environment variables for pcscd or libpcsclite, respectively:
* PCSC_HOST_OR_IP has to contain either a host name or an IPv4/IPv6 address. The pcscd listens on this address (the UNIX domain socket is handled as before!)
* PCSC_TCP_PORT has to define the TCP port
* PCSC_TCP_KEEPALIVE (evaluated only by pcscd, the server side!): This optional environment variable has the format "<idle seconds>,<seconds between probes>,<number of probes before dying>" and optionally performs the setup of the TCP keepalive after accept() on the server side. An example string is "10,3,10", which means: start TCP keepalives after 10 idle seconds, wait 3 seconds between probes, declare connection "dead" if 10 probes failed

The UNIX domain socket (aka /var/run/pcscd.comm) still works as before. If the above mentioned pair of environment variables PCSC_HOST_OR_IP,PCSC_TCP_PORT is defined, then pcscd listens on this TCP socket, too (beside the normal UNIX domain socket). If these variables are defined before libpcsclite.so gets loaded on the client side, then libpcsclite.so does not use the UNIX domain socket anymore but the TCP socket instead.
  
The server side sets the LINGER time to zero (0). If the optional environment variable PCSC_TCP_KEEPALIVE is defined as well, then pcscd configure the TCP keepalives after having called accept() on a new client connection.

A second patch is currently being developed, introducing TLSv1.3 support via OpenSSL. **Please DO note these security implications:**
* Please use the TCP/IP patch only locally (localhost, from virtual guest to host, etc.);
* Please tunnel the traffic e.g. over SSH if the client and the server run on isolated hosts over the Internet; _you could go with OpenSSH alone in this scenario because OpenSSH is able to tunnel a UNIX domain socket already_;
* Once the TLSv1.3 version is ready, the tunneling is not necessary anymore (obviously).
  
Even if the smartcards make use of secure messaging, a connection between the client and the server over the Internet has to be secured because of man-in-the-middle attacks. Furthermore, PINs and PUKs may be transferred as cleartext!
