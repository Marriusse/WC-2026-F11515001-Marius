# Lab 3: Deploying a 5G Standalone Network Using OpenAirInterface

## Task 1 — Deploy and Verify the Complete 5G SA Network

The 5G Standalone network (Core, gNB, and nrUE) was successfully deployed using Docker and the OAI software suite.

### Connection Stages Verification
| Stage | Successful? | Evidence from logs |
|---|---|---|
| CN5G initialization | Yes | Docker containers showing "Up (healthy)" in `docker compose ps` |
| gNB NG Setup | Yes | gNB logs showing successful NG Setup response from AMF |
| UE synchronization | Yes | nrUE logs indicating successful synchronization with the gNB cell |
| RRC connection establishment | Yes | UE logs displaying "RRCSetupComplete" and state transitions |
| UE Registration | Yes | AMF logs showing successful registration and UE transitioning to RM-REGISTERED |
| PDU Session Establishment | Yes | SMF/UPF logs and UE logs showing PDU session setup success |
| UE tunnel creation | Yes | Creation of the `oaitun_ue1` interface on the host machine |

<img width="1445" height="877" alt="Capture d’écran, le 2026-11-15 à 23 20 46" src="https://github.com/user-attachments/assets/230d80d4-52eb-403b-92e3-e72de91fd65b" />
<img width="1448" height="878" alt="Capture d’écran, le 2026-11-15 à 23 21 26" src="https://github.com/user-attachments/assets/44a95f8f-f181-40e4-bf14-b261addaadd4" />

### Questions

**1. Does a successful NG Setup guarantee that a UE can register? Explain.**
No. A successful NG Setup only establishes the N2 control-plane interface between the gNB and the AMF. UE registration is a separate Non-Access Stratum (NAS) procedure that involves authentication and subscription verification (IMSI check) which can fail independently of the gNB-AMF connection.

**2. Why can the UE establish an RRC connection but still fail Registration?**
The RRC connection is a Layer 3 radio-level connection established strictly between the UE and the gNB. Registration is a NAS-level procedure handled between the UE and the 5G Core (AMF/AUSF/UDM). The radio link can be perfectly fine, but the Core Network may still reject the UE if its credentials (like the IMSI) are invalid or unauthorized.

**3. What additional procedure is required after Registration before user data can be exchanged?**
After a successful Registration, a PDU Session Establishment procedure is required. This allocates an IP address to the UE and sets up the GTP-U user-plane tunnel between the gNB and the UPF to route actual user data.

---

## Task 2 — Investigate RFsimulator Communication

The RFsimulator replaces the physical radio interface with a software-based TCP connection to exchange IQ samples between the gNB and the nrUE.

### RFsimulator Connection Details
| Item | Observed Value |
|---|---|
| gNB process ID | 4156 |
| nrUE process ID | 7181 |
| RFsimulator transport protocol | TCP |
| RFsimulator port | 4043 |
| TCP connection state | ESTAB (Established) |
| Connected process(es) | `nr-uesoftmodem` and `nr-softmodem` |

<img width="1447" height="879" alt="Capture d’écran, le 2026-11-15 à 23 36 00" src="https://github.com/user-attachments/assets/827e16a9-9458-491f-aab7-4a1251813070" />

### Questions

**1. Why does RFsimulator use a TCP connection instead of a physical antenna?**
RFsimulator is designed to bypass the need for Software Defined Radios (SDRs) and physical antennas. It uses a TCP connection to reliably transmit the digital baseband IQ samples directly between the gNB and UE software processes over the host's loopback network.

**2. Are the TCP packets exchanged by RFsimulator the same as the UE's application-layer IP packets?**
No. The TCP packets exchanged by the RFsimulator contain raw, low-level IQ samples representing the radio waves. The UE's application-layer IP packets (like ICMP ping) are encapsulated and modulated into these IQ samples before being sent over the TCP socket.

**3. Does using RFsimulator mean that the UE and gNB are performing no physical-layer processing? Explain.**
No, the physical-layer (PHY) processing is still fully active. Both the gNB and UE still perform encoding, scrambling, modulation (e.g., QAM), and OFDM symbol generation in software. The RFsimulator simply replaces the final digital-to-analog (DAC) conversion and wireless transmission by sending the resulting digital OFDM baseband signal over TCP.

