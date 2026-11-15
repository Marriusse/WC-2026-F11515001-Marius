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
A successful ping proves that the end-to-end IP routing, NAS signaling, and software protocol stack (PDCP/RLC/MAC) are functioning correctly. However, because it uses RFsimulator, it does not prove real-world physical layer (PHY) performance, as it bypasses actual radio transmission, interference, signal fading, and over-the-air scheduling.

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
No. Real over-the-air 5G URLLC latency is typically in the range of 1 to 10 milliseconds. The RTTs observed here (>100 ms) are artificially inflated due to the heavy computational burden of processing the PHY layer purely in software on a standard x86 CPU, rather than using dedicated hardware (FPGAs or ASICs) as in real telecom equipment.

## Task 5 — Diagnose a Controlled UE Registration Failure

A controlled failure was introduced by modifying the UE's IMSI to an unauthorized value (`001010000000099`) to observe the network's rejection behavior. 

### Controlled Failure Observation
| Item | Observation |
|---|---|
| Modified parameter | `uicc0.imsi` changed from `...01` to `...99` |
| UE synchronized with gNB? | Yes (RFsimulator TCP connection and PHY initialization started successfully) |
| RRC connection established? | Attempted |
| Registration successful? | No |
| Error or rejection evidence | The AMF logs did not show a NAS reject message. Instead, the AMF logs showed the gNB status as `Disconnected` (`1 | Disconnected | 0x0E00 | gNB-OAI`), and the UE list remained empty. |
| Likely cause | The IMSI `001010000000099` is not provisioned in the core network (UDM/MySQL). However, the absence of a NAS reject log indicates the connection dropped or stalled at the gNB/radio level before the AMF could process the authentication failure, likely due to a timeout or the gNB disconnecting during the process. |

<img width="1447" height="879" alt="Capture d’écran, le 2026-11-15 à 23 54 05" src="https://github.com/user-attachments/assets/a281073a-0823-4089-b79a-5c9bb2c99498" />
<img width="1447" height="878" alt="Capture d’écran, le 2026-11-15 à 23 54 57" src="https://github.com/user-attachments/assets/c7f7aa7a-a0eb-44be-bc60-da559dc84627" />

### Questions

**1. Why can the UE potentially synchronize with the gNB even when the IMSI is incorrect?**
Synchronization and physical layer attachment (PHY/MAC) rely on radio cell parameters (like band, frequencies, and SSB broadcast), not on the subscriber's identity. The IMSI is only checked later at the NAS layer by the Core Network.

**2. Which network function is involved in UE authentication?**
The Authentication Server Function (AUSF) and the Unified Data Management (UDM) are the 5G Core network functions responsible for verifying the UE's credentials.

**3. Why does changing an IMSI affect Registration but not the RFsimulator TCP connection itself?**
The RFsimulator operates at the lowest level (Layer 1/Physical), simply transporting raw IQ samples over a TCP socket to emulate radio waves. The IMSI operates at Layer 3 (NAS) for logical authentication. The physical link does not care about the logical identity of the data it carries.

**4. What evidence proves that restoring the correct configuration solved the problem?**
After restoring the original IMSI (`001010000000001`) and restarting the nrUE, the `oaitun_ue1` interface was successfully recreated on the host machine, and ICMP ping requests to the external DN (`192.168.70.135`) were successful again.

---

## Task 7 — Explain the Relationship Between Simulation and Real 5G

Even without physical antennas, the OpenAirInterface software performs real 5G NR protocol processing.

### Simulation vs. Physical 5G System
| Feature | OAI RFsimulator | Physical 5G System |
|---|---|---|
| gNB and UE protocol stack | Fully executed in software on the host CPU. | Executed in software and hardware (baseband units). |
| OFDM signal processing | Mathematically calculated in software (IQ samples). | Processed by dedicated ASICs/FPGAs and radio heads. |
| Physical antennas | Replaced by a TCP connection exchanging data packets. | Actual Tx/Rx MIMO antennas transmitting radio waves. |
| RF hardware impairments | Ideal mathematically simulated environment. | Real hardware noise, thermal distortion, and interference. |
| Wireless propagation effects | Simple or no pathloss models configured by default. | Multipath fading, shadowing, and Doppler effects. |
| 5G Core connectivity | Standard IP traffic over virtual Docker networks. | Standard IP traffic over physical fiber/Ethernet backhaul. |

