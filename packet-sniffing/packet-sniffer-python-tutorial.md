# How to Make a Packet Sniffer in Python

This guide explains, in simple English, how to start building a **packet sniffer** (also called a **network sniffer**) using Python. A packet sniffer is a tool that looks at the data moving across a network.

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

*This guide is based on a video tutorial about building a packet sniffer in Python. It has been rewritten in simple, easy-to-understand English for beginners.*
