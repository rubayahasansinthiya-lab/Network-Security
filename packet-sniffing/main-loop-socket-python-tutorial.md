# How to Build the Main Loop of a Packet Sniffer in Python (Easy English Guide)

This guide explains, in simple English, how to build the **main program** for our packet sniffer. This is the part that actually listens for network data and uses our earlier functions to read it.

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

*This guide is based on a video tutorial about building a packet sniffer in Python. It has been rewritten in simple, easy-to-understand English for beginners.*
