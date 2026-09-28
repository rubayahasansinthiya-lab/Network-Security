# DHCP and the DORA Process (Complete Notes)

> Tip for Notion: paste this file into a Notion page. Upload `dhcp_dora_diagram.png` under Section 10.

---

## 1. What is DHCP?

- DHCP means **Dynamic Host Configuration Protocol**.
- DHCP is an **Application Layer protocol**.
- It gives IP addresses to devices **automatically**.
- A device does not need a manual setup.

## 2. Why do we use DHCP?

- Imagine a lab or home with 50 computers.
- If you set the IP, subnet mask and gateway by hand on every computer, it takes a lot of time.
- Manual setup also causes mistakes.
- DHCP solves this. When a new device connects, DHCP **leases** (lends) a free IP address to it.
- DHCP also **stops IP address conflicts**.
- DHCP can be a service on a server, or it can be built into a router.

## 3. What does DHCP give to a device?

- IP address
- Subnet mask
- Router (gateway) address
- DNS address
- Vendor class identifier
- Lease time (how long the device can use the IP)

## 4. Question: How does DHCP work? (Briefly illustrate)

**Answer:** DHCP uses a 4-step process called **DORA**. It gets the IP address from a central server.

**D** = Discover, **O** = Offer, **R** = Request, **A** = Acknowledge

### 4.1 Summary table

| Step | Message | Sender → Receiver | Simple meaning |
|---|---|---|---|
| 1 (D) | DHCP Discover | Client → Server | The client looks for a DHCP server. |
| 2 (O) | DHCP Offer | Server → Client | The server offers a free IP and lease time. |
| 3 (R) | DHCP Request | Client → Server | The client accepts the offer and asks to lock that IP. |
| 4 (A) | DHCP Acknowledge (ACK) | Server → Client | The server confirms the IP, subnet mask and lease time. |

### 4.2 Each step in detail

**Step 1: DHCP Discover**
- The client sends it first.
- The new client has no IP address yet, so it shouts to the whole network: "Is there any DHCP server here? I need an IP!"
- It is a broadcast message.
- Source IP: `0.0.0.0` (the client has no IP yet)
- Destination IP: `255.255.255.255` (everyone)
- Source MAC: the client's MAC address
- Destination MAC: `FF:FF:FF:FF:FF:FF` (broadcast MAC)

**Step 2: DHCP Offer**
- The server receives the Discover message.
- It checks its list and picks a free IP.
- It replies: "I have a free IP for you, for a fixed time."
- The offer has the IP address and the lease time.
- Source IP: the DHCP server's IP
- Destination IP: `255.255.255.255` (the client still has no IP)
- Source MAC: the DHCP server's MAC
- Destination MAC: the client's MAC

**Step 3: DHCP Request**
- The client gets the offer.
- It says: "Yes, I accept this IP. Please lock it for me."
- The client may get offers from many servers. It accepts the offer that reaches it first.
- The message is sent as a **broadcast**, so all other servers hear it. They know their offers are rejected and can free their IPs.
- Source IP: `0.0.0.0` (the IP is not officially assigned yet)
- Destination IP: `255.255.255.255`
- Source MAC: the client's MAC
- Destination MAC: the DHCP server's MAC (as written in the source text)

**Step 4: DHCP Acknowledge (ACK)**
- This is the last step.
- The server confirms the request.
- The message has the IP address, subnet mask and lease time.
- After this step, the client can use the network.
- Source IP: the DHCP server's IP
- Destination IP: `255.255.255.255`
- Source MAC: the DHCP server's MAC
- Destination MAC: the client's MAC

### 4.3 Class example (easy to remember)

- Discover = A new student walks in and shouts, "Who can give me a roll number?"
- Offer = The teacher says, "I have roll number 50 for you."
- Request = The student says, "Yes, please give me roll number 50."
- Acknowledge = The teacher says, "Done. Roll number 50 is yours now."

---

## 5. Question: DHCP is an Application Layer protocol. How does it work in the Network Layer and Data Link Layer?

**Answer:** It works through **Encapsulation**.

- When data goes down from the top layer to the bottom layer, each layer adds its own header to the data.
- This is called Encapsulation.

### Steps of encapsulation for a DHCP message

