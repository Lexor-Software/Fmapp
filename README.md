# fmapp - Personal Multi-SIM Finance Manager for Ethiopia

**Package ID (Android):** `com.redx.fmapp`
**Current Date for Development Context:** May 11, 2025
**Target Location Context:** Ethiopia

## 1. Project Overview

`fmapp` is a personal Android application designed for a **single user** in **Ethiopia**. The primary goal of the app is to enable this user to efficiently manage their personal finances across multiple SIM cards and their associated financial accounts. This includes tracking income, expenses, and internal fund movements between the user's own accounts.

The app must be tailored to the Ethiopian context, understanding local currency (ETB), telecom providers (e.g., Ethio Telecom, Safaricom Ethiopia), and popular mobile money services (e.g., Telebirr, M-Pesa Ethiopia). A key feature will be on-device OCR (Optical Character Recognition) to extract transaction details from images and PDFs of Ethiopian M-Pesa and Telebirr receipts/statements.

All user data will be stored securely in Firebase and should sync across devices if the user reinstalls or changes their phone, accessible only after re-authentication.

## 2. Core Features

### 2.1. User Authentication (Single User Focus)
* Secure registration and login for a single user account using Email & Password (Firebase Authentication).
* Persistent login sessions.
* Data synchronization upon login on a new/reinstalled device.

### 2.2. SIM Card Management
* Allow the user to Add, View, Edit, and Delete (CRUD) SIM card records.
* **Data for each SIM:**
    * `phoneNumber` (String, unique, e.g., "+2519XXXXXXXX")
    * `simNickname` (String, user-defined)
    * `officialRegisteredName` (String, for transaction matching)
    * `telecomProvider` (String, e.g., "Ethio Telecom", "Safaricom Ethiopia")

### 2.3. Financial Account Management
* CRUD operations for financial accounts, each linked to one of the user's SIM cards.
* **Account Types:** "Bank Account", "Mobile Wallet" (e.g., Telebirr, M-Pesa), "Online Money" (for internal virtual funds).
* **Data for each Account:**
    * `accountName` (String, user-defined)
    * `accountIdentifier` (String, e.g., bank account number, phone number for mobile wallet, user-defined for Online Money)
    * `accountType` (String)
    * `linkedSimId` (Reference to SIM card)
    * `initialBalance` (Number, in ETB)
    * `createdAt` (Timestamp)
* The app must calculate and display the **current balance** for each account.

### 2.4. Transaction Recording & Management
* CRUD operations for transactions.
* **2.4.1. On-Device OCR for Receipts (Key Feature):**
    * User can select an image (from camera/gallery) or PDF of a transaction receipt.
        * *(Refer to provided examples: `Receipt_TE97TFTY19.pdf` for M-Pesa and image.png examples for Telebirr).*
    * Utilize Google ML Kit Text Recognition (on-device) to parse text.
    * Attempt to automatically extract: Transaction Date & Time, Amount (ETB), Payer/Sender info, Payee/Receiver info, Transaction ID/Reference.
        * The parsing logic needs to be robust for formats commonly found in Ethiopian M-Pesa and Telebirr receipts (e.g., fields like "Transaction number", "Paid To", "Trx ID", "Amount", dates in various formats).
    * Present extracted data in an editable form for user review before saving.
    * Store uploaded receipt files in Firebase Cloud Storage, linking them to the transaction record.
* **2.4.2. Manual Transaction Entry:**
    * Full form for manual input of all transaction details. This is the primary method for "Online Money" transactions.
