# Hacker Holidays 2026 – Day 04: Covert Concierge

## 🎯 Challenge Overview

Day 4 of the **TryHackMe Hacker Holidays 2026** event focused on **network forensics, PCAP analysis, covert communication, and cryptography**.

VERA captured a short packet capture from the Byte Lotus Hotel guest network before the suspicious connection dropped. The traffic appeared to be ordinary HTTP communication, but the repeated requests to port `8080`, unusual request headers, and encoded cookie values suggested that data was being secretly exfiltrated through a covert communication channel.

The objective was to:

* Analyze the provided packet capture.
* Identify the covert communication channel.
* Determine where the exfiltrated data was hidden.
* Reassemble the hidden data in the correct order.
* Decode and decrypt the recovered payload.
* Retrieve the final flag.

---

## 🛎️ Concierge Briefing

> Tiny packets. Odd hours. Suspiciously regular. Someone's smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.
>
> A short capture from the guest network is all VERA could pull before the connection dropped. Somewhere in that traffic, a quiet little errand is running on a loop, and it isn't part of any service the hotel actually offers.

### 📸 @0xMia's Story

> "not me watching my laptop ping some random :8080 address every single second like clockwork 🚩 the request headers are giving 'not a real app' ngl also what is with the crypto 😭 #HackerHolidays"

---

## 🧰 Tools Used

* **Wireshark** – Packet capture and HTTP traffic analysis.
* **CyberChef** – Base64 decoding, hexadecimal conversion, and XOR decryption.

---

## 🔍 Step 1: Analyze the PCAP in Wireshark

The provided packet capture was opened using **Wireshark**.

Based on the clue mentioning communication with a suspicious service running on port `8080`, the following display filter was used:

```wireshark
tcp.port == 8080
```
<img width="1919" height="841" alt="image" src="https://github.com/user-attachments/assets/2c1b6699-af1c-4dd3-9673-6988da0b3310" />

This filtered the capture and revealed repeated HTTP requests from the guest device to:

```text
byte-lotus-hotel.thm:8080
```

The requests appeared to occur repeatedly and contained a suspicious custom client identifier:

```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) ByteLotusClient/1.1
```

The traffic looked like normal HTTP communication at first, but the repeated requests and unusual client behavior indicated that the connection required further investigation.

---

## 🍪 Step 2: Identify the Covert Channel

The HTTP requests contained the following cookie:

```http
Cookie: hotel_sess_state=HA==
```

Additional requests contained different values in the same cookie:

```text
hotel_sess_state=HA==
hotel_sess_state=AA==
hotel_sess_state=BQ==
hotel_sess_state=Mw==
hotel_sess_state=Hg==
...
```

The values were short and ended with `==`, which is commonly associated with **Base64 encoding**.

This revealed that the exfiltrated data was being hidden inside the:

```text
hotel_sess_state
```

HTTP cookie.

The covert communication channel was therefore:

```text
HTTP requests
      ↓
Cookie header
      ↓
hotel_sess_state parameter
      ↓
Small Base64-encoded data chunks
```

---

## 🔐 Step 3: Decode the Cookie Values

Each `hotel_sess_state` value was Base64-decoded.

For example:

```text
HA==
```

Base64-decoded to the byte:

```text
0x1C
```

Another value:

```text
Kw==
```

Base64-decoded to:

```text
+
```

The decoded values did not form readable text. This indicated that Base64 was only an encoding layer and that the recovered data was still encrypted or obfuscated.

---

## 🧩 Step 4: Reassemble the Hidden Data

The cookie values were extracted in chronological order based on the packet timestamps.

After decoding and reassembling the data, the recovered hexadecimal payload was:

```text
1c0005331e7b3a7c17793b173f7c3c2b2079262f17783e2d1a1731783d35
```

The data was not readable as plain text, indicating that an additional cryptographic transformation was required.

---

## 🔓 Step 5: Identify the XOR Key

The expected TryHackMe flag format begins with:

```text
THM{
```

The beginning of the recovered payload was compared with the expected flag prefix.

The XOR operation revealed a repeating key:

```text
H
```

This indicated that the payload had been encrypted using a **single-byte repeating XOR key**.

---

## 🧪 Step 6: Decrypt the Payload in CyberChef

The recovered hexadecimal payload was processed in **CyberChef** using the following recipe:

```text
From Hex
↓
XOR
```

The XOR configuration was:

```text
Key: H
Key encoding: UTF-8
```

After applying the XOR operation, the plaintext flag was recovered.

---

## 🚩 Flag

```text
THM{V3r4_1s_w4tch1ng_0veR_y0u}
```
<img width="1920" height="909" alt="image" src="https://github.com/user-attachments/assets/9a36590e-eeef-4dfa-bb26-e4095a3ca93b" />

---

## 🧠 Attack Chain

```text
Suspicious HTTP traffic
          ↓
Repeated requests to port 8080
          ↓
Inspect HTTP request headers
          ↓
Identify hotel_sess_state cookie
          ↓
Extract Base64-encoded values
          ↓
Decode and reassemble the data
          ↓
Recover the hexadecimal payload
          ↓
Convert from Hex
          ↓
Decrypt using XOR key: H
          ↓
Recover the TryHackMe flag
```

---

## 📚 Key Takeaways

* HTTP traffic can be used as a covert channel for data exfiltration.
* Attackers may hide data inside legitimate-looking HTTP headers such as cookies.
* Repeated requests at regular intervals can indicate automated beaconing or covert data transfer.
* Base64 encoding is not encryption and should be investigated when found in suspicious network traffic.
* Small encoded chunks may need to be extracted and reassembled in chronological order.
* XOR encryption can be identified by comparing known plaintext patterns with encrypted data.
* Wireshark and CyberChef can be combined to investigate network traffic and decode hidden payloads.

---

## 🛡️ Defensive Perspective

Security analysts should monitor for:

* Repeated HTTP requests occurring at fixed intervals.
* Unexpected outbound communication to unusual ports.
* Unusually short or frequently changing cookie values.
* Custom or suspicious user-agent strings.
* Base64-encoded data in HTTP headers.
* HTTP traffic that does not match normal application behavior.
* Potential command-and-control or data exfiltration activity hidden within legitimate protocols.

---

## 🏁 Conclusion

This challenge demonstrated how attackers can hide data inside ordinary-looking HTTP traffic.

By analyzing the PCAP in Wireshark, I identified repeated requests to port `8080` and discovered that the hidden data was stored in the `hotel_sess_state` cookie. The cookie values were Base64-decoded and reassembled in chronological order to recover an encrypted hexadecimal payload.

The payload was then processed in CyberChef and decrypted using a repeating XOR key of `H`, revealing the final TryHackMe flag.

This exercise provided practical experience in:

* Network traffic analysis
* PCAP investigation
* HTTP header inspection
* Covert channel detection
* Data reconstruction
* Base64 decoding
* XOR decryption
* Network forensics
