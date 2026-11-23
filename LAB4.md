# Lab 4 — 5G CU/DU Split Deployment and F1-C/F1-U Analysis

## Task 1 — Inspect the Split Configurations

### Configuration Verification
Both the gNB-CU (`gnb-cu.sa.f1.conf`) and gNB-DU (`gnb-du.sa.band78.106prb.rfsim.pci0.conf`) configuration files were verified under `~/openairinterface5g-v2.4.0/targets/PROJECTS/GENERIC-NR-5GC/CONF/`.

| Parameter | Observed Value | Source File |
| :--- | :--- | :--- |
| **CU F1 Address (`local_s_address`)** | `127.0.0.3` | `gnb-cu.sa.f1.conf` |
| **CU Remote Address (`remote_s_address`)** | `0.0.0.0` (accepts multiple DUs) | `gnb-cu.sa.f1.conf` |
| **DU F1 Address (`local_n_address`)** | `127.0.0.4` | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |
| **DU Target CU (`remote_n_address`)** | `127.0.0.3` | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |
| **F1-U GTP-U Ports (`portd`)** | `2152` (CU & DU) | Both files |
| **AMF IP Address** | `192.168.70.132` | `gnb-cu.sa.f1.conf` |
| **Physical Cell ID (`physCellId`)** | `0` | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |
| **DU SSB NR-ARFCN (`absoluteFrequencySSB`)** | `630048` (3,450.72 MHz) | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |
| **Point A (`dl_absoluteFrequencyPointA`)** | `628776` | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |
| **Carrier Bandwidth (`dl_carrierBandwidth`)** | `106` PRBs | `gnb-du.sa.band78.106prb.rfsim.pci0.conf` |

<details>
<summary>raw CLI output</summary>

```bash
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-cu.sa.f1.conf:20:    local_s_address     = "127.0.0.3";
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-cu.sa.f1.conf:21:    remote_s_address    = "0.0.0.0"; # multiple DUs
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-cu.sa.f1.conf:22:    local_s_portd       = 2152;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-cu.sa.f1.conf:23:    remote_s_portd      = 2152;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-cu.sa.f1.conf:36:    amf_ip_address = ({ ipv4 = "192.168.70.132"; });
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:21:    min_rxtxtime = 4;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:27:      physCellId                                               = 0;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:32:      absoluteFrequencySSB                                      = 630048;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:35:      dl_absoluteFrequencyPointA                                = 628776;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:41:        dl_carrierBandwidth                                     = 106;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:160:    local_n_address     = "127.0.0.4";
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:161:    remote_n_address    = "127.0.0.3";
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:162:    local_n_portd       = 2152;
targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb-du.sa.band78.106prb.rfsim.pci0.conf:163:    remote_n_portd      = 2152;
```

</details>

### Questions (Task 1)

1. **Which protocol layers mainly reside in the CU, and which reside in the DU?**
   - **gNB-CU (Central Unit):** Hosts the higher, non-real-time protocol layers — specifically **RRC** (Radio Resource Control), **SDAP** (Service Data Adaptation Protocol), and **PDCP** (Packet Data Convergence Protocol).
   - **gNB-DU (Distributed Unit):** Hosts the lower, real-time protocol layers — specifically **RLC** (Radio Link Control), **MAC** (Medium Access Control), and **PHY** (Physical layer).

2. **What is the purpose of the F1 interface?**
   - The **F1 interface** connects the gNB-CU and the gNB-DU in a split 5G RAN architecture. It is divided into **F1-C** (Control Plane, using F1AP over SCTP) to exchange RAN signaling such as F1 Setup, RRC message transfers, and UE Context management, and **F1-U** (User Plane, using GTP-U over UDP) to transport UE user-plane data packets between the CU and DU.

3. **Why must the UE radio frequency be compatible with the DU rather than taken blindly from the previous lab?**
   - In a split architecture, the **gNB-DU** handles the PHY layer and transmits the Synchronization Signal Block (SSB) and system information over the radio carrier. While Lab 3's monolithic gNB used `absoluteFrequencySSB = 641280` (3,619.20 MHz), this split DU uses `630048` (3,450.72 MHz). If the UE searches on the wrong carrier frequency (`-C 3619200000`), it will fail to detect the SSB, synchronize, or decode SIB1.