1. **Application Layer:** DHCP makes the message, for example "I need an IP" or "Here is your IP". DHCP cannot send this message through the wire by itself.
2. **Transport Layer:** DHCP uses **UDP**. This layer adds the port numbers (67 and 68).
3. **Network Layer:** This layer adds the **Source IP** and **Destination IP**. For a new client, these are `0.0.0.0` and `255.255.255.255`.
4. **Data Link Layer:** This layer adds the **Source MAC** and **Destination MAC**.

**Easy line:** DHCP is an Application Layer protocol, but it needs the Network Layer (IP) and Data Link Layer (MAC) to travel through the network.

### Port numbers

- DHCP works on top of **UDP**.
- **Port 67** = DHCP server
- **Port 68** = DHCP client

---

## 6. Question: When is it Broadcast and when is it Unicast?

### 6.1 Basic meaning

- **Broadcast** = a message for everyone. Example: a new student shouts in the classroom because he does not know anyone.
- **Unicast** = a message for one device only. Example: the teacher looks at that one student and speaks, because the teacher already knows the student's face (MAC address).

### 6.2 Main table

| Step | Message | Network Layer (IP) | Data Link Layer (MAC) |
|---|---|---|---|
| 1 | Discover | Broadcast | Broadcast |
| 2 | Offer | Broadcast | Unicast |
| 3 | Request | Broadcast | Unicast |
| 4 | Acknowledge | Broadcast | Unicast |

### 6.3 Why is each step like this?

- **Step 1 (Discover):** The client has no IP and does not know who the server is. So it broadcasts on both layers.
- **Step 2 (Offer):** IP layer is broadcast because the client has not set the offered IP on itself yet. MAC layer is unicast because the server already learned the client's MAC address from Step 1.
- **Step 3 (Request):** IP layer is broadcast so that all other servers know which offer was accepted. MAC layer is unicast to the chosen server, as the source text says.
- **Step 4 (Acknowledge):** Same as Step 2. IP layer is broadcast until the client activates its IP. MAC layer is unicast to that client.

### 6.4 One-line shortcut

> In all 4 steps, the **Network Layer (IP) is always Broadcast**. In the **Data Link Layer (MAC)**, only Step 1 is Broadcast. Steps 2, 3 and 4 are Unicast.

---

## 7. Question: Why is the IP layer always Broadcast?

- **Step 1 and Step 3:** The client has no valid IP (it is `0.0.0.0`). Without an IP, it cannot do one-to-one IP communication. So it must send to `255.255.255.255`.
- **Step 2 and Step 4:** The server is offering or confirming an IP. But until the client receives the final ACK and activates the IP, the client has no IP. So the server sends the message as an IP broadcast, so that the client can still receive it.
- **Extra reason for Step 3:** The network can have more than one DHCP server. The broadcast tells the other servers, "I took another server's offer. Free your IPs."

## 8. Question: Why is the MAC layer Unicast after Step 1?

- **Step 1:** The client does not know the server's MAC address. So it uses `FF:FF:FF:FF:FF:FF` (broadcast MAC). Every device on the switch gets it, and the server hears it too.
- **Steps 2, 3, 4:** In Step 1, the client wrote its own **Source MAC** in the packet. The server saw it and saved it. Now both sides know each other's MAC address.
- A switch works with a **MAC table**. So it can send the packet only to the correct port (unicast).
- Even without an IP, devices in a LAN can talk one-to-one by using MAC addresses.

**Core reason in one line:** The IP layer stays broadcast because the client's IP setup is not complete until the end. The MAC layer becomes unicast after Step 1 because everyone now knows each other's MAC address.

---

## 9. Points to remember for the exam

- DHCP = Application Layer protocol.
- DHCP uses UDP. Server port = 67, client port = 68.
- DORA = Discover, Offer, Request, Acknowledge.
- Until the ACK, the client has no IP, so it uses `0.0.0.0` as the source IP.
- The client broadcasts to `255.255.255.255` because it does not know where the server is.
- Discover and Request are sent by the client. Offer and Acknowledge are sent by the server.
- In Discover and Request, the source IP is `0.0.0.0`.
- In Offer and Acknowledge, the source IP is the server's IP.
- IP layer: all 4 messages are Broadcast.
- MAC layer: Discover is Broadcast. Offer, Request and Acknowledge are Unicast.

