### 3. Task 1 - Confirm the Traffic Direction

| Scenario | Reverse value | Direction |
| --- | --- | --- |
| A | 1 | Downlink |
| B | 0 | Uplink |
| C | 0 | Uplink |

**Question:** Does the traffic direction shown in the JSON match the scenario description? Show the proof.

**Answer:**
Yes, the traffic direction in the JSON files matches the scenario descriptions perfectly. The proof is located in the `test_start` block of each file:
* `scenario-a-downlink.json` contains `"reverse": 1`, confirming Downlink traffic.
* `scenario-b-uplink.json` and `scenario-c-uplink.json` contain `"reverse": 0`, confirming Uplink traffic.

### 4. Task 2 - Compare Throughput

| Scenario | Direction | Throughput (Mbps) |
| --- | --- | ---: |
| A | Downlink | 90.65 |
| B | Uplink | 109.12 |
| C | Uplink | 124.42 |

Improvement (%) = (124.42 - 109.12) / 109.12 * 100 = 14.02%

**Answer:**
1. The downlink throughput in Scenario A is approximately 90.65 Mbps.
2. The uplink throughput in Scenario B is approximately 109.12 Mbps.
3. The uplink throughput is higher in Scenario C.
4. Scenario C improves over Scenario B by approximately 14.02%.

### 5. Task 3 - Check the Trade-off

| Scenario | Throughput (Mbps) | Packet loss (%) | Jitter (ms) |
| --- | ---: | ---: | ---: |
| B | 109.12 | 0.24 | 0.12 |
| C | 124.42 | 2.13 | 0.16 |

**Answer:**
1. Yes, Scenario C has a higher uplink throughput than Scenario B.
2. No, Scenario C does not have lower packet loss than B; it has a significantly higher packet loss.
3. Compared with Scenario B, Scenario C has higher throughput (124.42 vs 109.12 Mbps), higher packet loss (2.13% vs 0.24%), and slightly higher jitter (0.16 ms vs 0.12 ms). These results suggest a trade-off: in this measurement, the throughput gain coincides with more packet loss and a small increase in jitter. A single 5-second run does not show what causes this, so the result should be read as an observation and not as proof that more UL resources always increase packet loss.
   
### 6. Task 4 - Read SIB1 and Draw the TDD Configuration

#### 4.3 Record the TDD Parameters

| Item | DL-heavy: A/B | Balanced: C |
| :--- | :--- | :--- |
| Configuration capture filename | tdd-dl-heavy-sib1.pcapng | tdd-balanced-sib1.pcapng |
| SIB1 packet number | 2 | 2 |
| `referenceSubcarrierSpacing` (kHz) | 30 | 30 |
| `dl-UL-TransmissionPeriodicity` (ms) | 5 | 2.5 |
| `nrofDownlinkSlots` | 7 | 2 |
| `nrofDownlinkSymbols` | 6 | 6 |
| `nrofUplinkSlots` | 2 | 2 |
| `nrofUplinkSymbols` | 4 | 4 |
| Is `pattern2` present? | No | No |

<img width="1512" height="950" alt="Capture d’écran, le 2026-11-08 à 22 51 54" src="https://github.com/user-attachments/assets/fb7342a5-71ad-455f-bd3f-f9387f261a31" />
<img width="1512" height="950" alt="Capture d’écran, le 2026-11-08 à 22 54 04" src="https://github.com/user-attachments/assets/4284b7de-46f0-4ebc-a28e-663acee818aa" />

### 4.4 Draw the TDD Time Allocation

| TDD setting | Repeating duration (ms) | Total reference slots | DL symbols | UL symbols | Flexible symbols | Configured UL symbol share (%) |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 10 | 104 | 32 | 4 | 22.86 |
| Balanced | 2.5 | 5 | 34 | 32 | 4 | 45.71 |

| TDD setting | Repeating duration (ms) | Repetitions in 5 ms | DL symbols in 5 ms | UL symbols in 5 ms | Flexible symbols in 5 ms |
| :--- | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 1 | 104 | 32 | 4 |
| Balanced | 2.5 | 2 | 68 | 64 | 8 |

