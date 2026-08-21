# How to Unpack ICMP and TCP Packets in Python (Easy English Guide)

This guide explains, in simple English, how to figure out what **type of data** is inside an IP packet, and how to unpack **ICMP** and **TCP** data.

---

## Step 1: Quick Recap

So far, we have unpacked the **IPv4 packet** and got information like the version, header length, TTL, protocol, source address, and destination address.

Now we need to figure out **what kind of data** is actually inside the packet. We do this by looking at the **protocol number** we already extracted.

---

## Step 2: Understand Protocol Numbers

The protocol number tells us what type of data the packet is carrying. There are many possible protocols, but we only care about the three most common ones:

| Protocol Number | Protocol Name | Used For |
|------------------|----------------|----------|
| `1` | **ICMP** | Diagnosing network problems (Internet Control Message Protocol) |
| `6` | **TCP** | Most web traffic — about 95% of packets (websites, apps, etc.) |
| `17` | **UDP** | Things like DNS requests |

These three cover about 99% of the traffic we'll see, so that's all we'll handle for now.

Later, our program will use logic like:
```python
if protocol == 1:
    # unpack using icmp_packet()
elif protocol == 6:
    # unpack using tcp_segment()
elif protocol == 17:
    # unpack using udp_segment()
```

---

## Step 3: Build the `icmp_packet` Function

Let's start with **ICMP** (Internet Control Message Protocol) — useful for diagnosing network issues.

```python
def icmp_packet(data):
    icmp_type, code, checksum = struct.unpack('! B B H', data[:4])
    return icmp_type, code, checksum, data[4:]
```

### What this does, step by step:

1. **`struct.unpack('! B B H', data[:4])`** — takes the **first 4 bytes** of the data and unpacks them into three values:
   - **`B`** — the **type** (unsigned byte)
   - **`B`** — the **code** (unsigned byte)
   - **`H`** — the **checksum** (unsigned short)
2. **`data[:4]`** — we only look at the first 4 bytes, since that's the ICMP header size.
3. **`return ... data[4:]`** — we return the type, code, and checksum, plus everything **after** the first 4 bytes, which is the actual payload (data).

---

## Step 4: Build the `tcp_segment` Function

Now let's handle **TCP** — the protocol used for around 90% of everyday traffic (visiting websites, apps, etc.).

Since we already unpacked the IP part earlier, what's left now is the TCP information: source port, destination port, sequence number, acknowledgment number, flags, and more.

### Step 4a: Unpack the Main Fields

```python
(src_port, dest_port, sequence, acknowledgment, offset_reserved_flags) = struct.unpack('! H H L L H', data[:14])
```

### What this does:

1. **`H`** — source port (unsigned short)
2. **`H`** — destination port (unsigned short)
3. **`L`** — sequence number (unsigned long)
4. **`L`** — acknowledgment number (unsigned long)
5. **`H`** — a combined 16-bit chunk that holds the **offset**, some **reserved bits**, and the **TCP flags** all together
6. **`data[:14]`** — we grab the first **14 bytes**, since that's the size of this part of the TCP header.

Most of these values (source port, destination port, sequence, acknowledgment) are ready to use right away. But the last value — `offset_reserved_flags` — needs more work, since it has several pieces of information squeezed into one chunk.

### Step 4b: Extract the Offset

The **offset** tells us where the TCP header ends and the actual data begins (similar to the header length in the IP packet).

```python
offset = (offset_reserved_flags >> 12) * 4
```

### What this does:

1. **`offset_reserved_flags >> 12`** — uses a **bitwise right shift** by 12 bits. This pushes out the reserved bits and flags, leaving only the **offset** value.
2. **`* 4`** — multiplying by 4 converts this into the actual offset (in bytes).

### Step 4c: Extract the TCP Flags

TCP flags are like signals computers use to say "hello" and "goodbye" when starting or ending a connection (for example, during the TCP 3-way handshake). The main flags are:

- **URG** (urgent)
- **ACK** (acknowledgment)
- **PSH** (push)
- **RST** (reset)
- **SYN** (synchronize — used to start a connection)
- **FIN** (finish — used to end a connection)

Each flag is extracted using a similar pattern — using the **bitwise AND (`&`)** operator to check a specific bit, similar to how we did it for the IP header length earlier:

```python
flag_urg = (offset_reserved_flags & 32) >> 5
flag_ack = (offset_reserved_flags & 16) >> 4
flag_psh = (offset_reserved_flags & 8) >> 3
flag_rst = (offset_reserved_flags & 4) >> 2
flag_syn = (offset_reserved_flags & 2) >> 1
flag_fin = offset_reserved_flags & 1
```

Each line checks a specific bit position to see if that flag is turned on (`1`) or off (`0`).

### Step 4d: Return Everything

Now we return all the information we extracted, plus the actual data (payload):

```python
return src_port, dest_port, sequence, acknowledgment, flag_urg, flag_ack, flag_psh, flag_rst, flag_syn, flag_fin, data[offset:]
```

### What this does:

- Returns the **source port**, **destination port**, **sequence number**, and **acknowledgment number** — these were already in good shape.
- Returns all the **flags** we extracted.
- Returns **`data[offset:]`** — everything **after** the offset. This is the actual data — most likely something like an **HTTP request** (for example, the contents of a webpage being requested).

---

## Step 5: Summary

- We learned that the **protocol number** (from the IP packet) tells us what type of data we're dealing with: ICMP, TCP, or UDP.
- We built an **`icmp_packet`** function to unpack ICMP data: type, code, checksum, and payload.
- We built a **`tcp_segment`** function to unpack TCP data:
  - Source port, destination port, sequence number, acknowledgment number
  - Used **bitwise shifting** to extract the **offset** (where the real data starts)
  - Used **bitwise AND** operations to extract each individual **TCP flag** (URG, ACK, PSH, RST, SYN, FIN)
  - Returned the actual payload (the real data, like an HTTP request)

---

## What's Next?

The next step is to build a similar function for **UDP** packets — the third common protocol type (used for things like DNS requests).

---

*This guide is based on a video tutorial about building a packet sniffer in Python. It has been rewritten in simple, easy-to-understand English for beginners.*