> Note about real networks: The Request message is often also sent as a broadcast at the MAC layer (`FF:FF:FF:FF:FF:FF`), because the client wants every server to hear it. If you see this in Wireshark, do not be confused. For the exam, write the version from your source text (the table in Section 6).

## 10. Diagram

![DHCP DORA diagram](dhcp_dora_diagram.png)

Text version of the diagram:

```
[ Client (IP: 0.0.0.0) ]                              [ DHCP Server ]
        |                                                    |
        | -- 1. DHCP DISCOVER (Broadcast) -----------------> |
        |    IP: 0.0.0.0 -> 255.255.255.255                  |
        |    MAC: Client MAC -> FF:FF:FF:FF:FF:FF            |
        |                                                    |
        | <---------------- 2. DHCP OFFER ------------------ |
        |    IP: Server IP -> 255.255.255.255                |
        |    MAC: Server MAC -> Client MAC (Unicast)         |
        |                                                    |
        | -- 3. DHCP REQUEST (Broadcast) ------------------> |
        |    IP: 0.0.0.0 -> 255.255.255.255                  |
        |    MAC: Client MAC -> Server MAC (Unicast)         |
        |                                                    |
        | <------------ 4. DHCP ACKNOWLEDGE ---------------- |
        |    IP: Server IP -> 255.255.255.255                |
        |    MAC: Server MAC -> Client MAC (Unicast)         |
        |                                                    |
[ Connected! IP active ]                     [ IP leased to Client MAC ]
```

---

## 11. Lab connection (Python Scapy) — Code and Implementation

- In the lab, you may be asked to build DORA packets with Scapy.
- In Scapy, the `Ether()`, `IP()`, `UDP()`, `BOOTP()` and `DHCP()` layers are stacked together to build one DHCP packet. This is the same **Encapsulation** idea from Section 5, just written in code.
- Each layer in the code matches one layer of the network:

| Scapy layer | Network layer it builds |
|---|---|
| `Ether()` | Data Link Layer (MAC) |
| `IP()` | Network Layer (IP) |
| `UDP()` | Transport Layer (Port 67/68) |
| `BOOTP()` | Carries client/server IP fields |
| `DHCP()` | Application Layer (the DHCP message type) |

### 11.1 Step 1: Send DHCP Discover (Client side)

```python
from scapy.all import *

# Build the Discover packet, layer by layer (bottom to top in the code,
# but this is exactly the encapsulation from Section 5)
discover = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /                     # MAC layer: Broadcast
    IP(src="0.0.0.0", dst="255.255.255.255") /            # IP layer: Broadcast
    UDP(sport=68, dport=67) /                              # Client port -> Server port
    BOOTP(chaddr=get_if_hwaddr("eth0")) /                  # Client's own MAC in the payload
    DHCP(options=[("message-type", "discover"), "end"])
)

sendp(discover, iface="eth0")
print("DHCP Discover sent.")
```

- This matches **Step 1** in Section 4: Source IP `0.0.0.0`, Destination IP `255.255.255.255`, Destination MAC `ff:ff:ff:ff:ff:ff`.

### 11.2 Step 2: Sniff and reply with DHCP Offer (Server side, for lab testing)

```python
from scapy.all import *

def make_offer(pkt):
    if DHCP in pkt and pkt[DHCP].options[0][1] == 1:  # 1 = discover
        client_mac = pkt[Ether].src
        offer = (
            Ether(src="aa:bb:cc:dd:ee:ff", dst=client_mac) /   # MAC layer: Unicast
            IP(src="192.168.1.1", dst="255.255.255.255") /     # IP layer: Broadcast
            UDP(sport=67, dport=68) /                           # Server port -> Client port
            BOOTP(yiaddr="192.168.1.50", siaddr="192.168.1.1",
                  chaddr=pkt[BOOTP].chaddr) /
            DHCP(options=[("message-type", "offer"),
                           ("subnet_mask", "255.255.255.0"),
                           ("lease_time", 86400),
                           "end"])
        )
        sendp(offer, iface="eth0")
        print("DHCP Offer sent to", client_mac)

sniff(filter="udp and (port 67 or port 68)", prn=make_offer, iface="eth0")
```

- This matches **Step 2**: MAC is Unicast (sent straight to `client_mac`), IP is still Broadcast.