**Question:** Can two patterns have the same UL symbol count per period but provide different numbers of UL symbols per unit time? Explain using your two tables.

**Answer:**
Yes. Both configurations provide exactly 32 UL symbols per repeating period. However, because the Balanced configuration repeats every 2.5 ms (twice as fast as the 5 ms DL-heavy configuration), it fits two full repetitions within a 5 ms interval. This effectively doubles the uplink resources per unit time (64 UL symbols in 5 ms for Balanced vs. 32 UL symbols in 5 ms for DL-heavy).

#### TDD Diagram: DL-heavy (5 ms period)
*Legend: D = Downlink, U = Uplink, F = Flexible. 1 Slot = 0.5 ms (14 symbols)*

| Slot (Time) | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** (0.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **1** (0.5 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **2** (1.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **3** (1.5 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **4** (2.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **5** (2.5 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **6** (3.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **7** (3.5 ms) | D | D | D | D | D | D | F | F | F | F | U | U | U | U |
| **8** (4.0 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
| **9** (4.5 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
*(End of repeating period at 5.0 ms)*

<br>

#### TDD Diagram: Balanced (2.5 ms period, repeated twice in 5 ms)
*Legend: D = Downlink, U = Uplink, F = Flexible. 1 Slot = 0.5 ms (14 symbols)*

| Slot (Time) | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** (0.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **1** (0.5 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **2** (1.0 ms) | D | D | D | D | D | D | F | F | F | F | U | U | U | U |
| **3** (1.5 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
| **4** (2.0 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
| *(Repeat at 2.5 ms)*| | | | | | | | | | | | | | |
| **5** (2.5 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **6** (3.0 ms) | D | D | D | D | D | D | D | D | D | D | D | D | D | D |
| **7** (3.5 ms) | D | D | D | D | D | D | F | F | F | F | U | U | U | U |
| **8** (4.0 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
| **9** (4.5 ms) | U | U | U | U | U | U | U | U | U | U | U | U | U | U |
*(End of 2nd repeating period at 5.0 ms)*

### 4.5 Connect SIB1 to the Performance Results

| Scenario | Reference TDD setting | JSON traffic direction | Configured UL symbol share (%) | Throughput (Mbps) | Packet loss (%) |
| :--- | :--- | :--- | ---: | ---: | ---: |
| A | DL-heavy | Downlink | 22.86 | 90.65 | 0.44 |
| B | DL-heavy | Uplink | 22.86 | 109.12 | 0.24 |
| C | Balanced | Uplink | 45.71 | 124.42 | 2.13 |

**Answer:**

1. **Which decoded TDD parameters differ between DL-heavy and balanced?**
   The `dl-UL-TransmissionPeriodicity` (5 ms vs 2.5 ms) and the `nrofDownlinkSlots` (7 vs 2) differ between the two configurations. 

2. **Which configuration provides more configured UL symbols per unit time? Support your answer with the diagram and calculated share.**
   The Balanced configuration provides more configured UL symbols per unit time. As shown in our previous 5 ms interval calculation, the Balanced setting provides 64 UL symbols (45.71% share), whereas the DL-heavy setting only provides 32 UL symbols (22.86% share) in the same time frame.

3. **How does that difference help explain the B/C throughput comparison?**
   The Balanced configuration offers 64 configured UL symbols per 5 ms, compared with 32 for the DL-heavy configuration, so Scenario C has twice as many configured uplink opportunities as Scenario B. This is consistent with the higher uplink throughput measured in C (109.12 → 124.42 Mbps, +14.02%). However, the throughput gain is far from proportional to the doubling of UL resources, because scheduling, radio conditions, overhead, offered traffic and other radio settings also affect the result. SIB1 does not show the actual slot-by-slot scheduling, so the TDD pattern may contribute to the difference but cannot be established as its only cause.
   
4. **Why can A and B use the same SIB1 TDD configuration even though their traffic directions differ?**
   According to the scenario mapping supplied by the instructor, A and B both use the DL-heavy setting. The TDD configuration in SIB1 is a common, cell-wide broadcast that describes DL, UL and flexible time resources. It does not depend on the direction of a particular iperf3 test. A and B differ only in the direction of the traffic sent over that same configuration (downlink with `reverse: 1` for A, uplink with `reverse: 0` for B).
   
### 7. Task 5 - Is the TDD Setting Suitable?

| Scenario | TDD setting | Traffic direction | Is the setting suitable for this direction? Why? |
| :--- | :--- | :--- | :--- |
| A | DL-heavy | Downlink | **Yes.** A DL-heavy setting is suitable for downlink traffic. More DL resources provide more downlink transmission opportunities. |
| B | DL-heavy | Uplink | **Less suitable.** A DL-heavy setting provides fewer UL resources (32 configured UL symbols per 5 ms, 22.86% share), so it may limit heavy uplink traffic. B still reaches 109.12 Mbps, so the limitation is a reduced number of uplink opportunities, not a blocked uplink. |
| C | Balanced | Uplink | **Yes.** A balanced setting gives uplink traffic more transmission opportunities than the DL-heavy setting in this lab. |

**Question:** Why might Scenario C achieve better uplink throughput than Scenario B? Refer to your SIB1 diagrams and measured results.

**Answer:**
Scenario C may achieve better uplink throughput than Scenario B because its Balanced TDD configuration provides more configured uplink opportunities: 64 UL symbols per 5 ms (45.71% share), compared with 32 UL symbols per 5 ms (22.86% share) for the DL-heavy setting in Scenario B, as shown in the SIB1 diagrams. This is consistent with the measured improvement from 109.12 Mbps to 124.42 Mbps (+14.02%). However, the increase is not proportional to the doubling of UL resources, and other factors (scheduling, radio conditions, overhead, offered traffic) may also play a role. The SIB1 captures were recorded separately, so they are reference settings and not independent proof of what was broadcast during each iperf3 run.

### 8. Task 6 - Find GTP-U Information

| Item | Value |
| :--- | :--- |
| Scenario capture filename | scenario-a-dl-heavy-downlink.pcapng |
| Packet number | 1831 |
| Outer source IP | 192.168.70.129 |
| Outer destination IP | 192.168.70.134 |
| Inner source IP | 10.0.0.6 |
| Inner destination IP | 192.168.70.135 |
| TEID | 0x00000011 (17) |

<img width="1512" height="951" alt="Capture d’écran, le 2026-11-08 à 23 27 39" src="https://github.com/user-attachments/assets/ea60bf2a-a780-4ded-8338-047f061a5359" />

### 9. Task 7 - TCP Retransmissions

| Scenario | Displayed TCP retransmissions |
| :--- | ---: |
| A | 14 |
| B | 12 |
| C | 12 |

<img width="1512" height="950" alt="Capture d’écran, le 2026-11-08 à 23 26 43" src="https://github.com/user-attachments/assets/65607bfd-21f7-4dbb-a7a5-0f09c90e2889" />
<img width="1512" height="949" alt="Capture d’écran, le 2026-11-08 à 23 26 26" src="https://github.com/user-attachments/assets/e536192f-1215-4652-b292-b4b252c997ad" />
<img width="1512" height="947" alt="Capture d’écran, le 2026-11-08 à 23 26 02" src="https://github.com/user-attachments/assets/5e3c3120-50b9-4aea-a6db-a4d39b83cd13" />


**Question:** Do these TCP retransmissions directly represent lost UDP performance packets?

**Answer:**
No. The iperf3 performance traffic is UDP, whereas the TCP packets belong to the small iperf3 control connection used to set up and manage the test, so a TCP retransmission is not a lost UDP data packet. In addition, the count displayed by Wireshark with `tcp.analysis.retransmission` is not, by itself, proof of loss, even on the control channel: some of the packets flagged as retransmissions (for example packets 1831 and 1832 in Scenario A, which carry the same SYN about 12 µs apart) look like the same packet observed twice in the capture, not a real retransmission.

### 10. Task 8 - RTT and Packet Interval

#### TCP RTT
| Scenario | Packet number | Example TCP ACK RTT |
| :--- | ---: | :--- |
| A | 92583 | 1.782000 milliseconds |

#### Packet Interval
| Scenario | Display filter | Packet numbers | Time interval |
| :--- | :--- | :--- | :--- |
| A | `udp.port == 5201` | 92580 and 92582 | 70.000 microseconds |

<img width="1512" height="950" alt="Capture d’écran, le 2026-11-08 à 23 38 14" src="https://github.com/user-attachments/assets/e1428d5d-62a8-45f6-9000-6313e51e02fb" />
<img width="1512" height="949" alt="Capture d’écran, le 2026-11-08 à 23 38 48" src="https://github.com/user-attachments/assets/d56ccefc-520e-473d-b64a-8e32e104710b" />

---

### 11. Final Questions

1. **Does Scenario A use a TDD setting that matches its traffic direction?**
   Yes. Scenario A uses a DL-heavy TDD setting which provides the majority of the time slot resources to the downlink, perfectly matching its Downlink traffic direction.

2. **Why is Scenario B expected to have limited uplink resources? Cite a parameter or count from the DL-heavy SIB1 reference configuration.**
   Scenario B uses the DL-heavy setting for Uplink traffic. It is expected to have limited uplink resources because the configuration only allocates 2 full uplink slots (`nrofUplinkSlots: 2`) and 4 uplink symbols per 5 ms period, resulting in only 22.86% of the symbols being available for UL.

3. **How much does uplink throughput change from B to C?**
   The uplink throughput increases by approximately 14.02%, jumping from 109.12 Mbps in Scenario B to 124.42 Mbps in Scenario C.

4. **What negative result, if any, is visible in C? Support your answer with packet loss or jitter values.**
   The negative result in Scenario C is a higher packet loss: it rises from 0.24% in B to 2.13% in C. Jitter also increases slightly, from 0.12 ms to 0.16 ms. These values show that, in this measurement, the higher throughput comes with lower reliability, even though the jitter change is small.
   
5. **Why can increasing UL resources improve uplink throughput?**
   Increasing UL resources provides the UE with more transmission time slots and symbols within a given timeframe. This directly increases the maximum amount of data the device can push to the network per second.

6. **Is adding more UL resources always the best solution? Explain using throughput and packet loss.**
   No. In Scenario C, adding UL resources increased throughput by 14.02% (109.12 → 124.42 Mbps), but packet loss also rose from 0.24% to 2.13%. More UL resources can therefore provide more uplink opportunities without guaranteeing a better overall result. With a single test, the cause of the higher loss is not established (it could involve the sending rate, scheduling, or radio conditions), so the trade-off between throughput and reliability should be evaluated with further tests.
   
7. **Suggest one simple follow-up test that could improve throughput without producing too much packet loss.**
   A simple follow-up test would be to run the iperf3 test in Scenario C with a manually tuned target bandwidth limit (e.g., using the `-b` flag). By finding the optimal sending rate just below the network's maximum processing capacity, we could maintain high throughput while keeping packet loss near 0%.

8. **What can the configuration captures tell you that the iperf3 JSON cannot? Why are separately recorded configuration captures not direct proof of the configuration during an earlier test?**
   The configuration captures show the broadcast SIB1 TDD parameters (period, number of DL/UL slots and symbols), whereas the iperf3 JSON only reports the traffic direction and performance results (throughput, loss, jitter) and contains no gNB TDD configuration. The configuration captures were recorded separately from the original iperf3 runs, so they are reference recordings of the supplied settings and not simultaneous recordings. They therefore cannot prove what the gNB broadcast during an earlier test. Also, SIB1 describes the configured pattern, not the actual slot-by-slot scheduling of data.

   