---

## Task 3 — Verify End-to-End Connectivity

With the 5G network running, E2E user-plane connectivity was verified using ICMP ping requests through the UE's tunnel interface.

### Ping Measurement Results
| Measurement | Observed Value |
|---|---|
| UE IP address | 10.0.0.2 |
| Destination IP | 192.168.70.135 |
| Packets transmitted | 5 |
| Packets received | 5 |
| Packet loss (%) | 0% |
| Minimum RTT (ms) | 163.911 ms |
| Average RTT (ms) | 173.657 ms |
| Maximum RTT (ms) | 185.889 ms |

<img width="1444" height="831" alt="Capture d’écran, le 2026-11-15 à 23 23 38" src="https://github.com/user-attachments/assets/0555a3ad-5380-4ece-a921-9905925671d8" />

### Questions

**1. Why must Ping use the `oaitun_ue1` interface in this experiment?**
Ping must be forced to use the `oaitun_ue1` interface so the ICMP traffic is encapsulated by the UE software into the 5G user-plane protocols (PDCP/RLC/MAC/PHY) and routed through the simulated 5G network to the Core, rather than going through the host machine's default Ethernet/Wi-Fi network.

**2. Does successful Registration guarantee successful user-plane connectivity?**
No, successful registration only handles the control plane (mobility and signaling). User-plane connectivity requires the successful establishment of a PDU Session and the correct configuration of the GTP tunnels and routing tables in the UPF.

**3. What does a successful Ping prove, and what does it not prove about the simulated 5G system?**
A successful ping proves that end-to-end IP connectivity through the 5G user plane is working correctly. It demonstrates that the PDU session, GTP-U tunnel, IP routing, and the relevant user-plane protocol stack are operational. However, it does not prove real-world RF performance, physical antenna behavior, or radio propagation characteristics.

## Task 4 — Investigate the Effect of Packet Size

The impact of different ICMP payload sizes on the network Round-Trip Time (RTT) was measured using the established 5G SA connection.

### Measurement Results
| Scenario | ICMP Payload | Avg. RTT (ms) | Max. RTT (ms) | Packet Loss (%) |
|---|---:|---:|---:|---:|
| A | 56 bytes | 120.806 ms | 184.690 ms | 0% |
| B | 512 bytes | 152.623 ms | 239.295 ms | 0% |
| C | 1200 bytes | 134.307 ms | 181.698 ms | 0% |

**RTT Difference Calculation:**
```text
RTT Difference = Average RTT(C) - Average RTT(A)
RTT Difference = 134.307 ms - 120.806 ms = 13.501 ms
```

### Questions

**1. Which scenario has the highest average RTT?**
In this specific experimental run, Scenario B (512 bytes) had the highest average RTT at 152.623 ms.

**2. Does increasing packet size always increase RTT in your measurements?**
No. While Scenario B (512 bytes) increased the RTT compared to Scenario A (56 bytes), Scenario C (1200 bytes) actually showed a lower average RTT (134.307 ms) than Scenario B. This shows that in a software-simulated environment, the correlation between packet size and latency is not always perfectly linear.

**3. Why might a larger packet require more transmission or processing resources?**
Larger packets take longer to transmit over the air interface because they occupy more Resource Blocks (RBs) and require more OFDM symbols. Additionally, larger packets require more intensive physical-layer software processing (such as CRC calculation, channel coding, and modulation) before transmission.

**4. What factors other than packet size could affect RTT in a software-based 5G network?**
In a software-based network, RTT is heavily influenced by the host machine's CPU load, OS thread scheduling, context switching, and the Docker virtual network overhead. Fluctuations in host CPU availability often cause random latency spikes that overshadow the actual transmission delays.

**5. Can these results be interpreted as real over-the-air 5G latency? Explain.**
No. These measurements cannot be interpreted as real over-the-air 5G latency. The observed RTT includes software processing, host scheduling, and TCP communication overhead introduced by RFsimulator. Real 5G latency depends on the deployment, radio conditions, network architecture, and service requirements. In particular, URLLC latency targets refer to specific radio-interface performance requirements and should not be directly compared with these end-to-end RTT measurements.

