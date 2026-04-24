# Enterprise Data Classification & Zero Trust Storage Architecture

## 🛡️ Azure Infrastructure Security | Microsoft Purview | Compliance Logging

### 📌 Executive Summary
This project demonstrates the design and validation of a cloud-native data protection architecture using **Microsoft Purview** and **Azure**. By shifting away from traditional perimeter security, this implementation focuses on a layered defense model: data classification, strict network isolation via Private Endpoints, and comprehensive audit logging.

By disabling public access and routing all traffic through a private backbone, this architecture ensures data assets are invisible to the public internet. The implementation concludes with a verified audit trail in **Azure Monitor**, satisfying the "Monitoring and Measurement" requirements of **ISO 27001** and **SOC 2**.

---

### 📄 Problem Statement
Organizations storing data in cloud environments often lack proper classification and access controls, leading to increased risk of unauthorized access, data leakage, and regulatory non-compliance. 

In this scenario, sensitive customer data (including PII) is stored in Azure without:
* **Clear data classification**
* **Access restrictions**
* **Network isolation**

This creates high-risk exposure to:
* **Public data access:** "Leaky buckets" accessible via the internet.
* **Insider threats:** Lateral movement and unauthorized viewing of sensitive files.
* **Regulatory Violations:** Non-compliance with privacy regulations such as **Quebec’s Bill 25**.

---

### 🎯 Project Objectives
* **Data Governance:** Classify data assets based on sensitivity (Public, Confidential, PII) using Microsoft Purview.
* **Network Isolation:** Eliminate the public attack surface of Azure Storage using Private Endpoints.
* **Audit & Visibility:** Implement diagnostic logging to track all data plane activities (Read/Write/Delete).
* **Compliance Mapping:** Align technical controls with NIST, ISO 27001, SOC 2, and Quebec Bill 25.

---

### 🏗️ Architecture & Component Overview
* **Governance:** Microsoft Purview (Sensitivity Labeling & Policy Design)
* **Storage Layer:** Azure Blob Storage (Isolated)
* **Network Security:** Virtual Network (VNet) & Private Endpoints
* **Monitoring:** Azure Monitor & Log Analytics (Diagnostic Logging)

---

### 🔐 Part 1: Data Classification Strategy
I defined a classification schema within Microsoft Purview to handle diverse data sensitivity levels:

| Label | Rationale | Technical Control |
| :--- | :--- | :--- |
| **Public** | Low risk; non-sensitive. | No restriction; minimal friction. |
| **Confidential** | Internal business data. | Encryption + Watermarking (Policy side). |
| **PII** | Highly sensitive personal info. | Strict encryption + Regulatory scoping. |

> **Validation Note:** While M365 client-side enforcement (Word/Excel) was out of scope, all encryption policies and label scopes were successfully validated within the Purview control plane.

img/[Labels-List.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Labels-List.png)

img/[Confidential-Label-Settings.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Confidential-Label-Settings.png)

img/[PII-Label-Settings.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/PII-Label-Settings.png)

img/[Label-Policy.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Label-Policy.png)


---

### ☁️ Part 2: Secure Storage & Network Isolation

#### 🚫 Public Access Mitigation
To eliminate "Leaky Bucket" syndrome, public access was disabled at the storage account level. > img/[Public-Access-disabled.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Public-Access-disabled.png)
* **Test:** Attempted to get access to a blob in the portal and via storage explorer.
* **Result:** **Access Denied.** The resource remained unreachable from the public internet. > 
img/[Blob-Access-Denied.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Blob-Access-Denied.png)  ;  img/[Storage-Explorer-Failure.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Storage-Explorer-Failure.png)

#### 🌐 Private Endpoint Implementation
A Private Endpoint was provisioned to give the storage account a private IP address within a secured VNet. > img/[Private-Endpoint.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Private-Endpoint.png)
* **Validation:** Access was tested via a Virtual Machine (VM) located inside the VNet.
* **Result:** Using Azure Storage Explorer, I successfully connected to the storage and performed data operations (Upload/Download), proving that the private path was functional.

---

### 🔍 Part 3: Monitoring & Auditability
A Zero Trust architecture is incomplete without visibility. I implemented Diagnostic Settings to ensure a full audit trail.

* **Configuration:** Enabled `StorageRead`, `StorageWrite`, and `StorageDelete` logs.
* **Destination:** Data streamed to a **Log Analytics Workspace**.

#### Activity Verification via KQL
Using Kusto Query Language (KQL) in Azure Monitor, I verified the telemetry:
* **Audit Success:** Logs confirmed legitimate file operations from the internal IP.
* **Audit Failure:** Logs captured blocked connection attempts, providing the necessary data for incident response. > img/[Logs-Failure.png](https://github.com/Aziz-NG/Data-Classification-Zero-Trust-Storage-Architecture-Project/blob/main/img/Logs-Failure.png)

---

### 📊 Compliance Framework Mapping

| Framework | Control Mapping | Implementation |
| :--- | :--- | :--- |
| **NIST 800-53** | SC-7, AU-2 | Boundary protection and event logging. |
| **ISO 27001** | A.12.4, A.13 | Logging/Monitoring and Network Security. |
| **SOC 2** | Confidentiality | Restricting access via network controls and encryption. |
| **Quebec Bill 25** | PII Safeguards | Protection of PII and auditability of access. |

---

### 🧨 Threat Modeling

| Threat | Control | Result |
| :--- | :--- | :--- |
| **Public Exposure** | Public access disabled | **Blocked** |
| **Data Exfiltration** | Private Endpoint | **Blocked** |
| **Unauthorized Access**| Label Encryption | **Restricted** |

---

### 🧾 Key Takeaways
This project validates that **Defense in Depth** is achieved through the combination of data classification, network cloaking, and robust logging. Even without identity-based RBAC, the use of Private Endpoints significantly reduces the threat landscape by removing the public endpoint entirely, while Azure Monitor provides the continuous oversight required by modern privacy laws like **Quebec's Bill 25**.
