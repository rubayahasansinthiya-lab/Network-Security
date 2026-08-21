# Building a Packet Sniffer in Python — Full Step-by-Step Guide

This is the complete, guide for building a packet sniffer in Python. 

## Table of Contents

1. [Part 1: What Is a Packet Sniffer? (Ethernet Frame Basics)](#part-1)
2. [Part 2: Formatting a MAC Address](#part-2)
3. [Part 3: Building the Main Loop and Socket](#part-3)
4. [Part 4: Unpacking an IPv4 Packet](#part-4)
5. [Part 5: Unpacking ICMP and TCP Packets](#part-5)
6. [Part 6: Displaying IPv4 and ICMP Data](#part-6)
7. [Part 7: Testing and Running the Packet Sniffer](#part-7)

---


<a id="part-1"></a>

# How to Make a Packet Sniffer in Python

This guide explains, how to start building a **packet sniffer** (also called a **network sniffer**) using Python. A packet sniffer is a tool that looks at the data moving across a network.

---

## Step 1: What Is a Packet Sniffer?

When your computer sends data through an Ethernet cable to your router, it sends the data as electricity pulses.

- A pulse of electricity = the number **1**
- No pulse = the number **0**

Your computer turns these 1s and 0s into **binary**, and then into real information like numbers, letters, and images.

A **packet sniffer** is a program that looks at this data as it moves across the network, and turns it back into readable information.

---

## Step 2: Why Build a Packet Sniffer?

Here are some reasons people build packet sniffers:

- To **monitor network traffic** (for example, checking what websites are visited on a home network).
- To see **what requests are being made** and **who is talking to whom** on a network.
- To **find problems** on a large network (for example, finding out why the network is slow).

Along the way, you also learn a lot about how networking works.

---

## Step 3: Understand the Basics First

Before starting, it helps to already know the basics of networking. If you don't, it's a good idea to learn:

- Basic networking concepts
- How to use Python's `struct` module (used to convert data to and from "bytes" format)

---

## Step 4: How Data Travels — A Simple Example

Let's say you want to look at a picture on Reddit.

1. Your browser creates a request for the image.
2. This request is wrapped inside an **HTTP request**.
3. To send this request, your computer needs:
   - The **IP address** of Reddit's server (where the request is going)
   - Your computer's **return address** (so Reddit knows where to send the answer back)
4. The HTTP request is wrapped inside something called an **IP packet** (this holds the addresses).
5. The IP packet is wrapped inside an **Ethernet frame**.

The **Ethernet frame** is the very first thing your computer sends to your router. This is what we need to understand first, and it's where our packet sniffer will start.

---

## Step 5: What Is Inside an Ethernet Frame?

An Ethernet frame has several parts:

| Part | What It Means |
|------|----------------|
| **Sync** | Makes sure your computer and router are "in sync" so they know when data is being sent. Not very useful for humans to read. |
| **Receiver / Destination** | The address of the computer or device **receiving** the data. |
| **Sender / Source** | The address of the computer or device **sending** the data. |
| **Type** | The type of protocol used (for example, IP version 4 — normal internet traffic). |
| **Payload (Data)** | The actual data being sent — this is the "package contents." |
| **Frame Check** | Checks that all the data arrived correctly, with no errors. |

Think of it like mailing a package:
- The **sender/receiver addresses** = the writing on the outside of the package
- The **payload** = what's inside the package

---

## Step 6: Start Coding — Create the `ethernet_frame` Function

Now let's start building the code.

1. Create a function called `ethernet_frame`.
2. This function will take in `data` — this is the raw information (a bunch of 1s and 0s) that we sniff from the network.
3. Inside the function, we will **unpack** this data to find:
   - The **destination** address
   - The **source** address
   - The **protocol/type**

Example idea (in plain terms, not exact code):

```
def ethernet_frame(data):
    dest_mac, src_mac, proto = struct.unpack('! 6s 6s H', data[:14])
    return dest_mac, src_mac, socket.htons(proto), data[14:]
```

### What this code does, step by step:

1. **`struct.unpack`** — takes the raw bytes and converts them into a readable format.
2. **`!` (exclamation mark)** — tells Python to treat this as **network data** (this fixes a difference in how computers store data vs. how it travels across a network — this is called "big-endian" vs "little-endian" ordering).
3. **`6s`** (used twice) — the destination and source addresses (called **MAC addresses**) are each **6 bytes** long.
4. **`H`** — the protocol/type is a **small unsigned integer** (2 bytes).
5. **`data[:14]`** — we only look at the **first 14 bytes** of the data (6 bytes + 6 bytes + 2 bytes = 14 bytes total). This gives us the destination, source, and type.
6. **`data[14:]`** — everything **after** the first 14 bytes is the **payload** (the actual data being sent, like an image or text). We don't know exactly how big this will be — it depends on what's being sent.

---

## Step 7: Why the Result Isn't Human-Readable Yet

After unpacking, the destination and source addresses are **not yet** in the normal MAC address format you're used to seeing (like `00:1A:2B:3C:4D:5E`).

To fix this, we will need a **second function** (covered in the next video/tutorial) called something like `get_mac_address`. This function will:

1. Take the raw destination and source addresses.
2. Format them properly into a human-readable MAC address.

---

## Step 8: Summary of What We Did

- We learned what a packet sniffer is and why it's useful.
- We learned that an Ethernet frame is the first layer of data sent from your computer to your router.
- We built a function (`ethernet_frame`) that:
  - Takes in raw network data
  - Unpacks the first 14 bytes to get the **destination**, **source**, and **protocol type**
  - Returns the **remaining data** as the payload

---

## What's Next?

In the next part of this tutorial series, we will:
- Write the `get_mac_address` function
- Properly format the MAC addresses so they are human-readable

---


<a id="part-2"></a>

# How to Format a MAC Address in Python

This guide explains, how to take a raw MAC address (from a packet sniffer) and turn it into a normal, human-readable format.

---

## Step 1: What Are We Trying to Do?

When we unpack a MAC address from network data, it comes out **broken up into chunks** — not in the format we normally see.

The **standard format** for a MAC address looks like this:

```
AA:BB:CC:DD:EE:FF
```

Rules for this format:
- Each part has **two characters** (two hexadecimal digits).
- Each part is separated by a **colon** (`:`).
- Letters are usually shown in **uppercase**.

Our goal in this step is to write a function that turns the raw, broken-up address into this clean format.

---

## Step 2: Set Up the Function

We already created a function called `get_mac_address` in an earlier step. Now we will fill in what it does.

1. Add a comment above it explaining what it does, for example:
   ```python
   # Return properly formatted MAC address
   ```
2. Define the function `get_mac_address`.
3. This function takes in **one input**: the raw bytes address (the MAC address broken into chunks).

---

## Step 3: Format Each Chunk of the Address

The raw address is **iterable** — meaning it's made of several small chunks (bytes) that we can loop through.

To fix the formatting, we use Python's **`map`** function.

- The `map` function lets you take a function and apply it to every item in a list (or iterable).
- We want to apply a formatting rule to **each chunk** of the MAC address.

The formatting rule we use is:

```python
'{:02x}'.format(chunk)
```

### What this does:
- **`02x`** means: format the value as a **hexadecimal number**, and always show **two digits** (adding a `0` in front if needed).
- This makes sure every chunk looks consistent (for example, `A` becomes `0A`).

So the full line looks like:

```python
bytes_str = map('{:02x}'.format, bytes_address)
```

**Note:** When using `map`, you don't need to add the curly braces separately — you just pass the format string as the function, and the list of chunks as the iterable.

---

## Step 4: Join the Chunks Together with Colons

Now that each chunk is properly formatted (two characters each), we need to:

1. **Join** all the chunks together.
2. Add a **colon (`:`)** between each chunk.
3. Make sure everything is in **UPPERCASE**.

This is done using Python's `.join()` method:

```python
return ':'.join(bytes_str).upper()
```

### What this does, step by step:
- **`':'.join(...)`** — takes all the formatted chunks and puts a colon between each one.
- **`.upper()`** — converts all the letters to uppercase (so `aa` becomes `AA`).
- **`return`** — instead of saving this into a separate variable first (like `mac_address`) and then returning it, we can just return the result directly. This makes the code shorter and cleaner.

---

## Step 5: Final Result

After these steps, calling `get_mac_address()` on raw MAC address bytes will give you a nice, clean, human-readable result like:

```
AA:BB:CC:DD:EE:FF
```

(Note: these are **colons** `:`, not semicolons `;`.)

This makes it easy for anyone reading the output later to understand which device the data is going to or coming from.

---

## Step 6: Summary

- We took the raw MAC address (broken into chunks) from our earlier unpacking step.
- We used `map` to format each chunk into a **2-digit hexadecimal** value.
- We used `join` to combine the chunks with **colons** between them.
- We used `.upper()` to make sure the letters are **uppercase**.
- We returned the final result directly for a clean, simple function.

Now we have a properly formatted MAC address ready to display to the user, and we're ready to move on to the next step in building the packet sniffer.

---

---


<a id="part-3"></a>

# How to Build the Main Loop of a Packet Sniffer in Python

This guide explains, how to build the **main program** for our packet sniffer. This is the part that actually listens for network data and uses our earlier functions to read it.

---

## Step 1: What Are We Building?

Now that we have functions to:
- Unpack an Ethernet frame (`ethernet_frame`)
- Format a MAC address (`get_mac_address`)

...we can build the **main function**. This is the heart of the program.

The main function will:
1. Run **forever** in a loop (as long as the program is running).
2. **Listen** for packets of data coming across the network.
3. When a packet arrives, **extract the information** from it.
4. **Display** that information (or do something else with it).

---

## Step 2: Create a Socket

Before we can listen for anything, we need a **socket**. A socket is what allows your program to have a connection with other computers on the network.

```python
connection = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.ntohs(3))
```

### What each part means:

| Part | Meaning |
|------|---------|
| `socket.AF_PACKET` | Tells Python we want to work with **raw network packets** (low-level data), not a normal connection. |
| `socket.SOCK_RAW` | Means we want **raw data** — the actual unprocessed information moving across the network. |
| `socket.ntohs(3)` | Makes sure the byte order is correct so the data can be read properly on any machine (fixes the "big-endian vs little-endian" issue mentioned in earlier steps). |

> 💡 If you're not familiar with sockets, it helps to first learn the basics of how sockets work in Python.

---

## Step 3: Create the Main Function

Once we have our raw socket, we can build our main loop. Let's call this function `main`.

```python
def main():
    connection = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.ntohs(3))
    while True:
        raw_data, addr = connection.recvfrom(65536)
        # ... process the data here
```

### What this does, step by step:

1. **`while True:`** — This creates an **infinite loop**. The program will keep running and listening forever (until you stop it).
2. **`connection.recvfrom(65536)`** — This tells the socket to **receive data** whenever it arrives.
   - `65536` (or `65535`) is the **largest buffer size** allowed — meaning the biggest chunk of data we can grab at once.
3. This gives us two things:
   - **`raw_data`** — the actual data (a bunch of 1s and 0s) being sent.
   - **`addr`** — the address of where the data is being sent to/from (the source).

---

## Step 4: Send the Raw Data to Our `ethernet_frame` Function

Remember, our `ethernet_frame` function (built in an earlier step) takes raw data and unpacks it into **four pieces**:

1. **Destination MAC address**
2. **Source MAC address**
3. **Ethernet protocol/type**
4. **The actual payload (data)**

So now we take the raw data we just received and pass it into that function:

```python
dest_mac, src_mac, eth_proto, data = ethernet_frame(raw_data)
```

This line takes the raw pulses of data from the network and turns them into four clean, usable variables.

---

## Step 5: Print the Results to Check It's Working

Before moving on, it's a good idea to check that everything works so far by printing out some of the information.

We'll print:
- The **destination** address
- The **source** address
- The **protocol**

(We'll deal with the actual payload/data later, since there's more work needed to process it.)

Example:

```python
print('\nEthernet Frame:')
print('Destination: {}, Source: {}, Protocol: {}'.format(dest_mac, src_mac, eth_proto))
```

### What this does:
- The **curly braces `{}`** act as **placeholders**.
- The `.format(...)` method fills in those placeholders with the actual values — in this case, the destination, source, and protocol.
- Since we already formatted the MAC addresses properly (in the earlier step), they will print out nicely, like `AA:BB:CC:DD:EE:FF`.

---

## Step 6: Test It

1. Run the program.
2. Open a web browser and refresh a page (or browse to any website) to generate some network traffic.
3. Check your program's output — you should see the destination, source, and protocol being printed for each packet.

At this point, the **protocol** value is especially important — a value like `8` typically represents normal internet traffic (IPv4), which is what we care about most.

---

## Step 7: Summary

- We created a **raw socket** so our program can listen to network traffic directly.
- We built a **main loop** that runs forever, listening for incoming packets.
- Each time data comes in, we pass it to our `ethernet_frame` function to unpack it into: destination, source, protocol, and payload.
- We printed out the destination, source, and protocol to confirm everything is working correctly.

---

## What's Next?

Now that the basic Ethernet frame handling is working, the next step is to start breaking down the **actual payload** (the real data). This will let us start figuring out things like:
- What websites are being visited
- What information is being typed in
- Other details about the network traffic

---


<a id="part-4"></a>

# How to Unpack an IPv4 Packet in Python

This guide explains, how to unpack an **IPv4 packet** — the data that lets your computer talk to other computers on the internet (like Facebook, YouTube, or Reddit).

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



<a id="part-5"></a>

# How to Unpack ICMP and TCP Packets in Python

This guide explains, how to figure out what **type of data** is inside an IP packet, and how to unpack **ICMP** and **TCP** data.

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



<a id="part-6"></a>

# How to Display IPv4 and ICMP Data in Python

This guide explains, how to check what type of data is inside a packet, and how to **display** the information we've unpacked so far in a clear, readable way.

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


<a id="part-7"></a>

# How to Test and Run Your Python Packet Sniffer

This is the final guide in the series. It explains, how to finish displaying TCP and UDP data, and how to **run and test** your finished packet sniffer.

---

## Step 1: Quick Recap

In the previous steps, we already:
- Displayed the **ICMP** data.
- The **TCP** and **UDP** display code follows the exact same pattern as ICMP — just with different fields.

Since it's repetitive, the TCP and UDP display formatting was already written ahead of time. If you want to customize how the information is displayed (colors, layout, wording, etc.), you can simply edit that formatting code however you like.

---

## Step 2: Run the Program with the Right Permissions

Packet sniffing requires special access to your network hardware, so you **must run the program with administrator/root privileges**.

### On Mac/Linux:
```bash
sudo python3 your_script_name.py
```

### On Windows:
Run your terminal, command prompt, or code editor (like PyCharm) **as Administrator**, then run the script normally.

> ⚠️ If you don't run it with the correct privileges, the program will **not work properly** — it won't have permission to access raw network data.

---

## Step 3: Test the Program

1. Run the script using the steps above.
2. Open your **web browser** and refresh a page (or browse to any website).
3. Watch the terminal — you should start seeing packet data appear as your browser sends and receives information.

---

## Step 4: Understand the Output Format

When you look at the output, you'll notice it uses **different levels of indentation** to show how the data is nested:

```
Ethernet Frame:
    Destination: ..., Source: ..., Protocol: ...
    IPv4 Packet:
        Version: ..., Header Length: ..., TTL: ...
        Protocol: ..., Source: ..., Target: ...
        TCP Segment:
            Source Port: ..., Destination Port: ...
            ...
```

### Why it's structured this way:
- The **outermost layer** is the **Ethernet frame** — the very first layer of data.
- Inside that is the **IPv4 packet** — one indentation level deeper.
- Inside that is the **TCP segment** (or ICMP/UDP) — another level deeper.

This matches how the real data is actually structured — each layer is wrapped inside the one before it, like nested boxes.

> 💡 You could indent things even further to separate every single piece of data, but keeping it at these levels helps prevent the text from running too wide off the screen, while still keeping things organized and easy to read.

---

## Step 5: Why the Payload Formatting Function Matters

Most of the time, when you're just browsing the internet, the protocol you'll see is **TCP** — since that's what's used for most everyday web traffic.

The actual payload data (like an **HTTP request**) can be very long — sometimes thousands of characters on a single line. Reading that as one giant line of text would be very hard to follow.

This is exactly why the **multi-line formatting function** (mentioned in the previous step) is useful:
- It takes that long string of payload data.
- It breaks it up into multiple, shorter lines.
- This makes it much easier for a human to actually read and understand what's inside the packet.

---

## Step 6: Full Summary — What We Built

Across this whole tutorial series, we built a Python packet sniffer that can:

1. **Listen** to raw network traffic using a socket.
2. **Unpack the Ethernet frame** to find the destination, source, and protocol.
3. **Unpack the IPv4 packet** to find the version, header length, TTL, protocol, source IP, and destination IP.
4. **Check the protocol number** to figure out what type of data is inside:
   - **ICMP** (protocol `1`) — used for network diagnostics.
   - **TCP** (protocol `6`) — used for most regular internet traffic.
   - **UDP** (protocol `17`) — used for things like DNS requests.
5. **Unpack and display** the specific data for whichever protocol was found.
6. **Format long payload data** into multiple readable lines instead of one giant line of text.

---

## Final Thoughts

That's the complete process:
1. Set up a socket to capture raw network data.
2. Break the data down layer by layer (Ethernet → IP → TCP/ICMP/UDP).
3. Display everything in a clean, readable, indented format.

With this foundation, you now understand the basics of how a packet sniffer works in Python, and how data actually travels across a network — from the raw electrical pulses all the way up to readable information like web requests.

---


