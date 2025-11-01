# Qubic Incubation Program Application: Data-Ledger

**Project Name:** Data-Ledger **Application Date:** November 1, 2025 **Applying Team/Organization:** Data-Ledger Team **Contact Information:** \*@gmail.com

---

## 1\. Project Summary

### 1.1. Overview

**LedgerSC** is a low-level data ledger smart contract (SC) designed to permanently record off-chain data and transactional events onto the Qubic Network.

By eliminating the significant cost, time, and risk associated with developing, auditing, and approving a custom SC, LedgerSC enables a wide range of applications—both decentralized (dApps) and centralized systems—to seamlessly leverage the Qubic Network for verifiable record-keeping. This accelerates the adoption and growth rate of the Qubic ecosystem.

### 1.2. Problem Statement

For any application that requires verifiable, immutable, and censorship-resistant records, the Qubic Network offers an ideal solution. However, integrating this functionality currently presents a high barrier to entry:

1. **Custom SC Development is Costly and Risky:** To utilize the `payload` field in Qubic transactions for custom data storage, an application must deploy its own Smart Contract. This process is costly, time-consuming, requires specialized C++ development expertise, and carries the risk of rejection.  
2. **Financial Burden for Simple Needs:** For applications that only need basic data recording without complex logic, the requirement to develop and fund a full SC is disproportionate to the need.

This high friction prevents many potential users from integrating with the Qubic Network for simple data recording.

### 1.3. Solution: LedgerSC

LedgerSC addresses this problem by providing a **universal, minimal, and fee-based data recording service**.

Instead of developing and maintaining a custom SC, any application can utilize LedgerSC's single function to securely record its data on-chain for a fixed, minimal fee. The application compresses and formats its data according to its own internal schema, sends it to LedgerSC, and LedgerSC permanently processes and stores the data in the transaction payload.

This approach offers the following benefits:

* **Low Barrier to Entry:** Developers can instantly access immutable record-keeping without any SC development.  
* **Cost-Effective:** A fixed, minimal fee replaces the high cost of custom SC development and maintenance.  
* **Speed and Certainty:** Integration is immediate, bypassing the lengthy and uncertain SC approval process.  
* **Universal Compatibility:** The stateless and schema-agnostic design ensures compatibility with any application (dApp, centralized service, IoT device, etc.) that needs to record data. LedgerSC maintains an append-only ledger of payloads but does not interpret their content; from the dApp perspective, the contract is stateless.

---

## 2\. Technical Implementation

### 2.1. Smart Contract (SC) Details

| Detail | Value |
| :---- | :---- |
| **SC Name** | LedgerSC |
| **Platform** | Qubic Network |
| **Language** | Qubic's native C++ based smart contract environment |
| **Key Features** | Fully on-chain data recording, Optimized for Qubic's 1024-byte transaction input limit, Minimalistic and Stateless design, Universal compatibility. |

### 2.2. Smart Contract Constants

LedgerSC operates under a set of protocol-defined network limits and governance-controlled economic parameters that determine how records are accepted, stored, and monetized.

| Constant | Value | Source/Control | Description |
| :---- | :---- | :---- | :---- |
| `MAX_PAYLOAD_SIZE` | 1024 bytes | Qubic protocol (network) | The absolute maximum size for the transaction input (Qubic network limit). |
| `RECORDING_FEE` | 10000 QUBIC | LedgerSC governance (shareholders) | Fixed fee required per recording operation. |

### 2.3. Smart Contract Functions and Operation

LedgerSC is a lightweight, append-only service. Its sole purpose is to validate the fixed recording fee and payload size, then permanently store the provided payload on-chain without interpreting its contents.

In order to fulfill its purpose as an immutable ledger and enable on-chain revenue distribution, LedgerSC exposes two primary functions: one for recording data and one for executing distribution. Additional helper functions may be included internally to support monitoring, reporting, or governance-related checks, but these remain secondary and do not alter the stateless, append-only nature of the contract.

#### `function record(payload)`

**Description:** Registers a new, permanent record entry for the calling application (issuer). Stores the raw `payload` exactly as provided. LedgerSC does not parse or interpret the remaining data.

**Operation Flow:**

1. **Fee Validation:** Checks if the transaction includes exactly `RECORDING_FEE` and if the sender has sufficient funds.  
2. **Size Validation:** Checks if the `payload` length is `<= MAX_PAYLOAD_SIZE`   
3. **Data Storage:** If validations pass, the `payload` is appended to the append-only ledger in immutable form.

