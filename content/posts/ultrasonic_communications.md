+++
title = "Ultrasonic Communications"
date = "2026-03-14T18:41:09+01:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = ""
authorTwitter = "" #do not include @
cover = ""
description = "Can computers talk using sound instead of Wi-Fi? This project explores ultrasonic communication by encoding data into high-frequency audio signals and decoding them back into text."
showFullContent = false
readingTime = false
hideComments = false
+++

# Ultrasonic Data Communication Protocol

Recently I started experimenting with an unusual way for computers to communicate: **sound**.

Instead of sending data through Wi-Fi, Bluetooth, or Ethernet, this project demonstrates how information can be transmitted using **ultrasonic frequencies** — sound waves that are near or above the upper limit of human hearing.

The idea is simple:

> Convert digital data into high-frequency sound, play it through a speaker, and decode it using a microphone.

This effectively creates a **software-defined acoustic modem**.

---

## Why This Is Interesting

Most communication systems rely on traditional radio technologies.

However, computers already contain everything needed for acoustic communication:

- Speakers
- Microphones
- Digital signal processing capabilities

By using frequencies around **17–19 kHz**, the signal becomes:

- Almost inaudible to humans
- Detectable by microphones
- Easy to analyze using signal processing

This opens the door to interesting experiments — and also some **security implications**.

---

## How the System Works

The system is composed of three main stages:

1. Encoding text into binary
2. Modulating binary into ultrasonic frequencies
3. Decoding the signal back into the original message

The modulation method used is **Binary Frequency Shift Keying (BFSK)**.

```
0 → 17 kHz
1 → 19 kHz
```

Each bit is represented by a short sine wave at one of these frequencies.

---

## Step 1 — Converting Text to Binary

First we convert a message into a stream of bits.

```python
def text_to_bits(text):
    """Convert string to a list of bits."""
    bits = []
    for char in text:
        # 1. Get ASCII value (e.g., 'A' -> 65)
        ascii_val = ord(char)
        
        # 2. Convert to binary (e.g., 65 -> '01000001')
        # [2:] removes the '0b' prefix, zfill(8) ensures it's always 8 bits
        bin_val = bin(ascii_val)[2:].zfill(8)
        
        # 3. Add individual integers to our list
        bits.extend([int(b) for b in bin_val])
    return bits
```

Example:

```
Text: "S"
ASCII: 83
Binary: 01010011
```

---

## Step 2 — Generating Ultrasonic Signals

Each bit is converted into a sine wave.

```
17 kHz → bit 0
19 kHz → bit 1
```

```python
import numpy as np
from scipy.io import wavfile

# Configuration
FS = 44100          # Sample rate (standard)
FREQ_0 = 17000      # Hz for bit '0'
FREQ_1 = 19000      # Hz for bit '1'
BIT_DURATION = 0.1  # Seconds per bit (start slow to ensure it works!)
FILENAME = "ultrasound_data.wav"
```

Now we generate the signal.

```python
def generate_ultrasound(bits):
    full_signal = np.array([], dtype=np.float32)
    
    # 1. Create a "Time Window" for a single bit
    # FS * BIT_DURATION gives us the number of data points (samples) per bit
    t = np.linspace(0, BIT_DURATION, int(FS * BIT_DURATION), endpoint=False)
    
    for bit in bits:
        # 2. Select Frequency (The "Modulation" step)
        freq = FREQ_1 if bit == 1 else FREQ_0
        
        # 3. Generate the Sine Wave
        # Formula: A * sin(2 * pi * f * t)
        tone = 0.5 * np.sin(2 * np.pi * freq * t)
        
        # 4. Append to the master recording
        full_signal = np.append(full_signal, tone)
        
    return full_signal
```

Finally we save the encoded data as an audio file.

```python
message = "SECRET"
bit_sequence = text_to_bits(message)
audio_data = generate_ultrasound(bit_sequence)

wavfile.write(FILENAME, FS, (audio_data * 32767).astype(np.int16))
```

This file now contains the **encoded ultrasonic transmission**.

---

## Step 3 — Visualizing the Signal

To understand what the transmission looks like, we can analyze the file using **MATLAB**.

A useful tool here is a **spectrogram**, which shows:

- Time (x-axis)
- Frequency (y-axis)
- Intensity (color)

```matlab
[y, fs] = audioread('ultrasound_data.wav');

window = round(fs * 0.05);
noverlap = round(window * 0.5);
nfft = 1024;

spectrogram(y, window, noverlap, nfft, fs, 'yaxis');
ylim([15 22]);
title('Spectrogram of Ultrasonic Transmission');
```

When you run this script you should see two alternating bands around:

```
17 kHz
19 kHz
```

Each band represents a **binary bit**.

---

## Step 4 — Decoding the Message

Once the bitstream is recovered, decoding is straightforward.

We simply convert groups of 8 bits back into ASCII characters.

```python
def bits_to_text(bit_list):
    """Convert a list of bits back into a string."""
    chars = []
    # Loop through the list 8 bits at a time
    for i in range(0, len(bit_list), 8):
        # 1. Grab an 8-bit chunk
        byte = bit_list[i:i+8]
        
        # 2. Join the integers into a string (e.g., [0,1,0...] -> "010...")
        bin_str = "".join(str(b) for b in byte)
        
        # 3. Convert binary string to integer, then integer to character
        char = chr(int(bin_str, 2))
        chars.append(char)
        
    return "".join(chars)
```

The original message can then be reconstructed.

---

## Security Implications

While this project is mainly an experiment in signal processing, similar techniques have been explored in **cybersecurity research**.

In particular, they can be used in **air-gapped environments**.

An **air-gapped computer** is completely isolated from networks:

- No Wi-Fi
- No Bluetooth
- No Ethernet connection

These systems are often used in:

- Nuclear facilities
- Military networks
- Critical infrastructure
- High-security financial systems

Because they cannot communicate over the internet, attackers have explored **side channels** to exfiltrate data.

One possible technique:

1. Malware on an air-gapped computer encodes data into ultrasound
2. The computer's speaker transmits the signal
3. A nearby device (phone, microphone, compromised camera) records it
4. The attacker decodes the signal later

This type of attack is extremely difficult to deploy in practice, but it highlights an important concept:

> Even isolated systems may leak information through unexpected physical channels.

---

## What I Learned From This Project

This project ended up touching multiple fields:

- Digital signal processing
- Acoustic communication
- Software-defined modems
- Cybersecurity research

It also demonstrates how **simple hardware can create unexpected communication channels**.

What started as a fun experiment became a small proof-of-concept showing how data can move through **sound instead of radio waves**.

---

## Future Improvements

Some interesting directions for future work include:

- Real-time decoding from microphone input
- Error correction codes
- Synchronization signals
- Higher bitrate modulation
- Fully automated receiver pipeline

---

## Final Thoughts

Modern computers are full of sensors and actuators that can be repurposed in creative ways.

Speakers and microphones are just one example.

Exploring these unconventional channels is not only technically interesting — it also helps us understand **how complex systems can behave in unexpected ways**.
