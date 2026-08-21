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