**Response:** Returns true on success, false on failure. Optionally, an error code may be returned to indicate the cause of failure.

**Notes:**

* LedgerSC does not parse, interpret, or enforce any schema on stored payloads.  
* The contract is **stateless** and **append-only**, ensuring chronological integrity and minimal on-chain footprint.  
* **Off-Chain Interpretation:** The application is solely responsible for reading its own recorded data from the ledger and parsing it according to its pre-defined schema.

### `function distribute()`

**Description**: Performs the revenue distribution according to the predefined allocation:  
Qubic Ecosystem Fund, Shareholders, Burn, Project Management

**Operation**: The contract executes distribution when the internally defined distribution conditions are met, and transfers the corresponding portions to the designated destinations in accordance with governance rules.

### 2.4. Application Data Flow

This flow illustrates how an application integrates with **LedgerSC**:

1. **Event Selection:**  
   The application decides that a specific event requires an immutable on-chain record.

2. **Data Preparation:**  
   The application formats and compresses the relevant metadata (e.g., tx\_id, operation\_type, user\_id, timestamp) into a byte array.

   The final payload size must be `≤ MAX_PAYLOAD_SIZE`

3. **SC Call:**  
   The application calls the `LedgerSC.record(payload)` function, including exactly `RECORDING_FEE` in the transaction.

4. **On-Chain Recording:**  
   LedgerSC validates the fee and payload size, then permanently writes the full payload to the Qubic ledger.

5. **Response Handling:**  
   The application may optionally parse an error code to identify the specific reason for failure.

6. **Off-Chain Retrieval & Parsing:**  
   When needed, the application retrieves all transactions sent to LedgerSC from its address and interprets the payload based on its internal schema. LedgerSC does not parse or enforce any structure on the stored data.

This clear separation of concerns ensures **LedgerSC** remains universal while granting applications full flexibility over their data schema.

2.5. SC Capacity Extension Mechanism

The Qubic Network imposes a fixed data capacity limit of **1024MB** per Smart Contract (SC), with a maximum transaction payload size of **1024 bytes**. Given these constraints, the initial LedgerSC is estimated to accommodate approximately **1 million** transaction payloads before reaching its capacity limit.

To ensure the long-term viability and scalability of the LedgerSC service, we have designed a **Transparent Capacity Extension Mechanism**:

**1\.  Capacity Monitoring:** The initial LedgerSC will include internal logic to monitor its remaining data capacity.

**2\.  Auxiliary SC Deployment:** When LedgerSC approaches its 1024MB limit, a new, dedicated **Auxiliary Storage SC** will be deployed. This Auxiliary SC's sole function will be to provide its 1024MB storage capacity exclusively to LedgerSC, only accepting data transactions initiated by the LedgerSC address.

**3\.  LedgerSC V2 Update:** The existing LedgerSC will receive a **V2 update**. This update will integrate the address of the newly deployed Auxiliary SC, allowing LedgerSC to seamlessly redirect its data storage operations to the Auxiliary SC when its own capacity is exhausted.

**4\.  Transparent Operation:** Crucially, this mechanism is **transparent** to end-users and applications. Applications will continue to interact solely with the original LedgerSC address and its ‘record(payload)’ function, ensuring no breaking changes and a virtually limitless data recording capacity for the service.

This robust solution is a critical component of the project's design, guaranteeing uninterrupted service and demonstrating a proactive approach to network limitations. Furthermore, the design is **future-proof:** should the Qubic Network increase the data capacity limit for Smart Contracts, the LedgerSC system will be seamlessly adapted to leverage the expanded capacity.

---

## 3\. Team Overview

The LedgerSC project will be developed by an experienced developer. The roles and expertise of the team members are as follows:

| Role | Team Member | Expertise and Responsibilities |
| :---- | :---- | :---- |
| **Smart Contract Developer** | Poly | In-depth knowledge of Qubic network-specific smart contract development. Responsible for all on-chain logic, transaction management, and deployment. Proficient in Qubic's C++ based SC environment. |
| **Project Manager & Coordinator** | Ozeron7 | Overall coordination, management of development processes, and communication. Post-deployment, responsible for gathering feedback for future enhancements, managing the revenue distribution for project management share, and overseeing ongoing project operations. |