---

## Task 2 — Deploy the Split 5G SA Network

The monolithic gNB from Lab 3 was replaced with a split **gNB-CU** (`gnb-cu.sa.f1.conf`) and **gNB-DU** (`gnb-du.sa.band78.106prb.rfsim.pci0.conf`) running over RFsimulator alongside the OAI 5G Core (`oai-cn5g`) and `nrUE` (`-C 3450720000`, IMSI `001010000000001`).

### Startup & Connection Evidence

**gNB-CU Startup:**

<img width="813" height="617" alt="gNB-CU startup" src="https://github.com/user-attachments/assets/bc73e468-a4af-41a1-990b-2eaae5a30137" />

**gNB-DU Startup:**

<img width="813" height="618" alt="gNB-DU startup" src="https://github.com/user-attachments/assets/69f8e721-6c85-415a-b725-ded396237b52" />

**nrUE Connected:**

<img width="937" height="753" alt="nrUE connected" src="https://github.com/user-attachments/assets/ca0774b5-ef08-491c-a9bc-6daf9bc5c100" />

**gNB-DU Connection with UE:**

<img width="813" height="618" alt="gNB-DU connection with UE" src="https://github.com/user-attachments/assets/8741fe87-ab92-4bb2-a239-d81aac0a4727" />

**UE Tunnel & External DN Ping:**

<img width="937" height="753" alt="UE Tunnel and Ping 10 packets" src="https://github.com/user-attachments/assets/5e087f29-b91a-41c0-8bd6-a9597287bb89" />

<img width="937" height="753" alt="UE Ping 30 packets" src="https://github.com/user-attachments/assets/247cd85e-9f4c-41f7-b275-a057f633a39a" />

### Observed Connectivity Results

- **UE Tunnel Interface:** `oaitun_ue1`
- **Assigned UE IP:** `10.0.0.2/24`
- **Target External DN:** `192.168.70.135`

| Test Command | Transmitted | Received | Packet Loss | Min / Avg / Max RTT |
| :--- | :---: | :---: | :---: | :---: |
| `ping -I oaitun_ue1 -c 10 192.168.70.135` | 10 | 10 | 0% | 131.443 / **151.401** / 168.835 ms |
| `ping -I oaitun_ue1 -c 30 -i 0.2 192.168.70.135` | 30 | 30 | 0% | 85.249 / **109.547** / 161.508 ms |

---

## Task 3 — Capture F1-C and F1-U in a Single PCAP

Before starting the gNB-DU, a `tcpdump` capture was started on the loopback interface (`lo`) filtering traffic between the CU (`127.0.0.3`) and DU (`127.0.0.4`) for both SCTP (F1-C) and UDP port 2152 (F1-U):

```bash
sudo tcpdump -i lo -s 0 -nn \
  -w ~/lab4-f1-complete.pcap \
  'host 127.0.0.3 and host 127.0.0.4 and (sctp or udp port 2152)'
```

The DU and nrUE were then started, followed by the 30-packet Ping test (`-c 30 -i 0.2`) to generate user-plane traffic. The resulting capture file is included in this repository (`lab4-f1-complete.pcap`).

---

## Task 4 — Analyze F1-C (F1AP / SCTP)

The `lab4-f1-complete.pcap` capture (137 total packets) was inspected in Wireshark using the `sctp` (52 packets) and `f1ap` (24 packets) display filters.

### Wireshark Evidence

**SCTP Association & F1 Setup (`sctp` filter):**

<img width="1510" height="946" alt="SCTP Association and F1 Setup" src="https://github.com/user-attachments/assets/f60bf8cb-e473-4bd2-a47f-7ba4dd4738d4" />

**Complete F1AP Control-Plane & UE-Associated Signaling (`f1ap` filter):**

<img width="1512" height="950" alt="F1AP UE Messages" src="https://github.com/user-attachments/assets/2d5d26b5-dd7f-43b0-a86e-a4eb66ec32a4" />

