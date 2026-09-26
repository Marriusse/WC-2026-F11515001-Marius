### Requirements 

To install Wireshark on macOS, I used the official installer. The installation process is as follows:

#### 1. Download Wireshark

Visit the official [Wireshark website](https://www.wireshark.org/) and click the **Download** button.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f866faf4-3092-42b6-92f1-e9dfadafefd8" alt="Wireshark download page">
</p>

#### 2. Select the macOS Version

On the download page, select the appropriate macOS installer for your computer. For Apple Silicon Macs, select the Arm 64-bit version.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d2cc8d81-6235-421e-9a19-d8ba2ef93c7a" alt="Wireshark macOS download options">
</p>

#### 3. Install the Application

Once the download is complete, open the `.dmg` file. A standard macOS installation window will appear. Drag the Wireshark application into the Applications folder to complete the installation.

<p align="center">
  <img src="https://github.com/user-attachments/assets/077d895b-1daa-4359-a0ce-3ffb2809edb5" alt="Wireshark macOS installation window">
</p>

## 1. Website Packet Capture (HTTPS)

#### 1. Which website did you access?
The website accessed was **[www.wikipedia.org](http://www.wikipedia.org)**.

#### 2. What are the IP address and port number of the website server?
The website server uses the following IPv6 address and port number:
* **IP address:** `2001:df2:e500:ed1a::1`
* **Port:** `443` (HTTPS)

#### 3. What are the IP address and source port number of your PC when initially accessing the website?
My Mac uses the following IPv6 address and source port:
* **IP address:** `2001:b011:2020:748c:d42a:465a:b515:c610`
* **Source port:** `53107`

#### 4. Identify the SYN, SYN-ACK, and ACK packets & explain their purpose

**SYN (Synchronize)**
* **Purpose:** The client sends this packet to the server to initiate a TCP connection and synchronize sequence numbers.
<p align="center">
  <img src="https://github.com/user-attachments/assets/ede52df7-c85c-46a7-aa97-17e6aabd7343" alt="SYN packet">
</p>

**SYN-ACK (Synchronize-Acknowledge)**
* **Purpose:** The server responds to acknowledge the client's connection request (ACK) and sends its own synchronization request (SYN).
<p align="center">
  <img src="https://github.com/user-attachments/assets/d8142628-ce62-4e6a-9031-7928a90c2cbc" alt="SYN-ACK packet">
</p>

**ACK (Acknowledge)**
* **Purpose:** The client sends this packet to acknowledge the server's SYN-ACK. The TCP three-way handshake is now complete, and data can be transmitted.
<p align="center">
  <img src="https://github.com/user-attachments/assets/a61d54b7-65f4-4137-869e-b6b5ad3c6a7d" alt="ACK packet">
</p>

---

## 2. DNS Packet Analysis

#### 1. What are the IP address and port number of the DNS server?
Based on the `nslookup` output and the packet capture, the DNS server uses the following:
* **IP address:** `2001:b000:168::2`
* **Port:** `53` (DNS)

#### 2. What is the domain name in the DNS query?
The domain name queried is `www.wikipedia.org`.

#### 3. Which protocols does this DNS packet use? (Layer 2 to Layer 5)
Based on the packet details pane in Wireshark, the protocols used according to the TCP/IP five-layer model are:
* **Layer 2 (Link Layer):** Ethernet II
* **Layer 3 (Network Layer):** Internet Protocol Version 6 (IPv6)
* **Layer 4 (Transport Layer):** User Datagram Protocol (UDP)
* **Layer 5 (Application Layer):** Domain Name System (DNS)

<p align="center">
  <img width="1512" height="922" alt="DNS Packet Capture" src="https://github.com/user-attachments/assets/007a76dd-4a3d-4b0b-b62b-b63b814ea453" />
</p>

## 3. Unencrypted HTTP Page Access

#### 1. Which HTTP page did you access?

The HTTP page accessed was `http://www.gzxyzn.com/Article/bjrk2/1644.html`.

#### 2. What are the IP address and port number of the server hosting the page?

Based on the destination details of the captured packet, the server's information is:

* **IP address:** `61.183.8.129`
* **Port:** `80` (the standard port for unencrypted HTTP)

#### 3. What is the HTTP request method?

The HTTP request method used to retrieve the page was **GET**, as shown in packet 85.

<p align="center">
  <img src="https://github.com/user-attachments/assets/ebd74bee-ae97-402b-b96f-d590d69d8be0" alt="HTTP GET request">
</p>

#### 4. What is the HTTP response status code, and what does it mean?

The server returned a **200 OK** status code, as shown in packet 99.

A **200 OK** status code indicates that the HTTP request was successfully processed by the server and that the requested resource was returned.

<p align="center">
  <img src="https://github.com/user-attachments/assets/53e08e14-0ebc-489f-bdb3-9dba9f6438d9" alt="HTTP 200 OK response">
</p>

