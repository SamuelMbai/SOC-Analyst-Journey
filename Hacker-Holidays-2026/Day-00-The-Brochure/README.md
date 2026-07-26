# 🏖️ Hacker Holidays 2026 — Day 0: The Brochure

> **Platform:** TryHackMe  
> **Difficulty:** Beginner  
> **Category:** OSINT  
> **Room:** The Brochure  
> **Tools Used:** Kali Linux · ExifTool · CyberChef · Instagram

---

## ⚠️ Spoiler Warning

> This write-up contains the complete solution to the **Hacker Holidays 2026 — Day 0: The Brochure** room.

---

## 📖 Introduction

The objective of this investigation was to analyze the **Byte Lotus Resort** brochure, identify clues related to its suspiciously perfect AI-generated hero image, use OSINT techniques to trace the associated social-media accounts, and recover the flag.

The investigation involved:

- 🖼️ Analyzing the provided brochure image
- 🧪 Inspecting image metadata
- 🔍 Identifying clues within the brochure
- 🌐 Tracing social-media accounts
- 🧩 Reconstructing Base64-encoded fragments
- 🚩 Decoding the final message

---

## 🔍 1. Analyzing the Brochure Image

I started by examining the provided brochure image:

<img width="726" height="934" alt="The Brochure" src="https://github.com/user-attachments/assets/4838aa0a-2228-48f6-85df-7606d72bc942" />

The brochure contained several important clues, including:

- **`CONCIERGE — VERA`**
- **`Find us on Instagram`**
- The suspiciously perfect, AI-generated appearance of the hero image

The AI-like appearance of the image suggested that the investigation would involve identifying the source and following the digital trail behind it.

---

## 🧪 2. Inspecting the Image Metadata

I used **ExifTool** to inspect the image metadata:

```bash
exiftool -a -u -g1 thebrochure.png
```

The output contained standard PNG information, including:

- Image dimensions
- Bit depth
- Color type
- Compression
- Rendering information

However, there was no useful author, creator, or obvious hidden flag information in the metadata.

<img width="1195" height="529" alt="ExifTool metadata output" src="https://github.com/user-attachments/assets/89170181-43d6-4563-abbb-c56072af9952" />

---

## 🌐 3. Following the OSINT Trail

The brochure contained a clue pointing toward Instagram:

> **"Find us on Instagram or not."**

Searching for the **Byte Lotus Resort** led to the account:

```text
@thebytelotusresort
```

<img width="961" height="671" alt="Byte Lotus Resort Instagram account" src="https://github.com/user-attachments/assets/7be30c78-44fd-4841-9966-94e7e93e2acd" />

The brochure also contained the clue:

```text
CONCIERGE — VERA
```

This led to the related account:

```text
@veratheconcierge
```

---

## 🕵️ 4. Investigating Vera's Account

The account **`@veratheconcierge`** contained three posts with encoded strings:

```text
VEhNe1YzckBzX2FD
QzB1bnRfaDRzX2lz
M25fZjB1bmQhfQ=
```

These appeared to be **Base64-encoded fragments**.

Instead of decoding the fragments separately, I collected all three pieces and reconstructed them in their correct order.

The fragments were then combined into one Base64 string.

<img width="1309" height="617" alt="Base64 fragments on Instagram" src="https://github.com/user-attachments/assets/97e91afd-45f2-464d-9ef0-154f094f5172" />

---

## 🧩 5. Reconstructing the Encoded String

The three fragments were joined together:

```text
VEhNe1YzckBzX2FDQzB1bnRfaDRzX2lzM25fZjB1bmQhfQ=
```

The combined string was then decoded using **CyberChef**.

### Decoding Process

```text
Input → From Base64 → Output
```

<img width="1919" height="756" alt="CyberChef Base64 decoding" src="https://github.com/user-attachments/assets/9d616d8d-04ec-483a-a99d-76eb8d57e13f" />

---

## 🚩 6. Flag

The decoded message revealed the flag:

```text
THg{V3r@s_aCC0unt_h4s_b33n_f0und!}
```

---

## 🧠 Key Takeaways

This challenge demonstrated several fundamental OSINT and basic data-decoding techniques:

- 🔍 Examining image metadata using **ExifTool**
- 🖼️ Identifying clues embedded in images
- 🌐 Following social-media breadcrumbs
- 🔗 Connecting related accounts
- 🔐 Recognizing Base64 encoding
- 🧩 Reconstructing fragmented encoded data
- 💻 Decoding data using **CyberChef** and the **Linux terminal**

The main lesson from this challenge was that the flag was not necessarily hidden directly inside the original image. Instead, the image acted as the starting point for a broader OSINT investigation.

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| 🐉 Kali Linux | Investigation environment |
| 🧪 ExifTool | Image metadata analysis |
| 🔐 CyberChef | Base64 decoding |
| 📸 Instagram | OSINT and social-media investigation |
| 🔤 Base64 | Encoding format identified during the investigation |

---

## ✅ Conclusion

The investigation began with a suspiciously perfect resort brochure image.

By analyzing the clues in the brochure and following the social-media trail, I discovered the **`@veratheconcierge`** account.

The account contained three Base64-encoded fragments. After collecting, reconstructing, and decoding them, I recovered the final flag.

> **Investigation complete. 🕵️‍♂️🏖️**