## Task 5 — Diagnose a Controlled UE Registration Failure

The original UE command was saved, then only the IMSI was changed:
```bash
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 \
  --uicc0.imsi 001010000000099 --rfsim --rfsimulator.serveraddr 127.0.0.1
```
CN5G and gNB were kept running, and the subscriber database was not modified.

### Controlled Failure Observation

| Item | Observation |
|---|---|
| Modified parameter | `--uicc0.imsi` changed from `001010000000001` to `001010000000099` (only change) |
| UE synchronized with gNB? | Yes: the RFsimulator TCP connection was established and the UE PHY started |
| RRC connection established? | Yes (inferred): the AMF created a UE context for this SUPI (RAN UE NGAP ID 0x03, AMF UE NGAP ID 0x04), which requires the gNB to have forwarded the Initial UE Message after RRC setup |
| Registration successful? | No: the UE stays `5GMM-DEREGISTERED` with an empty GUTI |
| Error or rejection evidence | AUSF: `Resource corresponding to User imsi-001010000000099 not found`, `CONTEXT_NOT_FOUND`, `Send Auth response to SEAF (Code 404)`. UDR: `Authentication Subscription no data` and HTTP 404 for `ueid='001010000000099'` |
| Likely cause | IMSI 001010000000099 is not provisioned in the subscriber database (UDR/MySQL), so the authentication data cannot be retrieved and registration fails at the NAS authentication step |

Core-network evidence (all at 17:14:44):
```text
oai-ausf | [ausf_app] [info] supiOrSuci imsi-001010000000099
oai-udm  | [udm_ueau] [debug] SUPI imsi-001010000000099 ...
oai-udr  | [udr_db] [info] MySQL Query: SELECT * FROM AuthenticationSubscription WHERE ueid='001010000000099'
oai-udr  | [udr_db] [error] Authentication Subscription no data
oai-udr  | [udr_server] [info] HTTP Response code 404
oai-ausf | [ausf_app] [error] Resource corresponding to User imsi-001010000000099 not found
oai-ausf | {"cause":"CONTEXT_NOT_FOUND", ... "status":404}
```

AMF state (gNB connected, UE not registered):
```text
Index | Status    | Global Id | gNB Name
1     | Connected | 0x0E00    | gNB-OAI

Index | 5GMM State        | IMSI/SUPI       | GUTI | RAN UE NGAP ID | AMF UE NGAP ID
2     | 5GMM-DEREGISTERED | 001010000000099 | (empty) | 0x03        | 0x04
```

<img width="1447" height="878" alt="Capture d’écran, le 2026-11-16 à 00 26 43" src="https://github.com/user-attachments/assets/f696d747-0399-4017-9350-d586030586b4" />
<img width="1449" height="878" alt="Capture d’écran, le 2026-11-16 à 00 27 03" src="https://github.com/user-attachments/assets/59da7364-a2db-4bfd-81ba-905f188eb7e3" />
<img width="1446" height="484" alt="Capture d’écran, le 2026-11-16 à 00 27 54" src="https://github.com/user-attachments/assets/f8aea3ac-2a7d-4577-8b0f-87f097b3ed90" />

### AMF Logs: gNBs' Information
| Index | Status | Global Id | gNB Name | PLMN |
|---|---|---|---|---|
| 1 | Connected | 0x0E00 | gNB-OAI | 001,01 |

### AMF Logs: UEs' Information
| Index | 5GMM State | IMSI/SUPI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |
|---|---|---|---|---|---|---|---|
| 1 | 5GMM-DEREGISTERED | 001010000000001 | 00101010041390221624 | 0x01 | 0x01 | 001,01 | 0000e014e |
| 2 | 5GMM-DEREGISTERED | 001010000000099 | | 0x03 | 0x04 | 001,01 | 0000e014e |

### Recovery

The nrUE was stopped (Ctrl+C) and restarted with `--uicc0.imsi 001010000000001`. The UE registered again, `oaitun_ue1` was recreated and ping succeeded:

