##############################
Network Bandwidth Expectations
##############################

What sort of network performance should I expect?
#################################################

The network bandwidth available to VMs will depend on the type of network your VMs are on and the contention on the hypervisor on which your VM is running:

Internal (172.16.*.*):

Private (project private networks eg 10.*.*.* or 192.168.*.*):

How can I test network bandwidth?
#################################

The best way to test network bandwidth is with iperf3. You will need two hosts, the location of these hosts will vary depending on what you are trying to test. If you want to test connection to an STFC service please contact us and we can arrange one of the hosts for testing.

On both hosts install iperf3

.. code :: bash

    yum install iperf3 -y

On one host (the server) start iperf3 in server mode (if we are providing a host to test with skip this)

.. code :: bash

    iperf3 -s

On the second host (the client) start

.. code :: bash

    iperf3 -c <ip of your first machine>

On the client side you should see an output like this:

.. code :: bash

    [root@test-private-network-3-1 ~]# iperf3 -c 192.168.248.65
    Connecting to host 192.168.248.65, port 5201
    [  4] local 192.168.248.52 port 51932 connected to 192.168.248.65 port 5201
    [ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
    [  4]   0.00-1.00   sec  1.87 GBytes  16.0 Gbits/sec   75   1.66 MBytes
    [  4]   1.00-2.00   sec  2.10 GBytes  18.0 Gbits/sec  159   1.39 MBytes
    [  4]   2.00-3.00   sec  2.13 GBytes  18.3 Gbits/sec   53   1.77 MBytes
    [  4]   3.00-4.00   sec  2.11 GBytes  18.2 Gbits/sec  256   1.45 MBytes
    [  4]   4.00-5.00   sec  2.14 GBytes  18.4 Gbits/sec   32   1.95 MBytes
    [  4]   5.00-6.00   sec  2.16 GBytes  18.5 Gbits/sec  209   1.76 MBytes
    [  4]   6.00-7.00   sec  2.16 GBytes  18.6 Gbits/sec    6   1.51 MBytes
    [  4]   7.00-8.00   sec  2.14 GBytes  18.4 Gbits/sec  110   1.56 MBytes
    [  4]   8.00-9.00   sec  2.15 GBytes  18.5 Gbits/sec   46   1.86 MBytes
    [  4]   9.00-10.00  sec  2.16 GBytes  18.5 Gbits/sec   42   1.69 MBytes
    - - - - - - - - - - - - - - - - - - - - - - - - -
    [ ID] Interval           Transfer     Bandwidth       Retr
    [  4]   0.00-10.00  sec  21.1 GBytes  18.1 Gbits/sec  988             sender
    [  4]   0.00-10.00  sec  21.1 GBytes  18.1 Gbits/sec                  receiver

    iperf Done.

From this you can see that there was an average bandwidth of 18.1 gbps during this test
