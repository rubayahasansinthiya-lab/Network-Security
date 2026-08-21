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