### Observed F1-C Message Sequence

| Packet No. | Source → Destination | Protocol | Message / Procedure | Meaning |
| :---: | :--- | :--- | :--- | :--- |
| **1–4** | `127.0.0.4` ↔ `127.0.0.3` | SCTP | `INIT`, `INIT_ACK`, `COOKIE_ECHO`, `COOKIE_ACK` | Establishes the SCTP transport association (ports `35255` ↔ `38472`) |
| **5** | `127.0.0.4` → `127.0.0.3` | F1AP | `F1SetupRequest` (MIB, SIB1) | DU initiates F1 interface setup and reports served cell info to CU |
| **7** | `127.0.0.3` → `127.0.0.4` | F1AP | `F1SetupResponse` | CU accepts the F1 Setup request |
| **9–10** | `127.0.0.4` ↔ `127.0.0.3` | F1AP | `GNBDUConfigurationUpdate` / `Acknowledge` | DU updates CU with active cell configuration; CU acknowledges |
| **16** | `127.0.0.4` → `127.0.0.3` | F1AP / NR RRC | `InitialULRRCMessageTransfer` (`RRC Setup Request`) | DU forwards the UE's initial RRC connection request to CU |
| **17** | `127.0.0.3` → `127.0.0.4` | F1AP / NR RRC | `DLRRCMessageTransfer` (`RRC Setup`) | CU sends RRC Setup configuration back to DU for the UE |
| **18** | `127.0.0.4` → `127.0.0.3` | F1AP / NR RRC / NAS-5GS | `ULRRCMessageTransfer` (`RRC Setup Complete`, `Registration request`) | UE completes RRC setup and piggybacks its NAS Registration Request |
| **20–24** | `127.0.0.3` ↔ `127.0.0.4` | F1AP / NR RRC / NAS-5GS | `DL/UL Information Transfer` (Authentication & NAS Security) | NAS authentication and security mode procedures relayed via RRC/F1AP |
| **26–29** | `127.0.0.3` ↔ `127.0.0.4` | F1AP / NR RRC | `Security Mode Command/Complete`, `UE Capability Enquiry/Information` | AS (Access Stratum) security activation and UE capability exchange |
| **34–35** | `127.0.0.3` ↔ `127.0.0.4` | F1AP | `UEContextSetupRequest` / `UEContextSetupResponse` | CU requests DU to allocate UE-specific radio/bearer resources; DU confirms |
| **36–37** | `127.0.0.3` ↔ `127.0.0.4` | F1AP / NR RRC / NAS-5GS | `DLRRCMessageTransfer` (`RRC Reconfiguration`) / `Complete` | CU configures DRB/SRB2 and delivers PDU session / NAS signaling to UE |
| **38–39** | `127.0.0.3` ↔ `127.0.0.4` | F1AP | `UEContextModificationRequest` / `Response` | CU updates the established UE context on the DU |

### Questions (Task 4)