### Questions

**1. Which parts of the 5G system remain functional when physical RF hardware is replaced with RFsimulator?**
The entire protocol stack above the physical transmission medium remains fully functional. This includes the MAC, RLC, PDCP, RRC, and NAS layers, as well as the complete 5G Core Network.

**2. Why is RFsimulator useful for testing the 5G protocol stack?**
It allows developers and students to deploy, test, and debug L2/L3 protocols and core network integration purely in software. It eliminates the need for expensive Software Defined Radios (SDRs) and shielded RF rooms.

**3. Why might latency and throughput differ between RFsimulator and a real 5G deployment?**
In RFsimulator, the host CPU performs all the heavy Physical-layer digital signal processing (DSP) sequentially, which artificially inflates latency. Real 5G deployments use dedicated hardware accelerators (FPGAs/ASICs) that process baseband signals with extremely low latency and high throughput.

**4. What would need to change if you wanted to test a physical UE instead of OAI nrUE?**
You would need to replace the RFsimulator on the gNB side with a physical Software Defined Radio (like a USRP B210), attach proper antennas, configure the RF frequencies and gains, and use a commercial 5G smartphone with a SIM card programmed with the Core Network's credentials.

---

## Final Analysis

**1. Describe the complete process from starting the OAI 5GC to successful user-plane connectivity.**
The deployment begins by initializing the 5G Core containers (AMF, SMF, UPF, etc.) to handle control and user planes. Next, the gNB is launched and connects to the AMF via the NGAP interface. Finally, the nrUE establishes a physical-layer sync via RFsimulator, completes an RRC connection, and authenticates via NAS, prompting the SMF and UPF to establish a PDU session and create a virtual tunnel interface for user data routing.

**2. Explain the difference between an RFsimulator TCP connection and a 5G PDU Session.**
The RFsimulator TCP connection operates at the physical layer, acting as a virtual cable that carries raw IQ radio samples between the gNB and UE processes. Conversely, a 5G PDU session is a high-level, logical IP connection managed by the Core Network (SMF/UPF) that allows the UE's applications to communicate with external data networks like the internet.

**3. Based on your measurements, how did packet size affect RTT and packet loss?**
Packet loss remained at 0% across all tests, indicating a highly stable logical connection. However, packet size did not have a strictly linear effect on RTT, as the 512-byte packets exhibited a higher average RTT than the 1200-byte packets. This non-linear latency suggests that software processing overhead and CPU scheduling on the host machine dominate the actual transmission delays.

**4. Describe the registration failure you introduced and explain how you diagnosed and recovered from it.**
I introduced a registration failure by intentionally changing the UE's IMSI to an unauthorized value (`001010000000099`). I diagnosed the issue by checking the AMF logs, which revealed that no NAS rejection was explicitly logged because the gNB was registered as `Disconnected`. This proved the connection stalled before the NAS request could be properly processed by the core. I successfully recovered the network by stopping the UE process, ensuring the gNB was running, and restarting the UE with the correct, provisioned IMSI (`...01`), which restored full ping connectivity.

**5. Identify two limitations of this simulated environment and explain how they might affect experimental results.**
First, using the host's general-purpose CPU for physical-layer processing creates an artificial computational bottleneck, resulting in RTT measurements (e.g., >100 ms) that are much higher than real 5G URLLC latency (1-10 ms). Second, the absence of real-world wireless impairments (like multipath fading, obstacles, and interference) means the network behaves more ideally than a physical deployment, masking potential packet loss or retransmission behaviors that occur in the real world.
