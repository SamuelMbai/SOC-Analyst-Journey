# Hacker Holidays 2026 — Day 06: The Hidden Account

## Category

**OSINT (Open-Source Intelligence)**

## Room Overview

Day 6 of the **Hacker Holidays 2026** challenge focused on using publicly available information to identify and locate a hidden online account.

The challenge began with a screenshot of a private conversation captured at the Byte Lotus Hotel breakfast terrace. The conversation contained several clues, including an email address and a reference to a free online service that “started with a G.”

The objective was to analyze the conversation carefully, extract the relevant identifying information, locate the hidden account, and retrieve the flag.

---

## Objectives

- Analyze the provided conversation
- Extract the relevant clues
- Identify the online service referenced in the conversation
- Locate the hidden account
- Decode the discovered message
- Submit the flag

---

## Task File Analysis

The provided screenshot contained a conversation between two Byte Lotus Hotel guests.

The most important details were:

- The guest's name: **Lambo**
- The email address: `lambobytelushotel@gmail.com`
- A reference to a **free online tool**
- The tool allowed users to upload a profile and link other social media accounts
- The service name “started with a G”

The story posted by **@0xMia** also contained an important hint:

> “Y'all need to actually READ what they said, not just skim it.”

This suggested that the wording in the conversation contained the key to identifying the service.

<img width="519" height="346" alt="image" src="https://github.com/user-attachments/assets/fab14c6d-c014-4cb4-a5be-54745b57cd82" />


---

## Identifying the Online Service

The conversation described a free service that:

- Uses email addresses
- Allows users to create public profiles
- Supports profile images
- Can link to other online accounts
- Starts with the letter **G**

These clues pointed to **Gravatar**.

Gravatar is an online profile and avatar service that associates profile information with an email address.

---

## Searching the Email Address

Using the email address extracted from the conversation:

`lambobytelushotel@gmail.com`

I searched for the associated Gravatar profile.

The search successfully revealed a profile belonging to:

**Lambo**

The profile contained the following message:

> Funny thing about email hashes, they follow you places you didn't expect. Glad you found the right corner of the internet! Here is your prize:
>
> `VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9`

<img width="331" height="269" alt="image" src="https://github.com/user-attachments/assets/b1f705f6-ae9d-4214-8571-1dc6715bd7bd" />


---

## Analyzing the Encoded Message

The discovered string was:

`VEhNe1MzY3JlVF9QcjBmaWwzX0g0c19iMzNuX0lkZW50MWZpM2R9`

The characters and structure suggested that the message was encoded using **Base64**.

Base64 is an encoding method commonly used to represent data using readable ASCII characters. It is encoding rather than encryption, meaning the original content can be recovered by decoding it.

---

## Decoding the Message

The Base64 string was decoded using CyberChef:

The output was:

`THM{S3cr3T_Pr0fil3_H4s_b33n_Id3nt1fi3d}`

<img width="959" height="391" alt="image" src="https://github.com/user-attachments/assets/a9eafe1e-97f1-47a2-b594-75dd0eb2f484" />


---

## Flag

`THM{S3cr3T_Pr0fil3_H4s_b33n_Id3nt1fi3d}`

---

## Investigation Path

    Conversation Screenshot
              ↓
    Extracted the Email Address
              ↓
    Analyzed the Clue About a Tool Starting With “G”
              ↓
    Identified Gravatar
              ↓
    Located Lambo's Profile
              ↓
    Found the Base64-Encoded Message
              ↓
    Decoded the Message
              ↓
    Retrieved the Flag

---

## Key Takeaways

- Small details in conversations can reveal valuable identifying information.
- Email addresses can be linked to public online profiles.
- OSINT investigations often involve connecting multiple clues across different platforms.
- Profile services can expose usernames, social media links, biographies, and other public information.
- Encoded strings should be analyzed rather than treated as random text.
- Base64 encoding is commonly encountered during cybersecurity investigations and CTF challenges.
- Carefully reading the wording of a clue can be more important than quickly scanning it.

---

## Tools and Techniques Used

- Manual conversation analysis
- Email-based OSINT
- Gravatar profile lookup
- Base64 decoding
- CyberChef

---

## Conclusion

This challenge demonstrated how seemingly harmless information can be used to discover an online identity.

The email address found in the conversation led to a public Gravatar profile. By carefully interpreting the clue about a free tool beginning with the letter **G**, the hidden profile was identified. The profile contained a Base64-encoded message, which was decoded to reveal the final flag.

The challenge highlighted the importance of understanding how digital identities can be connected across online platforms and how publicly available information can be used during an OSINT investigation.

---

## Disclaimer

This write-up was created for educational purposes as part of the **TryHackMe Hacker Holidays 2026** challenge. All activities were performed in an authorized training environment.