1. **Which node initiates F1 Setup, and which acknowledges it?**
   - The **gNB-DU** (`127.0.0.4`) initiates the procedure by sending `F1SetupRequest` (Packet #5), and the **gNB-CU** (`127.0.0.3`) acknowledges and accepts it with `F1SetupResponse` (Packet #7).

2. **Explain the difference between SCTP association establishment and F1 Setup.**
   - **SCTP association establishment** (Packets #1–4: `INIT`, `INIT_ACK`, `COOKIE_ECHO`, `COOKIE_ACK`) operates at the **transport layer** to create a reliable, message-oriented connection between the CU and DU IP/port endpoints.
   - **F1 Setup** (Packets #5 and #7) operates at the **application layer (F1AP)** on top of that SCTP association to exchange 5G RAN operational data (such as DU identity, served cells, MIB/SIB1, and RRC version) and bring the F1-C interface into service.

3. **How can a NAS Registration Request be transported over F1-C?**
   - The UE encapsulates the **NAS-5GS Registration Request** inside an **NR RRC** message (`RRCSetupComplete`). The gNB-DU does not terminate RRC or NAS; instead, it wraps that RRC container inside an F1AP **`ULRRCMessageTransfer`** message (Packet #18) and forwards it over SCTP to the gNB-CU, which extracts the NAS payload and forwards it to the AMF over N2 (NGAP).

4. **What is the role of UE Context Setup?**
   - `UEContextSetupRequest` / `UEContextSetupResponse` (Packets #34–35) establishes UE-specific state and radio resources inside the gNB-DU—including allocating SRBs/DRBs, RLC/MAC configuration, and F1-U GTP-U tunnel endpoint parameters (TEIDs)—so the DU can handle user-plane and dedicated control-plane traffic for that UE.

5. **Does F1 Setup alone prove successful UE Registration? Explain.**
   - **No.** F1 Setup only proves that the gNB-DU and gNB-CU have successfully established their internal RAN control-plane interface and brought the cell into service. It happens before any UE even attempts to connect. Proving UE Registration requires observing subsequent RRC connection establishment, NAS authentication/security signaling with the 5GC AMF, UE Context Setup, and PDU Session establishment.

---

## Task 5 — Analyze F1-U (GTP-U / UDP)

In the same `lab4-f1-complete.pcap` capture, applying the `gtp` display filter revealed **85 GTP-U packets** (62.0% of the 137 total packets) exchanged over UDP port `2152` between the gNB-DU (`127.0.0.4`) and gNB-CU (`127.0.0.3`).

### Wireshark Evidence

**GTP-U T-PDU Packet Details (Outer IP, UDP 2152, Message Type, TEID):**

<img width="1512" height="922" alt="GTP-U T-PDU Packet Details" src="https://github.com/user-attachments/assets/10426491-cf5d-4b6c-a1e9-1a55de7c918d" />

**Uplink F1-U Traffic (`gtp && ip.src == 127.0.0.4 && ip.dst == 127.0.0.3`):**

<img width="1512" height="922" alt="Uplink F1-U Traffic" src="https://github.com/user-attachments/assets/3e646bc1-7677-48f0-89be-81453c6d849f" />

**Downlink F1-U Traffic (`gtp && ip.src == 127.0.0.3 && ip.dst == 127.0.0.4`):**

<img width="1512" height="922" alt="Downlink F1-U Traffic" src="https://github.com/user-attachments/assets/e61f09a8-b435-447f-8339-93c1a0a9ec2c" />

### Observed F1-U Summary

| Direction | Endpoints (Outer IP) | UDP Ports | Observed Packets | Observed TEID | Notes |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Uplink (UL)** | `127.0.0.4` (DU) → `127.0.0.3` (CU) | `2152 → 2152` | **45** | `0xd38efa55` (Packet #41) | 40 ICMP Echo Requests (137 B) + 5 background UL packets (e.g., 101 B IPv6 RS/ND frames #41–43) |
| **Downlink (DL)** | `127.0.0.3` (CU) → `127.0.0.4` (DU) | `2152 → 2152` | **40** | `0x8c2847ac` (Packet #45, matching DU log) | 40 ICMP Echo Replies (149 B) corresponding to the `10 + 30` Ping tests |
| **Total F1-U** | `127.0.0.4` ↔ `127.0.0.3` | `2152` | **85** | — | Confirmed bidirectional user-plane traffic |

### Distinguishing F1-C, F1-U, and N3

| Interface | Endpoints | Plane | Transport & Protocol | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **F1-C** | gNB-DU (`127.0.0.4`) ↔ gNB-CU (`127.0.0.3`) | Control Plane | **F1AP over SCTP** (port `38472`) | RAN control signaling (F1 Setup, RRC message transfer, UE Context management) |
| **F1-U** | gNB-DU (`127.0.0.4`) ↔ gNB-CU (`127.0.0.3`) | User Plane | **GTP-U over UDP** (port `2152`) | Transports encapsulated UE user-plane traffic (DRBs) between DU and CU |
| **N3** | gNB-CU ↔ 5GC UPF | User Plane | **GTP-U over UDP** (port `2152`) | Transports encapsulated UE PDU session traffic between the RAN (CU) and the 5G Core UPF |

### Questions (Task 5)

1. **Which transport protocol carries F1-U?**
   - **UDP** (specifically on destination/source port `2152`), carrying **GTP-U** (GPRS Tunneling Protocol – User Plane).

2. **What does a GTP-U TEID identify?**
   - A **TEID (Tunnel Endpoint Identifier)** uniquely identifies a specific unidirectional GTP-U tunnel endpoint at the receiving node—mapping encapsulated user-plane packets to the exact UE and Data Radio Bearer (DRB) / PDU session context (e.g., `0xd38efa55` for UL at the CU, `0x8c2847ac` for DL at the DU).

3. **Which direction is `127.0.0.4 → 127.0.0.3` in this setup?**
   - **Uplink (UL)** — traffic flowing from the **gNB-DU** (`127.0.0.4`) to the **gNB-CU** (`127.0.0.3`).

4. **How would you distinguish F1-U from N3 in a packet capture?**
   - Because both F1-U and N3 use **GTP-U over UDP port 2152**, you cannot rely on the protocol name or port number alone. You must inspect the **outer source and destination IP addresses** (and compare them against the deployment configuration): packets between the **gNB-DU and gNB-CU** (`127.0.0.4` ↔ `127.0.0.3` on `lo`) belong to **F1-U**, whereas packets between the **gNB-CU and the 5GC UPF** (on the 5GC Docker subnet) belong to **N3**.

5. **Why do F1-C and F1-U use different transport protocols?**
   - **F1-C** carries critical control-plane signaling (F1 Setup, RRC, UE Context) where packet loss or out-of-order delivery breaks state synchronization; **SCTP** provides reliable, message-oriented, multi-streamed delivery without head-of-line blocking across independent streams.
   - **F1-U** carries real-time user-plane data where low latency and minimal overhead matter most; **UDP** avoids transport-layer retransmission delays (reliability is already handled by RLC/HARQ on the radio link or end-to-end TCP/application layers inside the tunnel).

---

## Task 6 — Compare Monolithic and Split gNB

### Architecture Comparison

| Feature | Lab 3: Monolithic gNB | Lab 4: CU/DU Split |
| :--- | :--- | :--- |
| **Number of RAN processes** | 1 (`nr-softmodem` monolithic gNB) | 2 (1 `nr-softmodem` for gNB-CU + 1 `nr-softmodem` for gNB-DU) |
| **Location of RRC/PDCP** | Inside the monolithic gNB | Inside the **gNB-CU** |
| **Location of RLC/MAC/PHY** | Inside the monolithic gNB | Inside the **gNB-DU** |
| **F1-C interface present?** | No | Yes (**F1AP over SCTP**, `127.0.0.4` ↔ `127.0.0.3`, port `38472`) |
| **F1-U interface present?** | No | Yes (**GTP-U over UDP**, `127.0.0.4` ↔ `127.0.0.3`, port `2152`) |
| **UE connected to the same 5GC?** | Yes (`oai-cn5g`, AMF `192.168.70.132`) | Yes (`oai-cn5g`, AMF `192.168.70.132`) |
| **End-to-end Ping works?** | Yes (`10.0.0.2` → `192.168.70.135`, 0% loss) | Yes (`10.0.0.2` → `192.168.70.135`, 0% loss) |

### Explanation

Splitting a gNB into a CU and a DU is an internal RAN architectural disaggregation that separates upper non-real-time layers (RRC, SDAP, PDCP) from lower real-time layers (RLC, MAC, PHY), introducing the F1-C and F1-U interfaces between them. From the UE's perspective over the Uu air interface, this split is completely transparent because the DU still transmits the SSB and system information while relaying RRC and NAS signaling to the CU. Consequently, the UE must still execute the exact same PHY/MAC synchronization and random access procedures with the cell before establishing an RRC connection. Likewise, because the 5G Core (AMF, SMF, UPF) and external N2/N3 interfaces remain unchanged, the UE still requires full NAS authentication, 5GC registration, and PDU session establishment to receive an IP address (`oaitun_ue1`) and exchange user-plane data.
