# Reactive Network Hackathon - Content Verification Platform (No-Code Track)

*Note: This repository outlines the approach only and does not include a full dApp implementation. Any code snippets provided should be considered as pseudo-code.*

---

### Contents
- [Concept Overview](#concept-overview)
- [Scenario Example](#scenario-example)
- [Demo Video](#demo-video)
- [Implementation Outline](#implementation-outline)
- [Workflow Summary](#workflow-summary)
  - [Listing Attempt](#listing-attempt)
  - [Initial Metadata Handling](#initial-metadata-handling)
    - [Verification Heirarchy](#verification-heirarchy)
  - [Verification Process Initiation](#verification-process-initiation)
  - [Verification Stages](#verification-stages)
  - [Verification Categorization](#verification-categorization)
  - [Completion and Status Update](#completion-and-status-update)
- [Modular Verification System](#modular-verification-system)

---

### Concept Overview

Content verification is essential to ensure that uploaded content aligns with platform standards. This platform is divided into two primary modules:

1. **Content Upload and Listing**
2. **Content Verification**

Using Reactive Smart Contracts (RSCs), these modules can operate independently, allowing for flexible combinations. A listing module, for instance, can connect to different verification modules based on suitability. Similarly, a verification module can service multiple listing modules.
This creates an asynchronous, event driven and modular verification process.

---

### Scenario Example

Consider an NFT Marketplace focused on travel-related content. Users list travel NFTs, which may include images or essays. For each listing, the NFT’s metadata is parsed to extract URLs and text, which must then be verified to ensure relevance to the travel theme. 

Too much manual verification could cause delays, so the platform employs automated verification methods. Users can challenge any verifications they deem incorrect, triggering a manual verification process if needed. Once verified, relevant NFTs appear on the dashboard, filtering out non-travel content (such as art or music).

The Verification process has the following **heirarchy**:
   Ai -> Manual -> Subject Matter Expert.

#### Verification Process Overview

- **Automated Initial Verification**: Speeds up the approval process by using AI to quickly confirm that NFTs are travel-related, filtering out non-relevant content like art or music.
- **Community-driven Manual Verification**: If a listing's verification status is contested by users, the item moves to **manual verification**. A community-driven approach allows any three independent users to collectively verify the content, each receiving a small payment per verified task. If all three users agree on the status, it becomes the new verification outcome.
- **Final SME (Subject Matter Expert) Review**: Provides an expert (expert belongs to the NFT Marketplace org) review for contested cases, maintaining high standards and accuracy in verification.

As we can see there is a lot going on in the verification process, so it makes sense to isolate it into a separate module so that in future more complex workflows can be accomodated as the process evolves.


---

#### Demo Video
[Watch Video: **All code snippets are to be treated as pseudo code.**](https://www.loom.com/share/c95dfb179d30416591734c536dfdd6cf?sid=51e58c8e-b01a-4477-ac89-dea16373a098)
https://www.loom.com/share/c95dfb179d30416591734c536dfdd6cf?sid=51e58c8e-b01a-4477-ac89-dea16373a098

---

### Implementation Outline

*Watch the video for better understanding.*

*Note: This repository provides the approach only, not the complete dApp implementation. Code snippets are illustrative, should be considered as pseudo code.*

<img width="988" alt="Screenshot 2024-11-04 at 6 50 32 AM" src="https://github.com/user-attachments/assets/48d827b5-8a7c-4387-b8ea-588848dee2a5">

#### Workflow Summary

1. **Listing Attempt**  
   The user lists their NFT on the platform. The listing is recorded on the contract but remains hidden on the dashboard until verification is complete.

2. **Initial Metadata Handling**  
   NFT details are stored with initial verification data: attempts set to 0, status as "InProgress," and `isMaxVerificationDone` as false.

   ##### Verification Heirarchy
   Ai -> Manual -> Subject Matter Expert

   Most of the Verification will be carried out by AI as manual efforts for a large number of listings is tedious. When challenged, the verification process will repeat, but     this time one step higher in the heirarchy, which the verification contract comes to know of using the `attempts` value.

   `isMaxVerificationDone` - When the verification is carried out by the top verification entity in the heirarchy, i.e.  Subject Matter Expert, it cannot be challenged any further.

4. **Verification Process Initiation**  
   Metadata is sent to `RSC1` for verification. An event is emitted, which is indexed by the listing dApp to display NFT's verification status to the user.

5. **Verification Stages**  
   `RSC1` subscribes to this event and calls `attemptVerificationCallback` on the `VerificationContract`. Verification is carried out through one of AI, manual, and subject-matter expert checks based on the number of attempts.

6. **Verification Categorization**  
   Each verification method (AI, manual, or expert) emits an event. The verification dApp indexes these events to display jobs accordingly. AI verification operates on batches via Chainlink Keepers calling an off chain Ai verification api via Chainlink External Adapters, while manual and expert jobs are picked by respective parties.

7. **Completion and Status Update**  
   Once verified, an event with the results is emitted. `RSC2` subscribes to this event and calls `verificationStatusUpdateCallback` on the `ListingContract`. If verified, the NFT becomes visible on the dashboard. Users may challenge the results, escalating to higher verification levels as needed.

---

While attmpting to verify the Listing Contract is the origin and the Verification Contract is the Destination and vice versa when updating the verification status.

<img width="818" alt="Screenshot 2024-11-04 at 7 33 36 AM" src="https://github.com/user-attachments/assets/1e28480b-ba28-427b-907f-eb65ca70859d">

---

### Modular Verification System

To replace the verification module:
- Update the subscriber in `RSC2` and the callback in `RSC1` to direct to the new verification module.
  
Organizations offering a standalone verification service can expose their module by connecting users’ RSCs with a provided listing dApp ID. This ID enables `RSC2` to filter events and trigger appropriate callbacks on the listing service.

---

This approach enables a scalable and modular content verification system using RSCs, ensuring compliance and flexibility in content verification processes across decentralized applications.
