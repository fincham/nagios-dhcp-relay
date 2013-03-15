nagios-dhcp-relay
=================

This Nagios plugin checks a DHCP server is functional by making a faked "relayed" DHCP request to it, and verifying that it recieves a DHCP ACK message back.

It depends on the Scapy library from http://www.secdev.org/projects/scapy/ and must be run as root (or with the NET_RAW capability) since it uses raw sockets.

The "fake client" is hard coded to the MAC address `ab:ab:ab:ab:ab:ab` and to request the IP address `192.0.2.1` though these can be changed by editing the script. Timeout and number of retries can also be adjusted by editing the script. Sometimes Scapy seems to miss the reply packets, so retrying 2 or 3 times helps to return a more accurate result.

Otherwise it should be called from Nagios like so:

    ./check_dhcp_relay <source IP for nagios server> <IP of DHCP server>

And will return a result:

    Response: ack

And a return code of `0` if the DHCP request succeeds, otherwise it'll display the DHCP code returned by the server and return code `2`.

If your Nagios server has the IP `198.51.100.1` and your DHCP server is at `203.0.113.1` then one way of configuring ISC dhcpd to respond to test requests would be:

    shared-network nagios {
        # Fake client will request 192.0.2.1
        subnet 192.0.2.0 netmask 255.255.255.0 {
        }

        # Nagios server
        subnet 198.51.100.1 netmask 255.255.255.255 {
        }
    }

    host nagios {
        fixed-address 192.0.2.1;
        hardware ethernet ab:ab:ab:ab:ab:ab;
    }

Then a test run of the check command:

    nagios:~/nagios-dhcp-relay# ./check_dhcp_relay 198.51.100.1 203.0.113.1
    Response: ack

Which should be accompanied by a log entry looking like this in the DHCP server log:

    Mar 15 17:36:51 dhcp dhcpd: DHCPREQUEST for 192.0.2.1 from ab:ab:ab:ab:ab:ab via 198.51.100.1
    Mar 15 17:36:51 dhcp dhcpd: DHCPACK on 192.0.2.1 to ab:ab:ab:ab:ab:ab via 198.51.100.1

