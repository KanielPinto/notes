| Command                                      | Explanation                                                                      |
| -------------------------------------------- | -------------------------------------------------------------------------------- |
| `tcpdump -i INTERFACE`                       | Captures packets on a specific network interface                                 |
| `tcpdump -w FILE`                            | Writes captured packets to a file                                                |
| `tcpdump -r FILE`                            | Reads captured packets from a file                                               |
| `tcpdump -c COUNT`                           | Captures a specific number of packets                                            |
| `tcpdump -n`                                 | Don’t resolve IP addresses                                                       |
| `tcpdump -nn`                                | Don’t resolve IP addresses and don’t resolve protocol numbers                    |
| `tcpdump -v`                                 | Verbose display; verbosity can be increased with `-vv` and `-vvv`                |
| `tcpdump host IP` or `tcpdump host HOSTNAME` | Filters packets by IP address or hostname                                        |
| `tcpdump src host IP` or                     | Filters packets by a specific source host                                        |
| `tcpdump dst host IP`                        | Filters packets by a specific destination host                                   |
| `tcpdump port PORT_NUMBER`                   | Filters packets by port number                                                   |
| `tcpdump src port PORT_NUMBER`               | Filters packets by the specified source port number                              |
| `tcpdump dst port PORT_NUMBER`               | Filters packets by the specified destination port number                         |
| `tcpdump PROTOCOL`                           | Filters packets by protocol; examples include `ip`, `ip6`, and `icmp`            |
| `tcpdump greater LENGTH`                     | Filters packets that have a length greater than or equal to the specified length |
| `tcpdump less LENGTH`                        | Filters packets that have a length less than or equal to the specified length    |
| `tcpdump -q`                                 | Quick and quite: brief packet information                                        |
| `tcpdump -e`                                 | Include MAC addresses                                                            |
| `tcpdump -A`                                 | Print packets as ASCII encoding                                                  |
| `tcpdump -xx`                                | Display packets in hexadecimal format                                            |
| `tcpdump -X`                                 | Show packets in both hexadecimal and ASCII formats                               |
Using pcap-filter, Tcpdump allows you to refer to the contents of any byte in the header using the following syntax `proto[expr:size]`, where:

- `proto` refers to the protocol. For example, `arp`, `ether`, `icmp`, `ip`, `ip6`, `tcp`, and `udp` refer to ARP, Ethernet, ICMP, IPv4, IPv6, TCP, and UDP respectively.
- `expr` indicates the byte offset, where `0` refers to the first byte.
- `size` indicates the number of bytes that interest us, which can be one, two, or four. It is optional and is one by default.

You can use `tcp[tcpflags]` to refer to the TCP flags field. The following TCP flags are available to compare with:

- `tcp-syn` TCP SYN (Synchronize)
- `tcp-ack` TCP ACK (Acknowledge)
- `tcp-fin` TCP FIN (Finish)
- `tcp-rst` TCP RST (Reset)
- `tcp-push` TCP Push