<img width="1446" height="877" alt="Capture d’écran, le 2026-11-16 à 00 38 26" src="https://github.com/user-attachments/assets/799b343c-d0c9-46c1-9da5-9b58a7434812" />
<img width="1447" height="878" alt="Capture d’écran, le 2026-11-16 à 00 40 57" src="https://github.com/user-attachments/assets/bedc77d9-59e6-4022-add1-f3f519e9927e" />

<img width="1435" height="373" alt="Capture d’écran, le 2026-11-16 à 00 41 50" src="https://github.com/user-attachments/assets/036665c9-211d-4ad1-ac32-69b578f897fa" />

### Evidence of Recovery: AMF Logs (UE Status)
| Index | 5GMM State | IMSI/SUPI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |
|---|---|---|---|---|---|---|---|
| 1 | 5GMM-REGISTERED | 001010000000001 | 00101010041272603560 | 0x04 | 0x05 | 001,01 | 0000e014e |
| 2 | 5GMM-DEREGISTERED | 001010000000099 | | 0x03 | 0x04 | 001,01 | 0000e014e |
<img width="1446" height="371" alt="Capture d’écran, le 2026-11-16 à 00 44 10" src="https://github.com/user-attachments/assets/acaef94e-589c-4a44-89c9-86416e69dfe1" />

### Questions

**1. Why can the UE potentially synchronize with the gNB even when the IMSI is incorrect?**
Synchronization and random access depend on radio parameters (band, center frequency, SSB, bandwidth), not on the subscriber identity. The IMSI is only used at the NAS layer, where the core checks it during authentication. In my test the gNB stayed `Connected` and the UE even reached the AMF, but it could not be authenticated.

**2. Which network function is involved in UE authentication?**
The AMF triggers authentication and relays the request to the AUSF, which asks the UDM for authentication data. The UDM retrieves the subscriber's credentials from the UDR (backed by MySQL). In my logs the chain AMF → AUSF → UDM → UDR ended with "no data" in the UDR and a 404 returned by the AUSF.

**3. Why does changing an IMSI affect Registration but not the RFsimulator TCP connection itself?**
The RFsimulator TCP connection only transports IQ samples and knows nothing about the identity of the UE. The IMSI is a NAS-level identity checked by the core during authentication. Therefore the physical link stays up while the registration fails.

**4. What evidence proves that restoring the correct configuration solved the problem?**
After restarting the UE with IMSI `001010000000001`, the AMF showed this IMSI in `5GMM-REGISTERED`, the PDU session was established, `oaitun_ue1` was recreated with an IP address, and ping to 192.168.70.135 succeeded. The authentication errors (`no data`, 404) no longer appeared for this IMSI.

---

## Task 7 — Explain the Relationship Between Simulation and Real 5G

### Simulation vs. Physical 5G System

| Feature | OAI RFsimulator | Physical 5G System |
|---|---|---|
| gNB and UE protocol stack | Fully executed in software on the host CPU (PHY, MAC, RLC, PDCP, RRC, NAS) | Same protocols, with PHY often accelerated by dedicated hardware (FPGA/ASIC) in commercial equipment |
| OFDM signal processing | Done in software, producing IQ samples | Done in software and/or hardware, then converted by DAC/ADC and RF chain |
| Physical antennas | None: replaced by a TCP connection carrying IQ samples | Real Tx/Rx antennas (possibly MIMO/beamforming) |
| RF hardware impairments | None by default (ideal front-end) | Noise, non-linearity, frequency offset, phase noise |
| Wireless propagation effects | None by default; optional channel models (e.g. AWGN, fading) can be enabled in RFsimulator | Multipath, shadowing, Doppler, path loss, interference |
| 5G Core connectivity | Standard IP/SCTP/GTP-U over Docker virtual networks | Same protocols over physical fiber/Ethernet backhaul |

### Questions

**1. Which parts of the 5G system remain functional when physical RF hardware is replaced with RFsimulator?**
Everything except the RF front-end: the PHY processing, MAC, RLC, PDCP, RRC and NAS layers all still run, as does the whole 5G Core (AMF, SMF, UPF, ...). The digital PHY processing remains active, including channel coding, modulation, and OFDM processing. However, the physical RF front-end (DAC/ADC, analog RF chain, and antennas) is absent. Instead, RFsimulator exchanges digital IQ samples between the gNB and UE through a TCP connection.