* **2.4.3. Transaction Data Fields (for Firestore):**
    * `userId` (Firebase Auth UID)
    * `affectedAccountId` (ID of the user's financial account)
    * `transactionDate` (Timestamp)
    * `amount` (Number, ETB)
    * `transactionType` (String: "Income/Credit" or "Expense/Debit")
    * `currency` (String: "ETB")
    * `descriptionNotes` (String)
    * `payerSenderRaw` (String, from OCR/manual)
    * `payeeReceiverRaw` (String, from OCR/manual)
    * `referenceNumber` (String)
    * `isInternalTransfer` (Boolean, default: false)
    * `counterpartyAccountId` (String, ID of the *other* user-owned account if `isInternalTransfer` is true)
    * `receiptFileUrl` (String, path in Firebase Storage)
    * `ocrExtractedText` (String, full text from OCR for reference)
    * `createdAt`, `updatedAt` (Timestamps)

### 2.5. Specific "Online Money" Transaction Logic
* "Online Money" accounts are virtual accounts used by the user to represent funds moved between their *own* other accounts (e.g., moving money from a Bank account to an "Online Money for M-Pesa top-up" account before actually sending it via M-Pesa).
* When adding an "Online Money" transaction:
    1.  User selects their "Online Money" account.
    2.  Specifies amount and direction (inflow/outflow for this Online Money account).
    3.  The "other party" is **always** another of the user's registered Mobile Wallets or Bank Accounts.
    4.  These are *always* flagged as `isInternalTransfer = true`.
    5.  The transaction updates balances of *both* user accounts involved.

### 2.6. Internal Transfer Identification
* A transaction is an "Internal Transfer" if both Payer/Sender and Payee/Receiver are identified as belonging to two *different* registered financial accounts owned by the user.
* All "Online Money" transactions (as defined above) are internal transfers.

### 2.7. Dashboard Overview
* Summary of recent/flagged Internal Transfers.
* List of each registered SIM card with its **current total balance** (sum of current balances of all financial accounts linked to it), sorted.
* Total number of SIM cards.
* Number of financial accounts under each SIM.
* Latest 3-5 transactions across all accounts (Amount, concise Sender/Receiver, affected user account name).

### 2.8. Reporting & Summaries
* Generate transaction summaries with filters:
    * By specific SIM card (option to filter by account type under that SIM or include all).
    * Across ALL registered SIMs/accounts.
    * Show only "Internal Transfers."
* Selectable Date Range.
* Summary to show: Total Inflows, Total Outflows, Net Change (all in ETB).
* Detailed list of transactions contributing to the summary.

## 3. Technology Stack

* **Platform:** Android (initial focus)
* **Framework:** Flutter (using Dart)
* **Backend & Cloud Services:** Firebase
    * Authentication: Firebase Authentication (Email/Password)
    * Database: Cloud Firestore
    * File Storage: Firebase Cloud Storage
* **On-Device OCR:** Google ML Kit Text Recognition (via `google_mlkit_text_recognition` Flutter plugin or similar).

## 4. Data Models (High-Level Firestore Structure)

* **`users/{userId}`:**
    * `email`: String
    * `displayName`: String (optional)
    * `createdAt`: Timestamp
* **`simCards/{simId}`:**
    * `userId`: String (owner's Firebase Auth UID)
    * (Fields as listed in 2.2)
* **`financialAccounts/{accountId}`:**
    * `userId`: String
    * (Fields as listed in 2.3, plus `currentBalance` which will be calculated)
* **`transactions/{transactionId}`:**
    * `userId`: String
    * (Fields as listed in 2.4.3)

## 5. Non-Functional Requirements

* **Security:** Firestore Security Rules must ensure a user can only access and manage their *own* data.
* **Performance:** Smooth UI, responsive OCR, efficient data queries.
* **Error Handling:** Robust error handling with clear feedback to the user.
* **Offline First:** Leverage Firestore's offline persistence. New data entered offline should sync when connectivity is restored. File uploads for receipts will need to queue and resume.
* **UI/UX:** Clean, intuitive, and user-friendly, suitable for managing financial data. Emphasis on clarity for ETB amounts and transaction details.

## 6. Key Challenges & Considerations

* **OCR Accuracy & Ethiopian Receipt Variations:** Parsing different receipt formats from M-Pesa and Telebirr, including variations in date formats, field labels, and layout, will be a key challenge. The system needs to be flexible.
* **Balance Calculation Logic:** Ensuring current balances for accounts are updated accurately and efficiently after every relevant transaction.
* **Data Integrity for Internal Transfers:** Correctly debiting one user account and crediting another user account in a single logical operation.

---
This README provides a comprehensive overview for the AI agent.
