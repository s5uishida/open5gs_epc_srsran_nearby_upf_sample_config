# Open5GS EPC & srsRAN 4G with ZeroMQ UE / RAN Sample Configuration - Select nearby UPF(PGW-U) according to the connected eNodeB
On 2023.05.05, Open5GS MME has a function to select SMF(PGW-C) by TAC and e_CellID.
Therefore I describe a very simple configuration that uses Open5GS and srsRAN 4G to select a nearby UPF(PGW-U) according to the connected eNodeB.

---

<a id="conf_list"></a>

## List of Sample Configurations

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. [One SMF, one UPF and one DNN](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
4. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
5. Select nearby UPF(PGW-U) according to the connected eNodeB (this article)
6. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
7. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
8. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
9. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
10. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
11. [Framed Routing](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config)
12. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
13. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
14. [eUPF(eBPF/XDP UPF(PGW-U))](https://github.com/s5uishida/open5gs_epc_srsran_eupf_sample_config)
15. [eUPF(eBPF/XDP UPF)](https://github.com/s5uishida/open5gs_5gc_ueransim_eupf_sample_config)

---

<a id="misc"></a>

## Miscellaneous Notes

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins)
- [A Note for Changing Network Interface of UPF from TUN to TAP in Open5GS](https://github.com/s5uishida/change_from_tun_to_tap_in_open5gs)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS CUPS-enabled EPC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS EPC and srsRAN 4G ZMQ UE / RAN](#changes)
  - [Changes in configuration files of Open5GS EPC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS EPC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS EPC U-Plane2](#changes_up2)
  - [Changes in configuration files of srsRAN 4G ZMQ UE / RAN](#changes_srs)
    - [Changes in configuration files of RAN (eNodeB1)](#changes_ran1)
    - [Changes in configuration files of RAN (eNodeB2)](#changes_ran2)
    - [Changes in configuration files of UE for Loc1 (IMSI-001010000000100)](#changes_ue_loc1)
    - [Changes in configuration files of UE for Loc2 (IMSI-001010000000100)](#changes_ue_loc2)
- [Network settings of Open5GS EPC and srsRAN 4G ZMQ UE / RAN](#network_settings)
  - [Network settings of Open5GS EPC C-Plane](#network_settings_cp)
  - [Network settings of Open5GS EPC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS EPC U-Plane2](#network_settings_up2)
  - [Network settings of srsRAN 4G ZMQ UE](#network_settings_ue)
- [Build Open5GS and srsRAN 4G ZMQ UE / RAN](#build)
- [Run Open5GS EPC and srsRAN 4G ZMQ UE / RAN](#run)
  - [Run Open5GS EPC C-Plane](#run_cp)
  - [Run Open5GS EPC U-Plane1 & U-Plane2](#run_up)
  - [Confirm in Loc1 (TAC=1)](#confirm_loc1)
    - [Run srsRAN 4G ZMQ RAN (eNodeB1) with TAC=1 in Loc1](#run_ran1)
    - [Run srsRAN 4G ZMQ UE (ue-loc1.conf) connected to eNodeB1 in Loc1](#run_ue1)
    - [Ping google.com going through PDN=10.45.0.0/16 on Loc1](#ping_ue1)
  - [Restart Open5GS MME](#restart_mme)
  - [Confirm in Loc2 (TAC=2)](#confirm_loc2)
    - [Run srsRAN 4G ZMQ RAN (eNodeB2) with TAC=2 in Loc2](#run_ran2)
    - [Run srsRAN 4G ZMQ UE (ue-loc2.conf) connected to eNodeB2 in Loc2](#run_ue2)
    - [Ping google.com going through PDN=10.46.0.0/16 on Loc2](#ping_ue2)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of Open5GS CUPS-enabled EPC Simulation Mobile Network

The following minimum configuration was set as a condition.
- The pair of eNodeB and SGW-U/UPF(PGW-U) exists in the same location.
- The UE connected to eNodeB connects to PDN managed by SGW-U/UPF(PGW-U) in the same location.

**In this example, TAC is matched to connect eNodeB and MME, and MME selects SGW-C/SMF(PGW-C) using TAC.**

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The EPC / UE / RAN used are as follows.
- EPC - Open5GS v2.6.4 (2023.09.02) - https://github.com/open5gs/open5gs
- UE / RAN - srsRAN 4G (2023.06.19) - https://github.com/srsran/srsRAN_4G

Each VMs are as follows.  
| VM# | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS EPC C-Plane | 192.168.0.111/24 <br> 192.168.0.112/24 <br> 192.168.0.113/24 <br> 192.168.0.114/24 <br> 192.168.0.115/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM2 | Open5GS EPC U-Plane1 | 192.168.0.116/24 <br> 192.168.0.117/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | Open5GS EPC U-Plane2 | 192.168.0.118/24 <br> 192.168.0.119/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | srsRAN 4G ZMQ RAN (eNodeB1) | 192.168.0.121/24 | Ubuntu 22.04 | 2GB | 10GB |
| VM5 | srsRAN 4G ZMQ RAN (eNodeB2) | 192.168.0.122/24 | Ubuntu 22.04 | 2GB | 10GB |
| VM6 | srsRAN 4G ZMQ UE | 192.168.0.123/24 <br> 192.168.0.124/24 | Ubuntu 22.04 | 2GB | 10GB |

MME, SGW-C, SMF(PGW-C) and PCRF addresses are as follows. 
| NF | IP address | Local address | Supported TACs |
| --- | --- | --- | --- |
| MME | 192.168.0.111 | 127.0.0.2 | 1, 2 |
| SGW-C1 | 192.168.0.112 | 127.0.0.3 | 1 |
| SMF1(PGW-C) | 192.168.0.113 | 127.0.0.4 | 1 |
| PCRF1 | -- | 127.0.0.9 | -- |
| SGW-C2 | 192.168.0.114 | 127.0.0.23 | 2 |
| SMF2(PGW-C) | 192.168.0.115 | 127.0.0.24 | 2 |
| PCRF2 | -- | 127.0.0.29 | -- |

The main information of eNodeBs is as follows.
| eNodeB# | Location# |IP address | MCC | MNC | TAC | eNodeB ID | Cell ID | E-UTRAN Cell ID |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| eNodeB1 | Loc1 | 192.168.0.121 | 001 | 01 | 1 | 0x19b | 0x01 | 0x19b01 |
| eNodeB2 | Loc2 | 192.168.0.122 | 001 | 01 | 2 | 0x19c | 0x01 | 0x19c01 |

Subscriber Information (other information is the same) is as follows.  
| UE | IMSI | PDN | OP/OPc | eNodeB# | IP address in ue.conf of srsRAN UE |
| --- | --- | --- | --- | --- | --- |
| UE | 001010000000100 | internet | OPc | eNodeB1 in Loc1 <br> eNodeB2 in Loc2 | 192.168.0.123 in ue-loc1.conf <br> 192.168.0.124 in ue-loc2.conf |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each PDNs are as follows.
| PDN | Location# |  TUNnel interface of PDN | APN | TUNnel interface of UE | U-Plane# |
| --- | --- | --- | --- | --- | --- |
| 10.45.0.0/16 | Loc1 | ogstun | internet | tun_srsue | U-Plane1 |
| 10.46.0.0/16 | Loc2 | ogstun | internet | tun_srsue | U-Plane2 |

<a id="changes"></a>

## Changes in configuration files of Open5GS EPC and srsRAN 4G ZMQ UE / RAN

Please refer to the following for building Open5GS and srsRAN 4G ZMQ UE / RAN respectively.
- Open5GS v2.6.4 (2023.09.02) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN 4G (2023.06.19) - https://docs.srsran.com/projects/4g/en/latest/

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS EPC C-Plane

- `open5gs/install/etc/open5gs/mme.yaml`
```diff
--- mme.yaml.orig       2023-09-03 15:52:59.000000000 +0900
+++ mme.yaml    2023-09-03 16:34:49.080255556 +0900
@@ -321,7 +321,7 @@
 mme:
     freeDiameter: /root/open5gs/install/etc/freeDiameter/mme.conf
     s1ap:
-      - addr: 127.0.0.2
+      - addr: 192.168.0.111
     gtpc:
       - addr: 127.0.0.2
     metrics:
@@ -329,15 +329,15 @@
         port: 9090
     gummei:
       plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       mme_gid: 2
       mme_code: 1
     tai:
       plmn_id:
-        mcc: 999
-        mnc: 70
-      tac: 1
+        mcc: 001
+        mnc: 01
+      tac: [1, 2]
     security:
         integrity_order : [ EIA2, EIA1, EIA0 ]
         ciphering_order : [ EEA0, EEA1, EEA2 ]
@@ -407,6 +407,9 @@
 sgwc:
     gtpc:
       - addr: 127.0.0.3
+        tac: 1
+      - addr: 127.0.0.23
+        tac: 2
 
 #
 # smf:
@@ -470,9 +473,10 @@
 #       e_cell_id: [12345, a9413, 98765]
 smf:
     gtpc:
-      - addr:
-        - 127.0.0.4
-        - ::1
+      - addr: 127.0.0.4
+        tac: 1
+      - addr: 127.0.0.24
+        tac: 2
 
 #
 # <GTPv1C Client>
```
- `open5gs/install/etc/open5gs/sgwc1.yaml`
```diff
--- sgwc.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ sgwc1.yaml  2023-05-03 10:50:12.000000000 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/sgwc.log
+    file: /root/open5gs/install/var/log/open5gs/sgwc1.log
 
 #
 #  <GTP-C Server>
@@ -81,7 +81,7 @@
     gtpc:
       - addr: 127.0.0.3
     pfcp:
-      - addr: 127.0.0.3
+      - addr: 192.168.0.112
 
 #
 #  <PFCP Client>>
@@ -130,7 +130,8 @@
 #
 sgwu:
     pfcp:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.116
+        apn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/sgwc2.yaml`
```diff
--- sgwc.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ sgwc2.yaml  2023-05-03 10:49:34.000000000 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/sgwc.log
+    file: /root/open5gs/install/var/log/open5gs/sgwc2.log
 
 #
 #  <GTP-C Server>
@@ -79,9 +79,9 @@
 #
 sgwc:
     gtpc:
-      - addr: 127.0.0.3
+      - addr: 127.0.0.23
     pfcp:
-      - addr: 127.0.0.3
+      - addr: 192.168.0.114
 
 #
 #  <PFCP Client>>
@@ -130,7 +130,8 @@
 #
 sgwu:
     pfcp:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.118
+        apn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/smf1.yaml`
```diff
--- smf.yaml.orig       2023-04-30 00:53:20.000000000 +0900
+++ smf1.yaml   2023-05-03 10:51:06.000000000 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf1.log
 
 #
 #  o TLS enable/disable
@@ -598,33 +598,24 @@
 #      maximum_integrity_protected_data_rate_downlink: bitrate64kbs|maximum-UE-rate
 #
 smf:
-    sbi:
-      - addr: 127.0.0.4
-        port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.113
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
     dns:
       - 8.8.8.8
       - 8.8.4.4
-      - 2001:4860:4860::8888
-      - 2001:4860:4860::8844
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf1.conf
 
 #
 #  <SBI Client>>
@@ -690,10 +681,6 @@
 #          l_linger: 10
 #
 #
-scp:
-    sbi:
-      - addr: 127.0.1.10
-        port: 7777
 
 #
 #  <SBI Client>>
@@ -808,7 +795,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.117
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/smf2.yaml`
```diff
--- smf.yaml.orig       2023-04-30 00:53:20.000000000 +0900
+++ smf2.yaml   2023-05-03 10:51:32.000000000 +0900
@@ -20,7 +20,7 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/smf.log
+    file: /root/open5gs/install/var/log/open5gs/smf2.log
 
 #
 #  o TLS enable/disable
@@ -598,33 +598,24 @@
 #      maximum_integrity_protected_data_rate_downlink: bitrate64kbs|maximum-UE-rate
 #
 smf:
-    sbi:
-      - addr: 127.0.0.4
-        port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.115
     gtpc:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 127.0.0.24
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.115
     metrics:
-      - addr: 127.0.0.4
+      - addr: 127.0.0.24
         port: 9090
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
     dns:
       - 8.8.8.8
       - 8.8.4.4
-      - 2001:4860:4860::8888
-      - 2001:4860:4860::8844
     mtu: 1400
     ctf:
       enabled: auto
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/smf2.conf
 
 #
 #  <SBI Client>>
@@ -690,10 +681,6 @@
 #          l_linger: 10
 #
 #
-scp:
-    sbi:
-      - addr: 127.0.1.10
-        port: 7777
 
 #
 #  <SBI Client>>
@@ -808,7 +795,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.119
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/pcrf1.yaml`
```diff
--- pcrf.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ pcrf1.yaml  2023-05-03 11:22:28.000000000 +0900
@@ -22,10 +22,10 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/pcrf.log
+    file: /root/open5gs/install/var/log/open5gs/pcrf1.log
 
 pcrf:
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/pcrf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/pcrf1.conf
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/open5gs/pcrf2.yaml`
```diff
--- pcrf.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ pcrf2.yaml  2023-05-03 11:23:00.000000000 +0900
@@ -22,10 +22,10 @@
 #    domain: core,sbi,ausf,event,tlv,mem,sock
 #
 logger:
-    file: /root/open5gs/install/var/log/open5gs/pcrf.log
+    file: /root/open5gs/install/var/log/open5gs/pcrf2.log
 
 pcrf:
-    freeDiameter: /root/open5gs/install/etc/freeDiameter/pcrf.conf
+    freeDiameter: /root/open5gs/install/etc/freeDiameter/pcrf2.conf
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```
- `open5gs/install/etc/freeDiameter/smf1.conf`  
`smf1.conf` is equal to the original `smf.conf`.

- `open5gs/install/etc/freeDiameter/smf2.conf`
```diff
--- smf.conf.orig       2023-04-30 00:53:22.000000000 +0900
+++ smf2.conf   2023-05-03 11:20:46.000000000 +0900
@@ -79,7 +79,7 @@
 #ListenOn = "202.249.37.5";
 #ListenOn = "2001:200:903:2::202:1";
 #ListenOn = "fe80::21c:5ff:fe98:7d62%eth0";
-ListenOn = "127.0.0.4";
+ListenOn = "127.0.0.24";
 
 
 ##############################################################
@@ -261,7 +261,7 @@
 # Examples:
 #ConnectPeer = "aaa.wide.ad.jp";
 #ConnectPeer = "old.diameter.serv" { TcTimer = 60; TLS_old_method; No_SCTP; Port=3868; } ;
-ConnectPeer = "pcrf.localdomain" { ConnectTo = "127.0.0.9"; No_TLS; };
+ConnectPeer = "pcrf.localdomain" { ConnectTo = "127.0.0.29"; No_TLS; };
 
 
 ##############################################################
```
- `open5gs/install/etc/freeDiameter/pcrf1.conf`  
`pcrf1.conf` is equal to the original `pcrf.conf`.

- `open5gs/install/etc/freeDiameter/pcrf2.conf`
```diff
--- pcrf.conf.orig      2023-04-30 00:53:22.000000000 +0900
+++ pcrf2.conf  2023-05-07 18:32:04.246141671 +0900
@@ -79,7 +79,7 @@
 #ListenOn = "202.249.37.5";
 #ListenOn = "2001:200:903:2::202:1";
 #ListenOn = "fe80::21c:5ff:fe98:7d62%eth0";
-ListenOn = "127.0.0.9";
+ListenOn = "127.0.0.29";
 
 
 ##############################################################
@@ -261,6 +261,6 @@
 # Examples:
 #ConnectPeer = "aaa.wide.ad.jp";
 #ConnectPeer = "old.diameter.serv" { TcTimer = 60; TLS_old_method; No_SCTP; Port=3868; } ;
-ConnectPeer = "smf.localdomain" { ConnectTo = "127.0.0.4"; No_TLS; };
+ConnectPeer = "smf.localdomain" { ConnectTo = "127.0.0.24"; No_TLS; };
```

<a id="changes_up1"></a>

### Changes in configuration files of Open5GS EPC U-Plane1

- `open5gs/install/etc/open5gs/sgwu.yaml`
```diff
--- sgwu.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ sgwu.yaml   2023-05-03 10:54:32.000000000 +0900
@@ -114,9 +114,9 @@
 #
 sgwu:
     pfcp:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.116
     gtpu:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.116
 
 #
 #  <PFCP Client>>
```
- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-04-30 00:53:20.000000000 +0900
+++ upf.yaml    2023-05-03 10:55:12.000000000 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.117
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.117
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<a id="changes_up2"></a>

### Changes in configuration files of Open5GS EPC U-Plane2

- `open5gs/install/etc/open5gs/sgwu.yaml`
```diff
--- sgwu.yaml.orig      2023-04-30 00:53:20.000000000 +0900
+++ sgwu.yaml   2023-05-03 10:56:36.000000000 +0900
@@ -114,9 +114,9 @@
 #
 sgwu:
     pfcp:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.118
     gtpu:
-      - addr: 127.0.0.6
+      - addr: 192.168.0.118
 
 #
 #  <PFCP Client>>
```
- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-04-30 00:53:20.000000000 +0900
+++ upf.yaml    2023-05-03 10:57:36.000000000 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.119
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.119
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.46.0.1/16
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<a id="changes_srs"></a>

### Changes in configuration files of srsRAN 4G ZMQ UE / RAN

<a id="changes_ran1"></a>

#### Changes in configuration files of RAN (eNodeB1)

- `srsRAN_4G/build/srsenb/enb.conf`
```diff
--- enb.conf.example    2023-05-02 10:51:20.000000000 +0900
+++ enb.conf    2023-05-03 11:39:50.000000000 +0900
@@ -22,9 +22,9 @@
 enb_id = 0x19B
 mcc = 001
 mnc = 01
-mme_addr = 127.0.1.100
-gtp_bind_addr = 127.0.1.1
-s1c_bind_addr = 127.0.1.1
+mme_addr = 192.168.0.111
+gtp_bind_addr = 192.168.0.121
+s1c_bind_addr = 192.168.0.121
 s1c_bind_port = 0
 n_prb = 50
 #tm = 4
@@ -80,8 +80,8 @@
 #time_adv_nsamples = auto
 
 # Example for ZMQ-based operation with TCP transport for I/Q samples
-#device_name = zmq
-#device_args = fail_on_disconnect=true,tx_port=tcp://*:2000,rx_port=tcp://localhost:2001,id=enb,base_srate=23.04e6
+device_name = zmq
+device_args = fail_on_disconnect=true,tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.123:2001,id=enb,base_srate=23.04e6
 
 #####################################################################
 # Packet capture configuration
```
- `srsRAN_4G/build/srsenb/rr.conf`
```diff
--- rr.conf.example     2023-05-02 10:51:20.000000000 +0900
+++ rr.conf     2023-05-02 11:52:54.000000000 +0900
@@ -55,7 +55,7 @@
   {
     // rf_port = 0;
     cell_id = 0x01;
-    tac = 0x0007;
+    tac = 0x0001;
     pci = 1;
     // root_seq_idx = 204;
     dl_earfcn = 3350;
```

<a id="changes_ran2"></a>

#### Changes in configuration files of RAN (eNodeB2)

- `srsRAN_4G/build/srsenb/enb.conf`
```diff
--- enb.conf.example    2023-05-02 10:51:20.000000000 +0900
+++ enb.conf    2023-05-05 16:47:30.000000000 +0900
@@ -19,12 +19,12 @@
 #
 #####################################################################
 [enb]
-enb_id = 0x19B
+enb_id = 0x19C
 mcc = 001
 mnc = 01
-mme_addr = 127.0.1.100
-gtp_bind_addr = 127.0.1.1
-s1c_bind_addr = 127.0.1.1
+mme_addr = 192.168.0.111
+gtp_bind_addr = 192.168.0.122
+s1c_bind_addr = 192.168.0.122
 s1c_bind_port = 0
 n_prb = 50
 #tm = 4
@@ -80,8 +80,8 @@
 #time_adv_nsamples = auto
 
 # Example for ZMQ-based operation with TCP transport for I/Q samples
-#device_name = zmq
-#device_args = fail_on_disconnect=true,tx_port=tcp://*:2000,rx_port=tcp://localhost:2001,id=enb,base_srate=23.04e6
+device_name = zmq
+device_args = fail_on_disconnect=true,tx_port=tcp://192.168.0.122:2000,rx_port=tcp://192.168.0.124:2001,id=enb,base_srate=23.04e6
 
 #####################################################################
 # Packet capture configuration
```
- `srsRAN_4G/build/srsenb/rr.conf`
```diff
--- rr.conf.example     2023-05-02 10:51:20.000000000 +0900
+++ rr.conf     2023-05-03 11:33:58.000000000 +0900
@@ -55,7 +55,7 @@
   {
     // rf_port = 0;
     cell_id = 0x01;
-    tac = 0x0007;
+    tac = 0x0002;
     pci = 1;
     // root_seq_idx = 204;
     dl_earfcn = 3350;
```

<a id="changes_ue_loc1"></a>

#### Changes in configuration files of UE for Loc1 (IMSI-001010000000100)

- `srsRAN_4G/build/srsue/ue-loc1.conf`
```diff
--- ue.conf.example     2023-05-05 16:18:12.000000000 +0900
+++ ue-loc1.conf        2023-05-03 11:35:38.000000000 +0900
@@ -42,8 +42,8 @@
 #continuous_tx     = auto
 
 # Example for ZMQ-based operation with TCP transport for I/Q samples
-#device_name = zmq
-#device_args = tx_port=tcp://*:2001,rx_port=tcp://localhost:2000,id=ue,base_srate=23.04e6
+device_name = zmq
+device_args = tx_port=tcp://192.168.0.123:2001,rx_port=tcp://192.168.0.121:2000,id=ue,base_srate=23.04e6
 
 #####################################################################
 # EUTRA RAT configuration
@@ -139,9 +139,9 @@
 [usim]
 mode = soft
 algo = milenage
-opc  = 63BFA50EE6523365FF14C1F45F88737D
-k    = 00112233445566778899aabbccddeeff
-imsi = 001010123456780
+opc  = E8ED289DEBA952E4283B54E88E6183CA
+k    = 465B5CE8B199B49FAA5F0A2EE238A6BC
+imsi = 001010000000100
 imei = 353490069873319
 #reader =
 #pin  = 1234
@@ -180,8 +180,8 @@
 #                      Supported: 0 - NULL, 1 - Snow3G, 2 - AES, 3 - ZUC
 #####################################################################
 [nas]
-#apn = internetinternet
-#apn_protocol = ipv4
+apn = internet
+apn_protocol = ipv4
 #user = srsuser
 #pass = srspass
 #force_imsi_attach = false
```

<a id="changes_ue_loc2"></a>

#### Changes in configuration files of UE for Loc2 (IMSI-001010000000100)

- `srsRAN_4G/build/srsue/ue-loc2.conf`
```diff
--- ue.conf.example     2023-05-05 16:18:12.000000000 +0900
+++ ue-loc2.conf        2023-05-05 16:33:18.000000000 +0900
@@ -42,8 +42,8 @@
 #continuous_tx     = auto
 
 # Example for ZMQ-based operation with TCP transport for I/Q samples
-#device_name = zmq
-#device_args = tx_port=tcp://*:2001,rx_port=tcp://localhost:2000,id=ue,base_srate=23.04e6
+device_name = zmq
+device_args = tx_port=tcp://192.168.0.124:2001,rx_port=tcp://192.168.0.122:2000,id=ue,base_srate=23.04e6
 
 #####################################################################
 # EUTRA RAT configuration
@@ -139,9 +139,9 @@
 [usim]
 mode = soft
 algo = milenage
-opc  = 63BFA50EE6523365FF14C1F45F88737D
-k    = 00112233445566778899aabbccddeeff
-imsi = 001010123456780
+opc  = E8ED289DEBA952E4283B54E88E6183CA
+k    = 465B5CE8B199B49FAA5F0A2EE238A6BC
+imsi = 001010000000100
 imei = 353490069873319
 #reader =
 #pin  = 1234
@@ -180,8 +180,8 @@
 #                      Supported: 0 - NULL, 1 - Snow3G, 2 - AES, 3 - ZUC
 #####################################################################
 [nas]
-#apn = internetinternet
-#apn_protocol = ipv4
+apn = internet
+apn_protocol = ipv4
 #user = srsuser
 #pass = srspass
 #force_imsi_attach = false
```

<a id="network_settings"></a>

## Network settings of Open5GS EPC and srsRAN 4G ZMQ UE / RAN

<a id="network_settings_cp"></a>

### Network settings of Open5GS EPC C-Plane

Add IP addresses for SGW-C1/SMF1(PGW-C) and SGW-C2/SMF2(PGW-C).
```
ip addr add 192.168.0.112/24 dev enp0s8
ip addr add 192.168.0.113/24 dev enp0s8
ip addr add 192.168.0.114/24 dev enp0s8
ip addr add 192.168.0.115/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<a id="network_settings_up1"></a>

### Network settings of Open5GS EPC U-Plane1

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, add IP address for UPF(PGW-U) and configure the TUNnel interface and NAPT.
```
ip addr add 192.168.0.117/24 dev enp0s8

ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

<a id="network_settings_up2"></a>

### Network settings of Open5GS EPC U-Plane2

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, add IP address for UPF(PGW-U) and configure the TUNnel interface and NAPT.
```
ip addr add 192.168.0.119/24 dev enp0s8

ip tuntap add name ogstun mode tun
ip addr add 10.46.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun -j MASQUERADE
```

<a id="network_settings_ue"></a>

### Network settings of srsRAN 4G ZMQ UE

Add IP address for UE in Loc2.
```
ip addr add 192.168.0.124/24 dev enp0s8
```

<a id="build"></a>

## Build Open5GS and srsRAN 4G ZMQ UE / RAN

Please refer to the following for building Open5GS and srsRAN 4G ZMQ UE / RAN respectively.
- Open5GS v2.6.4 (2023.09.02) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- srsRAN 4G (2023.06.19) - https://docs.srsran.com/projects/4g/en/latest/

Install MongoDB on Open5GS EPC C-Plane machine.
It is not necessary to install MongoDB on Open5GS EPC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
**See also [this](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins) for building srsRAN 4G.**

<a id="run"></a>

## Run Open5GS EPC and srsRAN 4G ZMQ UE / RAN

I will confirm in the following scenario.
The reasons to confirm in such a scenario are as follows:

**[The reason for Open5GS](https://github.com/open5gs/open5gs/issues/1791)**
- In the current Open5GS, SGW changes occurs only at TAU/Handover.

**[The reason for srsRAN 4G with ZMQ](https://docs.srsran.com/projects/4g/en/latest/app_notes/source/zeromq/source/index.html#known-issues)**
- For a clean tear down, the UE needs to be terminated first, then the eNB.
- eNB and UE can only run once, after the UE has been detached, the eNB needs to be restarted.
- Currently a single eNB and a single UE are only supported.

**This scenario:**
```
1) EPC start

     C-Plane start
               |
               V
    U-Plane1 start
               |
               V
    U-Plane2 start

2) eNodeB1/UE(Loc1) start, ping and eNodeB1/UE(Loc1) stop

     eNodeB1 start
               |
               V
    UE(Loc1) start
               |
               V
             ping google.com -I tun_srsue -n
               |
               V
    UE(Loc1) stop
               |
               V
     eNodeB1 stop

3) MME restart

4) eNodeB2/UE(Loc2) start, ping and eNodeB2/UE(Loc2) stop

     eNodeB2 start
               |
               V
    UE(Loc2) start
               |
               V
             ping google.com -I tun_srsue -n
               |
               V
    UE(Loc2) stop
               |
               V
     eNodeB2 stop
```
**Note. This scenario is just for confirming the function, and I think that restarting MME to change SGW is not proper for real usage.**

<a id="run_cp"></a>

### Run Open5GS EPC C-Plane

First, run Open5GS EPC C-Plane.

- Open5GS EPC C-Plane
```
./install/bin/open5gs-mmed &
./install/bin/open5gs-sgwcd -c install/etc/open5gs/sgwc1.yaml &
./install/bin/open5gs-sgwcd -c install/etc/open5gs/sgwc2.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf1.yaml &
./install/bin/open5gs-smfd -c install/etc/open5gs/smf2.yaml &
./install/bin/open5gs-hssd &
./install/bin/open5gs-pcrfd -c install/etc/open5gs/pcrf1.yaml &
./install/bin/open5gs-pcrfd -c install/etc/open5gs/pcrf2.yaml &
```

<a id="run_up"></a>

### Run Open5GS EPC U-Plane1 & U-Plane2

Next, run Open5GS EPC U-Planes.

- Open5GS EPC U-Plane1
```
./install/bin/open5gs-sgwud &
./install/bin/open5gs-upfd &
```
- Open5GS EPC U-Plane2
```
./install/bin/open5gs-sgwud &
./install/bin/open5gs-upfd &
```
Then run `tcpdump` on one more terminal for each U-Plane.
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ogstun, link-type RAW (Raw IP), snapshot length 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane2)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ogstun, link-type RAW (Raw IP), snapshot length 262144 bytes
```

<a id="confirm_loc1"></a>

### Confirm in Loc1 (TAC=1)

<a id="run_ran1"></a>

#### Run srsRAN 4G ZMQ RAN (eNodeB1) with TAC=1 in Loc1

Run srsRAN 4G ZMQ RAN (eNodeB1) and connect to Open5GS EPC.
```
# cd srsRAN_4G/build/srsenb
# ./src/srsenb enb.conf
---  Software Radio Systems LTE eNodeB  ---

Reading configuration file enb.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=fail_on_disconnect=true,tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.123:2001,id=enb,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
CHx id=enb
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.123:2001
CH0 tx_port=tcp://192.168.0.121:2000
CH0 fail_on_disconnect=true
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Setting frequency: DL=2680.0 Mhz, UL=2560.0 MHz for cc_idx=0 nof_prb=50

==== eNodeB started ===
Type <t> to view trace
```
The Open5GS C-Plane log when executed is as follows.
```
09/03 17:35:54.983: [mme] INFO: eNB-S1 accepted[192.168.0.121]:48973 in s1_path module (../src/mme/s1ap-sctp.c:114)
09/03 17:35:54.983: [mme] INFO: eNB-S1 accepted[192.168.0.121] in master_sm module (../src/mme/mme-sm.c:108)
09/03 17:35:54.983: [mme] INFO: [Added] Number of eNBs is now 1 (../src/mme/mme-context.c:2557)
09/03 17:35:54.983: [mme] INFO: eNB-S1[192.168.0.121] max_num_of_ostreams : 30 (../src/mme/mme-sm.c:150)
```

<a id="run_ue1"></a>

#### Run srsRAN 4G ZMQ UE (ue-loc1.conf) connected to eNodeB1 in Loc1

Run srsRAN 4G ZMQ UE (ue-loc1.conf), connect to eNodeB1 in Loc1 and connect to Open5GS EPC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue-loc1.conf
Reading configuration file ue-loc1.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.123:2001,rx_port=tcp://192.168.0.121:2000,id=ue,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
CHx id=ue
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.121:2000
CH0 tx_port=tcp://192.168.0.123:2001
Waiting PHY to initialize ... done!
Attaching UE...
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
.
Found Cell:  Mode=FDD, PCI=1, PRB=50, Ports=1, CP=Normal, CFO=-0.2 KHz
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Found PLMN:  Id=00101, TAC=1
Random Access Transmission: seq=32, tti=181, ra-rnti=0x2
RRC Connected
Random Access Complete.     c-rnti=0x46, ta=0
Network attach successful. IP: 10.45.0.2
 nTp) 3/9/2023 8:37:48 TZ:99
```
The Open5GS C-Plane log when executed is as follows.
```
09/03 17:37:47.657: [mme] INFO: InitialUEMessage (../src/mme/s1ap-handler.c:279)
09/03 17:37:47.657: [mme] INFO: [Added] Number of eNB-UEs is now 1 (../src/mme/mme-context.c:4426)
09/03 17:37:47.657: [mme] INFO: Unknown UE by S_TMSI[G:2,C:1,M_TMSI:0xc00001dc] (../src/mme/s1ap-handler.c:359)
09/03 17:37:47.657: [mme] INFO:     ENB_UE_S1AP_ID[1] MME_UE_S1AP_ID[1] TAC[1] CellID[0x19b01] (../src/mme/s1ap-handler.c:455)
09/03 17:37:47.658: [mme] INFO: Unknown UE by GUTI[G:2,C:1,M_TMSI:0xc00001dc] (../src/mme/mme-context.c:3280)
09/03 17:37:47.658: [mme] INFO: [Added] Number of MME-UEs is now 1 (../src/mme/mme-context.c:3089)
09/03 17:37:47.658: [emm] INFO: [] Attach request (../src/mme/emm-sm.c:412)
09/03 17:37:47.658: [emm] INFO:     GUTI[G:2,C:1,M_TMSI:0xc00001dc] IMSI[Unknown IMSI] (../src/mme/emm-handler.c:240)
09/03 17:37:47.710: [emm] INFO: Identity response (../src/mme/emm-sm.c:382)
09/03 17:37:47.710: [emm] INFO:     IMSI[001010000000100] (../src/mme/emm-handler.c:404)
09/03 17:37:47.859: [mme] INFO: [Added] Number of MME-Sessions is now 1 (../src/mme/mme-context.c:4440)
09/03 17:37:47.955: [sgwc] INFO: [Added] Number of SGWC-UEs is now 1 (../src/sgwc/context.c:237)
09/03 17:37:47.955: [sgwc] INFO: [Added] Number of SGWC-Sessions is now 1 (../src/sgwc/context.c:879)
09/03 17:37:47.955: [sgwc] INFO: UE IMSI[001010000000100] APN[internet] (../src/sgwc/s11-handler.c:237)
09/03 17:37:47.956: [gtp] INFO: gtp_connect() [127.0.0.4]:2123 (../lib/gtp/path.c:60)
09/03 17:37:47.957: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1010)
09/03 17:37:47.957: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3050)
09/03 17:37:47.957: [smf] INFO: UE IMSI[001010000000100] APN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/s5c-handler.c:255)
09/03 17:37:47.961: [gtp] INFO: gtp_connect() [192.168.0.117]:2152 (../lib/gtp/path.c:60)
09/03 17:37:48.306: [emm] INFO: [001010000000100] Attach complete (../src/mme/emm-sm.c:1298)
09/03 17:37:48.307: [emm] INFO:     IMSI[001010000000100] (../src/mme/emm-handler.c:278)
09/03 17:37:48.307: [emm] INFO:     UTC [2023-09-03T08:37:48] Timezone[0]/DST[0] (../src/mme/emm-handler.c:285)
09/03 17:37:48.307: [emm] INFO:     LOCAL [2023-09-03T17:37:48] Timezone[32400]/DST[0] (../src/mme/emm-handler.c:289)
```
The Open5GS U-Plane1 log when executed is as follows.
```
09/03 17:37:47.952: [sgwu] INFO: UE F-SEID[UP:0x89d CP:0x73e] (../src/sgwu/context.c:169)
09/03 17:37:47.952: [sgwu] INFO: [Added] Number of SGWU-Sessions is now 1 (../src/sgwu/context.c:174)
09/03 17:37:47.956: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:206)
09/03 17:37:47.956: [gtp] INFO: gtp_connect() [192.168.0.116]:2152 (../lib/gtp/path.c:60)
09/03 17:37:47.956: [gtp] INFO: gtp_connect() [192.168.0.113]:2152 (../lib/gtp/path.c:60)
09/03 17:37:47.956: [upf] INFO: UE F-SEID[UP:0xded CP:0x791] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:483)
09/03 17:37:47.956: [upf] INFO: UE F-SEID[UP:0xded CP:0x791] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:483)
09/03 17:37:47.957: [gtp] INFO: gtp_connect() [192.168.0.117]:2152 (../lib/gtp/path.c:60)
09/03 17:37:48.304: [gtp] INFO: gtp_connect() [192.168.0.121]:2152 (../lib/gtp/path.c:60)
```
The result of `ip addr show` on VM6 (UE) is as follows.
```
# ip addr show
...
6: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global tun_srsue
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue1"></a>

#### Ping google.com going through PDN=10.45.0.0/16 on Loc1

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane1.
```
# ping google.com -I tun_srsue -n
PING google.com (172.217.161.78) from 10.45.0.2 tun_srsue: 56(84) bytes of data.
64 bytes from 172.217.161.78: icmp_seq=1 ttl=61 time=438 ms
64 bytes from 172.217.161.78: icmp_seq=2 ttl=61 time=66.6 ms
64 bytes from 172.217.161.78: icmp_seq=3 ttl=61 time=69.9 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
17:39:33.917147 IP 10.45.0.2 > 172.217.161.78: ICMP echo request, id 3, seq 1, length 64
17:39:33.942728 IP 172.217.161.78 > 10.45.0.2: ICMP echo reply, id 3, seq 1, length 64
17:39:34.591150 IP 10.45.0.2 > 172.217.161.78: ICMP echo request, id 3, seq 2, length 64
17:39:34.606917 IP 172.217.161.78 > 10.45.0.2: ICMP echo reply, id 3, seq 2, length 64
17:39:35.593994 IP 10.45.0.2 > 172.217.161.78: ICMP echo request, id 3, seq 3, length 64
17:39:35.610527 IP 172.217.161.78 > 10.45.0.2: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the PDN of U-Plane1 in the same Loc1 according to the connected eNodeB1 in Loc1.**

**After this, stop eNodeB1 and UE in preparation for confirming Loc2 with TAC=2.**

<a id="restart_mme"></a>

### Restart Open5GS MME

Restart Open5GS MME in preparation for confirming Loc2 with TAC=2.
```
pkill open5gs-mmed
./install/bin/open5gs-mmed &
```

<a id="confirm_loc2"></a>

### Confirm in Loc2 (TAC=2)

<a id="run_ran2"></a>

#### Run srsRAN 4G ZMQ RAN (eNodeB2) with TAC=2 in Loc2

Run srsRAN 4G ZMQ RAN (eNodeB2) and connect to Open5GS EPC.
```
# cd srsRAN_4G/build/srsenb
# ./src/srsenb enb.conf
---  Software Radio Systems LTE eNodeB  ---

Reading configuration file enb.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=fail_on_disconnect=true,tx_port=tcp://192.168.0.122:2000,rx_port=tcp://192.168.0.124:2001,id=enb,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
CHx id=enb
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.124:2001
CH0 tx_port=tcp://192.168.0.122:2000
CH0 fail_on_disconnect=true
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Setting frequency: DL=2680.0 Mhz, UL=2560.0 MHz for cc_idx=0 nof_prb=50

==== eNodeB started ===
Type <t> to view trace
```
The Open5GS C-Plane log when executed is as follows.
```
09/03 17:50:00.581: [mme] INFO: eNB-S1 accepted[192.168.0.122]:40020 in s1_path module (../src/mme/s1ap-sctp.c:114)
09/03 17:50:00.581: [mme] INFO: eNB-S1 accepted[192.168.0.122] in master_sm module (../src/mme/mme-sm.c:108)
09/03 17:50:00.581: [mme] INFO: [Added] Number of eNBs is now 1 (../src/mme/mme-context.c:2557)
09/03 17:50:00.581: [mme] INFO: eNB-S1[192.168.0.122] max_num_of_ostreams : 30 (../src/mme/mme-sm.c:150)
```

<a id="run_ue2"></a>

#### Run srsRAN 4G ZMQ UE (ue-loc2.conf) connected to eNodeB2 in Loc2

Run srsRAN 4G ZMQ UE (ue-loc2.conf), connect to eNodeB2 in Loc2 and connect to Open5GS EPC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue-loc2.conf
Reading configuration file ue-loc2.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.124:2001,rx_port=tcp://192.168.0.122:2000,id=ue,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
CHx id=ue
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.122:2000
CH0 tx_port=tcp://192.168.0.124:2001
Waiting PHY to initialize ... done!
Attaching UE...
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
.
Found Cell:  Mode=FDD, PCI=1, PRB=50, Ports=1, CP=Normal, CFO=-0.2 KHz
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Current sample rate is 11.52 MHz with a base rate of 23.04 MHz (x2 decimation)
Found PLMN:  Id=00101, TAC=2
Random Access Transmission: seq=33, tti=181, ra-rnti=0x2
RRC Connected
Random Access Complete.     c-rnti=0x46, ta=0
Network attach successful. IP: 10.46.0.2
 nTp) 3/9/2023 8:51:14 TZ:99
```
The Open5GS C-Plane log when executed is as follows.
```
09/03 17:51:13.952: [mme] INFO: InitialUEMessage (../src/mme/s1ap-handler.c:279)
09/03 17:51:13.952: [mme] INFO: [Added] Number of eNB-UEs is now 1 (../src/mme/mme-context.c:4426)
09/03 17:51:13.952: [mme] INFO: Unknown UE by S_TMSI[G:2,C:1,M_TMSI:0xc00005b2] (../src/mme/s1ap-handler.c:359)
09/03 17:51:13.952: [mme] INFO:     ENB_UE_S1AP_ID[1] MME_UE_S1AP_ID[1] TAC[2] CellID[0x19c01] (../src/mme/s1ap-handler.c:455)
09/03 17:51:13.952: [mme] INFO: Unknown UE by GUTI[G:2,C:1,M_TMSI:0xc00005b2] (../src/mme/mme-context.c:3280)
09/03 17:51:13.952: [mme] INFO: [Added] Number of MME-UEs is now 1 (../src/mme/mme-context.c:3089)
09/03 17:51:13.952: [emm] INFO: [] Attach request (../src/mme/emm-sm.c:412)
09/03 17:51:13.952: [emm] INFO:     GUTI[G:2,C:1,M_TMSI:0xc00005b2] IMSI[Unknown IMSI] (../src/mme/emm-handler.c:240)
09/03 17:51:14.004: [emm] INFO: Identity response (../src/mme/emm-sm.c:382)
09/03 17:51:14.004: [emm] INFO:     IMSI[001010000000100] (../src/mme/emm-handler.c:404)
09/03 17:51:14.106: [mme] INFO: [Added] Number of MME-Sessions is now 1 (../src/mme/mme-context.c:4440)
09/03 17:51:14.200: [sgwc] INFO: [Added] Number of SGWC-UEs is now 1 (../src/sgwc/context.c:237)
09/03 17:51:14.200: [sgwc] INFO: [Added] Number of SGWC-Sessions is now 1 (../src/sgwc/context.c:879)
09/03 17:51:14.200: [sgwc] INFO: UE IMSI[001010000000100] APN[internet] (../src/sgwc/s11-handler.c:237)
09/03 17:51:14.201: [gtp] INFO: gtp_connect() [127.0.0.24]:2123 (../lib/gtp/path.c:60)
09/03 17:51:14.201: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1010)
09/03 17:51:14.201: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3050)
09/03 17:51:14.201: [smf] INFO: UE IMSI[001010000000100] APN[internet] IPv4[10.46.0.2] IPv6[] (../src/smf/s5c-handler.c:255)
09/03 17:51:14.205: [gtp] INFO: gtp_connect() [192.168.0.119]:2152 (../lib/gtp/path.c:60)
09/03 17:51:14.546: [emm] INFO: [001010000000100] Attach complete (../src/mme/emm-sm.c:1298)
09/03 17:51:14.547: [emm] INFO:     IMSI[001010000000100] (../src/mme/emm-handler.c:278)
09/03 17:51:14.547: [emm] INFO:     UTC [2023-09-03T08:51:14] Timezone[0]/DST[0] (../src/mme/emm-handler.c:285)
09/03 17:51:14.547: [emm] INFO:     LOCAL [2023-09-03T17:51:14] Timezone[32400]/DST[0] (../src/mme/emm-handler.c:289)
```
The Open5GS U-Plane2 log when executed is as follows.
```
09/03 17:51:14.199: [sgwu] INFO: UE F-SEID[UP:0x804 CP:0x801] (../src/sgwu/context.c:169)
09/03 17:51:14.199: [sgwu] INFO: [Added] Number of SGWU-Sessions is now 1 (../src/sgwu/context.c:174)
09/03 17:51:14.203: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:206)
09/03 17:51:14.203: [gtp] INFO: gtp_connect() [192.168.0.118]:2152 (../lib/gtp/path.c:60)
09/03 17:51:14.203: [gtp] INFO: gtp_connect() [192.168.0.115]:2152 (../lib/gtp/path.c:60)
09/03 17:51:14.203: [upf] INFO: UE F-SEID[UP:0x3b4 CP:0xff0] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:483)
09/03 17:51:14.203: [upf] INFO: UE F-SEID[UP:0x3b4 CP:0xff0] APN[internet] PDN-Type[1] IPv4[10.46.0.2] IPv6[] (../src/upf/context.c:483)
09/03 17:51:14.204: [gtp] INFO: gtp_connect() [192.168.0.119]:2152 (../lib/gtp/path.c:60)
09/03 17:51:14.547: [gtp] INFO: gtp_connect() [192.168.0.122]:2152 (../lib/gtp/path.c:60)
```
The result of `ip addr show` on VM6 (UE) is as follows.
```
# ip addr show
...
8: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.46.0.2/24 scope global tun_srsue
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue2"></a>

#### Ping google.com going through PDN=10.46.0.0/16 on Loc2

Confirm by using `tcpdump` that the packet goes through `if=ogstun` on U-Plane2.
```
# ping google.com -I tun_srsue -n
PING google.com (172.217.161.78) from 10.46.0.2 tun_srsue: 56(84) bytes of data.
64 bytes from 172.217.161.78: icmp_seq=1 ttl=61 time=78.7 ms
64 bytes from 172.217.161.78: icmp_seq=2 ttl=61 time=71.7 ms
64 bytes from 172.217.161.78: icmp_seq=3 ttl=61 time=105 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
17:53:03.985589 IP 10.46.0.2 > 172.217.161.78: ICMP echo request, id 6, seq 1, length 64
17:53:04.007017 IP 172.217.161.78 > 10.46.0.2: ICMP echo reply, id 6, seq 1, length 64
17:53:04.985819 IP 10.46.0.2 > 172.217.161.78: ICMP echo request, id 6, seq 2, length 64
17:53:05.001360 IP 172.217.161.78 > 10.46.0.2: ICMP echo reply, id 6, seq 2, length 64
17:53:06.018756 IP 10.46.0.2 > 172.217.161.78: ICMP echo request, id 6, seq 3, length 64
17:53:06.034966 IP 172.217.161.78 > 10.46.0.2: ICMP echo reply, id 6, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the PDN of U-Plane2 in the same Loc2 according to the connected eNodeB2 in Loc2.**

---
This makes it to confirm that at Initial Attach, MME can select SMF(PGW-C) by TAC according to the connected eNodeB, and UE connects to SGW-U/UPF(PGW-U) in the same location.
I would like to thank the excellent developers and all the contributors of Open5GS and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2023.09.03] Updated Open5GS and srsRAN 4G.
- [2023.05.07] Initial release.