**2. Why is RFsimulator useful for testing the 5G protocol stack?**
It allows complete gNB, UE and core tests on a single computer without SDRs, antennas or RF-shielded rooms. Experiments are reproducible and safe, and failures such as the unprovisioned IMSI in Task 5 can be injected and debugged through logs.

**3. Why might latency and throughput differ between RFsimulator and a real 5G deployment?**
RFsimulator is not real time: all processing is done by the host CPU, which also runs the core, and samples are exchanged over TCP. Real deployments use optimized and often hardware-accelerated baseband processing with deterministic timing. Real throughput also depends on radio conditions, which are absent here.

**4. What would need to change if you wanted to test a physical UE instead of OAI nrUE?**
The gNB would need a real RF front-end (e.g. a USRP) instead of `--rfsim`, with antennas, correct frequency/gain settings and a band supported by the phone (n78). The phone would need a SIM programmed with the IMSI, Ki and OPc provisioned in the core database, and an APN/DNN matching the core configuration (`oai`).

---

## Final Analysis

**1. Describe the complete process from starting the OAI 5GC to successful user-plane connectivity.**
I first started the OAI CN5G containers with Docker and checked that all were healthy. I then launched the gNB, which established the NGAP/N2 connection with the AMF (NG Setup). Next I started the nrUE, which connected to the gNB through RFsimulator, synchronized with the cell, completed the RRC setup and sent a Registration Request. The core authenticated the UE (AMF, AUSF, UDM, UDR) and accepted the registration, after which a PDU Session was established through the SMF and UPF. The UE then got the IP address 10.0.0.2 on `oaitun_ue1`, and ping to the external DN succeeded.

**2. Explain the difference between an RFsimulator TCP connection and a 5G PDU Session.**
The RFsimulator TCP connection (port 4043, gNB as server and nrUE as client) is only the emulated radio link, carrying raw IQ samples produced by the PHY layer. A PDU Session is a logical connection created by NAS signaling with the AMF and SMF, in which the UPF provides a GTP-U tunnel and the UE receives an IP address. The TCP connection exists as soon as both processes run, even when registration fails (Task 5), whereas the PDU session only exists after a successful registration. Application packets such as ping travel inside the PDU session, which is itself carried as IQ samples over the TCP connection.

**3. Based on your measurements, how did packet size affect RTT and packet loss?**
Packet loss was 0% in all three scenarios. The average RTT was 120.806 ms for 56 bytes, 152.623 ms for 512 bytes and 134.307 ms for 1200 bytes, so it did not increase monotonically with packet size. The C−A difference of 13.501 ms is of the same order as the measured variability. This suggests that CPU load, scheduling and TDD timing likely dominate over the payload size in this setup.

**4. Describe the registration failure you introduced and explain how you diagnosed and recovered from it.**
I changed only the UE IMSI to `001010000000099`, which is not provisioned in the core subscriber database. The UE synchronized and reached the AMF, which created a UE context with this SUPI, but the UE stayed `5GMM-DEREGISTERED` with no GUTI while the gNB remained `Connected`. In the core logs, the AUSF requested authentication data from the UDM, the UDR ran a `SELECT` on `AuthenticationSubscription` for this IMSI, found no data and returned HTTP 404, and the AUSF answered `CONTEXT_NOT_FOUND`. I recovered by stopping the UE and restarting it with `001010000000001`, after which the UE registered, `oaitun_ue1` was recreated and ping succeeded.

**5. Identify two limitations of this simulated environment and explain how they might affect experimental results.**
First, all PHY processing and the core run on one general-purpose host, with non-real-time sample exchange over TCP. This makes RTTs (>100 ms) much higher and more variable than in a real network, so latency results cannot be transferred to real deployments. Second, by default there is no propagation channel, interference or RF impairment, so the link is ideal. This hides effects such as fading, retransmissions and packet loss, and makes the observed 0% loss and stable connection overly optimistic.

