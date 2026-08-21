# How to Display IPv4 and ICMP Data in Python (Easy English Guide)

This guide explains, in simple English, how to check what type of data is inside a packet, and how to **display** the information we've unpacked so far in a clear, readable way.

---

## Step 1: Quick Recap

Before this step, two extra things were already built:

1. A function to unpack a **UDP segment** (similar to how ICMP was unpacked in the previous step).
2. A helper function (found online) to **format multi-line data**. This isn't related to packet sniffing directly — it just makes long strings of data (like 7,000+ characters) easier to read by breaking them into multiple lines.

Also, some **constants** were created at the top of the program to represent **tabs** (indentation). These make the text output easier to read and easier to update later, since you only have to change the tab formatting in one place.

---

## Step 2: Why Use Indentation (Tabs)?

Remember how our data is structured — like layers wrapped inside each other:

- **Ethernet frame** (outer layer)
  - **IPv4 packet** (inside the Ethernet frame)
    - **TCP / ICMP / UDP data** (inside the IPv4 packet)

To make this easy to read on screen, we display each layer with an extra level of **indentation**, like this:

```
Ethernet Data:
    IPv4 Data:
        TCP Data:
```

This way, it's visually clear which piece of data belongs inside which layer.

---

## Step 3: Check That We're Working with IPv4

Before displaying ICMP or TCP data, we should first check that the Ethernet protocol we found earlier is actually **IPv4** (regular internet traffic).

```python
# Make sure we're using Ethernet protocol 8 for IPv4
if eth_proto == 8:
```

### What this means:
- If `eth_proto` equals `8`, that means we are dealing with **IPv4** — normal internet traffic.
- If it's not `8`, we can skip processing (since we're not handling other protocol types in this program).

---

## Step 4: Unpack the IPv4 Packet

Once we've confirmed we're working with IPv4, we can extract the IP information using the function we built earlier:

```python
(version, header_length, ttl, proto, src, target, data) = ipv4_packet(data)
```

### What this does:
- We already listened for data and unpacked the Ethernet frame.
- Now we take that data and confirm it's an IPv4 packet.
- We extract: **version**, **header length**, **time to live (TTL)**, **protocol**, **source address**, **target (destination) address**, and the remaining **data** (payload).

---

## Step 5: Display the IPv4 Information

Now let's print this information to the screen in a readable, indented format:

```python
print(TAB_1 + 'IPv4 Packet:')
print(TAB_2 + 'Version: {}, Header Length: {}, TTL: {}'.format(version, header_length, ttl))
print(TAB_2 + 'Protocol: {}, Source: {}, Target: {}'.format(proto, src, target))
```

### What this does:
- **`TAB_1`** — adds one level of indentation, showing this is inside the Ethernet frame.
- **`TAB_2`** — adds a second level of indentation, showing individual details inside the IPv4 packet.
- The information is broken into **two lines** to keep it readable (you could combine or rearrange this however you like).
- This prints out the version, header length, time to live, protocol, source address, and target address — all nicely formatted.

---

## Step 6: Figure Out What's Inside the IPv4 Payload

Now that we've displayed the IP header information, we need to handle the **actual data** inside — which will be one of:

- **ICMP** (protocol number `1`)
- **TCP** (protocol number `6`)
- **UDP** (protocol number `17`)

We check this using the **protocol number** we already extracted from the IPv4 packet (`proto`).

```python
if proto == 1:
    icmp_type, code, checksum, data = icmp_packet(data)
    # display ICMP data here
```

### What this does, step by step:

1. **`if proto == 1:`** — checks if the protocol number equals `1`, which means the payload is an **ICMP packet**.
2. If it matches, we call our `icmp_packet` function (built in an earlier step) to unpack the ICMP data.
3. This returns **four pieces of information**: type, code, checksum, and the remaining data.

> ⚠️ **Important:** If you tried to unpack data using the wrong function (for example, unpacking TCP data using the ICMP function), it would cause errors — since the data isn't structured that way. That's why checking the protocol number first is essential.

---

## Step 7: Repeat for TCP and UDP

The same basic pattern is repeated for TCP (`proto == 6`) and UDP (`proto == 17`):

1. Check if the protocol number matches.
2. If it does, call the matching function (`tcp_segment()` or `udp_segment()`) to unpack the data.
3. Display the extracted information using `print()` statements with the appropriate indentation level.

Since this follows the same pattern each time, it's mostly repetitive — just swapping which function is called and which fields get displayed.

---

## Step 8: Summary

- We learned why **indentation (tabs)** is used to make nested packet data easier to read.
- We checked that the Ethernet protocol equals `8`, confirming we're working with **IPv4** traffic.
- We unpacked the **IPv4 packet** and displayed its header information (version, header length, TTL, protocol, source, target).
- We checked the **protocol number** inside the IPv4 packet to determine whether the payload is **ICMP**, **TCP**, or **UDP**.
- We unpacked and prepared to display the data for each protocol type, using the same basic pattern each time.

---

## What's Next?

The next step is to finish displaying the TCP and UDP information on screen (following the same approach used for ICMP), completing the full packet sniffer output.

---

*This guide is based on a video tutorial about building a packet sniffer in Python. It has been rewritten in simple, easy-to-understand English for beginners.*