---

## 4\. Milestones and Funding Request

This application requests funding for the development and initial deployment of the LedgerSC Smart Contract.

**Total Requested Grant Amount:** 8000 USD (≈6.7B Qus @ 0.0000012) **Total Project Duration:** 4 weeks

| Milestone ID | Description | Deliverable | Funding % |
| ----- | :---- | :---- | ----- |
| M1 | Design & Specification | Detailed technical specification: data flow, function logic, validation rules, security considerations, design of the transparent capacity extension mechanism, and edge case handling. | 20 |
| M2 | Core Smart Contract Development | Fully developed and audited LedgerSC code (C++). Includes the record(payload) and distribute() functions, capacity monitoring logic, and the V2 update mechanism for Auxiliary SC integration. Initial test suite. | 30 |
| M3 | Deployment and Integration Setup | Successful deployment of LedgerSC to the Qubic Testnet. Development of comprehensive documentation and a reference application demonstrating off-chain retrieval and parsing. | 20 |
| M4 | Testing, Review, and Final Submission | Comprehensive end-to-end testing of the SC and scaling mechanism. Final security review and submission of all project assets (code, documentation, final report) to the Qubic Incubation Program. | 30 |

---

## 5\. Budget and Expenses

The estimated budget and expense items for the development and maintenance of the Data-Ledger project are detailed below. This budget covers all costs necessary for the successful completion and long-term sustainability of the project.

### 5.1. Development Costs

| Role | Duration (Weeks) | Weekly Rate (USD) | Total Cost (USD) |
| :---- | :---- | :---- | :---- |
| Smart Contract Developer | 4 | 2000 | 8000 |
| **Total Development Cost** | **4** | **2000** | **8000** |

## 5.2. Operational Expenses

**Project Management:** Data and feedback tracking, coordination, and project management activities for development purposes, after the system becomes operational.  
---

## 6\. Sustainability and Revenue Model

The long-term sustainability of LedgerSC is secured through a transparent, smart contract-based revenue model. The fixed `RECORDING_FEE` serves as both operational coverage and a necessary spam-prevention mechanism.

**Spam-prevention:** Ensures spam-prevention by requiring a fixed recording fee per payload.

### 6.1. Revenue Distribution

The fee collected from each successful recording operation is automatically distributed according to the following model:

| Destination | Percentage | Purpose |
| :---- | :---- | :---- |
| **Qubic Ecosystem Fund** | 35% | Supports the overall development and growth of the Qubic ecosystem. |
| **Project Shareholders** | 30% | Distributed proportionally to stakeholders who invest in or contribute to the project. |
| **Burn** | 30% | Used to reduce the total Qubic supply, supporting deflationary mechanisms. |
| **Project Management** | 5% | It is allocated for ongoing project management, maintenance, and development. |
| **Total** | **100%** |  |

This model ensures that the project not only achieves self-sustainability but also provides a continuous, deflationary contribution to the overall health of the Qubic ecosystem.

---

## 7\. Commitment and Open-Source

### 7.1. Project Commitment

The LedgerSC team is committed to maintaining and supporting the project for a minimum of two years following its successful deployment. This commitment includes:

* **Ongoing Maintenance:** Ensuring the SC remains functional and compatible with Qubic network updates.  
* **Bug Fixes:** Promptly addressing any bugs or issues that may arise.  
* **Feature Enhancements:** Continuously evaluating new features based on community feedback.  
* **Community Support:** Providing technical support and maintaining comprehensive documentation.

### 7.2. Open-Source Commitment

All Smart Contract code for the LedgerSC project will be released as **open-source** under the **MIT License** upon completion of development. This commitment encourages community contributions, ensures transparency, and fosters trust within the Qubic developer community.

---

## 7\. Conclusion

The LedgerSC project offers a technically sound and urgently needed solution that drastically lowers the entry barrier for applications seeking immutable, verifiable record-keeping on the Qubic Network. By abstracting the complexity and risk of custom SC development into a simple, fee-based service, LedgerSC will significantly accelerate the adoption of the Qubic Network by a diverse range of applications.

We believe that with the support of the Qubic Incubation Program, LedgerSC will become a foundational utility, providing a faster, more flexible, transparent, and efficient experience for developers, and delivering long-term value to the Qubic ecosystem.

Thank you for your consideration.  