### 11.3 Full DORA in one script (for lab demonstration)

```python
from scapy.all import *

iface = "eth0"

def dhcp_discover():
    pkt = (Ether(dst="ff:ff:ff:ff:ff:ff") /
           IP(src="0.0.0.0", dst="255.255.255.255") /
           UDP(sport=68, dport=67) /
           BOOTP(chaddr=get_if_hwaddr(iface)) /
           DHCP(options=[("message-type", "discover"), "end"]))
    sendp(pkt, iface=iface)
    print("[1] Discover -> sent (Broadcast/Broadcast)")

def dhcp_request(offered_ip, server_ip, server_mac):
    pkt = (Ether(dst="ff:ff:ff:ff:ff:ff") /
           IP(src="0.0.0.0", dst="255.255.255.255") /
           UDP(sport=68, dport=67) /
           BOOTP(chaddr=get_if_hwaddr(iface)) /
           DHCP(options=[("message-type", "request"),
                          ("requested_addr", offered_ip),
                          ("server_id", server_ip),
                          "end"]))
    sendp(pkt, iface=iface)
    print("[3] Request -> sent for IP", offered_ip)

# In a real lab you would sniff the Offer and ACK in between
# and call dhcp_request() once the Offer is received.
dhcp_discover()
```

### 11.4 What the code teaches you (map back to theory)

- `Ether(dst="ff:ff:ff:ff:ff:ff")` = MAC broadcast, only used for Discover (Step 1).
- `IP(src="0.0.0.0", dst="255.255.255.255")` = why IP layer is always Broadcast (Section 7).
- `sport=68, dport=67` = client-to-server direction, matches Section 5 port numbers.
- `BOOTP(chaddr=...)` = this is where the client writes its own MAC, so the server can learn it and reply with Unicast later (Section 8).
- `DHCP(options=[("message-type", ...)])` = this single option is the actual Application Layer message (Discover/Offer/Request/ACK).

### 11.5 Security note: DHCP Starvation Attack

- If a script sends many fake Discover packets with different random MAC addresses in a loop, the server keeps giving out IPs until it has none left.
- This is called a **DHCP Starvation Attack**.
- Simple demo idea (do not run on a real/production network, lab only):

```python
from scapy.all import *
import random

def random_mac():
    return "02:%02x:%02x:%02x:%02x:%02x" % tuple(random.randint(0,255) for _ in range(5))

for i in range(20):
    pkt = (Ether(src=random_mac(), dst="ff:ff:ff:ff:ff:ff") /
           IP(src="0.0.0.0", dst="255.255.255.255") /
           UDP(sport=68, dport=67) /
           BOOTP(chaddr=random_mac()) /
           DHCP(options=[("message-type", "discover"), "end"]))
    sendp(pkt, iface="eth0", verbose=0)

print("Sent 20 fake Discover packets (lab demo only).")
```

- Defense against this attack: **DHCP Snooping** and **Port Security** on the switch, which limit how many MAC addresses/DHCP requests are allowed per port.

## 12. Diagram

![DHCP DORA diagram](dhcp_dora_diagram.png)

Text version of the diagram:

```
[ Client (IP: 0.0.0.0) ]                              [ DHCP Server ]
        |                                                    |
        | -- 1. DHCP DISCOVER (Broadcast) -----------------> |
        |    IP: 0.0.0.0 -> 255.255.255.255                  |
        |    MAC: Client MAC -> FF:FF:FF:FF:FF:FF            |
        |                                                    |
        | <---------------- 2. DHCP OFFER ------------------ |
        |    IP: Server IP -> 255.255.255.255                |
        |    MAC: Server MAC -> Client MAC (Unicast)         |
        |                                                    |
        | -- 3. DHCP REQUEST (Broadcast) ------------------> |
        |    IP: 0.0.0.0 -> 255.255.255.255                  |
        |    MAC: Client MAC -> Server MAC (Unicast)         |
        |                                                    |
        | <------------ 4. DHCP ACKNOWLEDGE ---------------- |
        |    IP: Server IP -> 255.255.255.255                |
        |    MAC: Server MAC -> Client MAC (Unicast)         |
        |                                                    |
[ Connected! IP active ]                     [ IP leased to Client MAC ]
```
