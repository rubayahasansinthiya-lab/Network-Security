# How to Unpack an IPv4 Packet in Python (Easy English Guide)

This guide explains, in simple English, how to unpack an **IPv4 packet** — the data that lets your computer talk to other computers on the internet (like Facebook, YouTube, or Reddit).

---

## Step 1: Quick Recap

So far, our packet sniffer program can:
- Listen for data on the network.
- Unpack the **Ethernet frame** to find the destination, source, and protocol.
- Find that there is **another package hidden inside** the Ethernet frame — this is the **IP packet**.

Now it's time to unpack that IP packet.

---

## Step 2: What Is an IP Packet?

While the Ethernet frame handles data moving between your computer and your router, the **IP packet** is what lets your computer connect to **any other computer on the internet** — like Facebook's server, YouTube, or Reddit.

**IP** stands for **Internet Protocol** — it's the system used to send data across the internet.

Every computer has an **IP address**. So when your computer wants to talk to Facebook, it needs:
- Facebook's IP address (where the data is going)
- Your own IP address (so Facebook knows where to send data back)

The IP packet contains:
- **IP version**
- **Header length**
- **Time to live (TTL)**
- **Protocol**
- **Source address** (your IP address)
- **Destination address** (the IP address you're connecting to)

---

## Step 3: Create the `ipv4_packet` Function

Let's start building the function that unpacks this information.

1. Add a comment above it:
   ```python
   # Unpack IPv4 packet
   ```
2. Define the function:
   ```python
   def ipv4_packet(data):
   ```
3. This function will receive the small bundle of data (the payload) that we found inside the Ethernet frame.

> Note: For now, we only care about normal internet traffic (IPv4) — not other types like ARP.

---

## Step 4: Understand the IP Header

Before extracting the data, it helps to know: an **IP header** comes before the actual **IP payload** (the real data). 

Think of it like a package sent by mail:
- The **header** = the label on the outside (address info, size, etc.)
- The **payload** = the actual contents inside the box

We need to extract several pieces of information from the header:
- Version
- Header length
- Time to live (TTL)
- Protocol
- Source address
- Destination address

---

## Step 5: Extract the Version and Header Length

The very first **byte** of the IP packet contains **both** the version and the header length combined together. We need to separate them.

```python
version_header_length = data[0]
version = version_header_length >> 4
header_length = (version_header_length & 15) * 4
```

### What this does, step by step:

1. **`data[0]`** — grabs the very first byte, which contains both the version and header length mixed together.
2. **`version_header_length >> 4`** — this uses a **bitwise right shift** by 4 bits. This removes the header length part, leaving only the **version number**.
3. **`version_header_length & 15`** — this uses a **bitwise AND** operation with the number 15. This compares the bits and keeps only the header length part.
4. **`* 4`** — multiplying by 4 converts this value into the actual **header length**.

### Why do we need the header length?

The header length tells us **where the header ends and the actual data begins**. Since the header and the data are placed right next to each other, we need to know exactly how long the header is so we know where to start reading the real data.

---

## Step 6: Unpack the Rest of the Header

Now we unpack the remaining information: time to live, protocol, source address, and target (destination) address.

```python
ttl, proto, src, target = struct.unpack('! 8x B B 2x 4s 4s', data[:20])
```

### What this does, step by step:

1. **`!`** — makes sure the byte order is correct (network byte order).
2. **`8x`** — skips the first 8 bytes we don't need (these were already handled, like version/header length).
3. **`B`** — unpacks the **time to live (TTL)** as an unsigned byte.
4. **`B`** — unpacks the **protocol** as an unsigned byte.
5. **`2x`** — skips 2 more bytes we don't need.
6. **`4s`** (used twice) — unpacks the **source address** and **target (destination) address**, each **4 bytes** long.
7. **`data[:20]`** — we only look at the **first 20 bytes**, because the entire IP header is always **20 bytes long**.

---

## Step 7: Return the Extracted Information

Now we return everything we've extracted so it can be used later:

```python
return version, header_length, ttl, proto, ipv4(src), ipv4(target), data[header_length:]
```

### What this includes:
- **`version`** — the IP version.
- **`header_length`** — the length of the header.
- **`ttl`** — time to live.
- **`proto`** — the protocol.
- **`ipv4(src)`** — the source IP address, properly formatted (using a function we'll write next).
- **`ipv4(target)`** — the destination IP address, properly formatted.
- **`data[header_length:]`** — everything **after** the header — this is the actual payload/data.

> Just like we formatted the MAC address earlier, we now need a similar function to properly format the IP address, since it comes out in a raw, unformatted style.

---

## Step 8: Create the `ipv4` Function to Format IP Addresses

An IP address is normally written like this: `123.456.789.0` — three numbers separated by dots (note: real IP numbers only go up to 255, unlike the example number used for illustration).

Let's build a function to format it properly.

1. Add a comment:
   ```python
   # Returns properly formatted IPv4 address
   ```
2. Define the function:
   ```python
   def ipv4(addr):
       return '.'.join(map(str, addr))
   ```

### What this does, step by step:

1. **`map(str, addr)`** — takes each chunk (number) in the raw address and converts it into a **string**.
2. **`'.'.join(...)`** — joins all those string chunks together, separated by a **dot (`.`)**.
3. The result is a normal-looking IP address, like `127.0.0.1`.

---

## Step 9: Summary

- We built a function (`ipv4_packet`) that unpacks the IP header to get: version, header length, time to live, protocol, source address, and destination address.
- We used **bitwise operations** (`>>` and `&`) to separate the version and header length, which were combined in a single byte.
- We used `struct.unpack` to extract the rest of the header fields.
- We built a second function (`ipv4`) to properly format IP addresses into the normal dotted format (e.g., `127.0.0.1`).
- We also returned the actual payload — the real data that comes right after the header ends.

---

## What's Next?

Now that we can unpack both the Ethernet frame and the IPv4 packet, we're ready to move on to figuring out even more details — like what specific websites are being visited and what information is being sent.

---

*This guide is based on a video tutorial about building a packet sniffer in Python. It has been rewritten in simple, easy-to-understand English for beginners.*
