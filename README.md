# multiSI
using a single virtual machine instance for multiple service instances
```
docker network create -d opencontrail -o name=dummy1 --subnet 172.16.0.0/29 dummy1
docker network create -d opencontrail -o name=dummy2 --subnet 172.16.0.8/29 dummy2

docker create --name SI --cap-add NET_ADMIN --net dummy1 alpine:3.1 tail -f /dev/null \
     && docker network connect dummy2 SI && docker start SI

docker network create -d opencontrail -o rt=1:2 -o name=net1_2 --subnet 10.1.2.0/24 net1_2
docker network create -d opencontrail -o rt=1:3 -o name=net1_3 --subnet 10.1.3.0/24 net1_3

docker network create -d opencontrail -o rt=2:1 -o name=net2_1 --subnet 10.2.1.0/24 net2_1
docker network create -d opencontrail -o rt=2:3 -o name=net2_3 --subnet 10.2.3.0/24 net2_3

docker network create -d opencontrail -o rt=3:1 -o name=net3_1 --subnet 10.3.1.0/24 net3_1
docker network create -d opencontrail -o rt=3:2 -o name=net3_2 --subnet 10.3.2.0/24 net3_2

docker exec -ti SI /bin/bash

ip link add name net12 link eth0 type vlan id 12
ip link add name net13 link eth0 type vlan id 13
ip link add name net23 link eth0 type vlan id 23

ip link add name net21 link eth1 type vlan id 12
ip link add name net31 link eth1 type vlan id 13
ip link add name net32 link eth1 type vlan id 23

ip link set address de:ad:be:ef:ba:12 dev net12
ip link set address de:ad:be:ef:ba:13 dev net13
ip link set address de:ad:be:ef:ba:23 dev net23

ip link set address de:ad:be:ef:ba:21 dev net21
ip link set address de:ad:be:ef:ba:31 dev net31
ip link set address de:ad:be:ef:ba:32 dev net32

ip address add 10.1.2.254/24 dev net12
ip address add 10.1.3.254/24 dev net13
ip address add 10.2.3.254/24 dev net23

ip address add 10.2.1.254/24 dev net21
ip address add 10.3.1.254/24 dev net31
ip address add 10.3.2.254/24 dev net32

ip link set dev net12 up
ip link set dev net13 up
ip link set dev net23 up

ip link set dev net21 up
ip link set dev net31 up
ip link set dev net32 up

docker run -itd --name 1_2 --net net1_2 alpine:3.1 /bin/sh
docker run -itd --name 1_3 --net net1_3 alpine:3.1 /bin/sh
docker run -itd --name 2_3 --net net2_3 alpine:3.1 /bin/sh

docker run -itd --name 2_1 --net net2_1 alpine:3.1 /bin/sh
docker run -itd --name 3_1 --net net3_1 alpine:3.1 /bin/sh
docker run -itd --name 3_2 --net net3_2 alpine:3.1 /bin/sh
```
```
                    +-----------------------+
                    |                       |
                    |   CONTAINER           |
                    |                       |
                    +----+                  |
                    |eth0|                  |
+-----------+       |    +-------+          |
|net1_2     |       |    |eth0.12|          |
|rt1:2      +--VLAN12----+       +-+        |
|10.1.2.0/24|       |    |.254   | |        |
+-----------+       |    +-------+ |        |
                    |    |         |        |
+-----------+       |    +-------+ |        |
|net1_3     |       |    |eth0.13| |        |
|rt1:3      +--VLAN13----+       +----+     |
|10.1.3.0/24|       |    |.254   | |  |     |
+-----------+       |    +-------+ |  |     |
                    |    |         |  |     |
+-----------+       |    +-------+ |  |     |
|net2_3     |       |    |eth0.23| |  |     |
|rt2:3      +--VLAN23----+       +-------+  |
|10.2.3.0/23|       |    |.254   | |  |  |  |
+-----------+       |    +-------+ |  |  |  |
                    |    |         |  |  |  |
                    +----+         |  |  |  |
                    |              |  |  |  |
                    +----+         |  |  |  |
                    |    |         |  |  |  |
+-----------+       |    +-------+ |  |  |  |
|net2_1     |       |    |eth1.12| |  |  |  |
|rt2:1      +--VLAN12----+       +-+  |  |  |
|10.2.1.0/24|       |    |.254   |    |  |  |
+-----------+       |    +-------+    |  |  |
                    |    |            |  |  |
+-----------+       |    +-------+    |  |  |
|net3_1     |       |    |eth1.13|    |  |  |
|rt3:1      +--VLAN13----+       +----+  |  |
|10.3.1.0/24|       |    |.254   |       |  |
+-----------+       |    +-------+       |  |
                    |    |               |  |
+-----------+       |    +-------+       |  |
|net3_2     |       |    |eth1.23|       |  |
|rt3:2      +--VLAN23----+       +-------+  |
|10.3.2.0/24|       |    |.254   |          |
+-----------+       |    +-------+          |
                    |eth1|                  |
                    +----+                  |
                    |                       |
                    +-----------------------+
```
