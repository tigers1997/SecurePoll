# SecurePoll 🗳️
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

**SecurePoll** is an open-source, privacy-preserving polling platform designed for economists and researchers. It solves a critical problem in modern data collection: validating the humanity and nationality of respondents without invading their privacy or collecting Personally Identifiable Information (PII).

Instead of relying on IP addresses, GPS tracking, or centralized user accounts, SecurePoll utilizes **local-first hardware attestation** and **Mobile Country Code (MCC) seasoning** to generate cryptographically signed trust scores.

## 🧠 The Hardware-Backed Trust Model

Our architecture ensures that the backend server never knows *who* a user is, only that they are a real human using a genuine device, and that they meet the geographic requirements for the poll.

![Mobile device hardware keystore attestation flow](docs/images/attestation-flow.png)

1. **Local-First MCC Logging:** The mobile app opportunistically logs the device's current Mobile Country Code (MCC) to a local, encrypted SQLite database. This history never leaves the device.
2. **Self-Calculated Eligibility:** When a user attempts to answer a region-specific poll (e.g., US only), the app queries its own internal MCC log to calculate a geographic "Trust Index" (e.g., `Tier_1_Resident` if the device has registered a US MCC for 90% of the last 180 days).
3. **Cryptographic Signing:** The app packages the poll answer and the Trust Index, then asks the device's physical Secure Enclave (iOS) or Hardware Keystore (Android) to cryptographically sign the payload. 
4. **Air-gapped Verification:** The FastAPI backend receives the payload, verifies the hardware signature using the device's registered public key, strips away all device identifiers, and stores only the anonymous vote and eligibility tier.

## 🏗️ Architecture Stack

This repository is split into two primary components:

* **Mobile Client (`/client`):** Built with React Native. It utilizes `@silencelaboratories/react-native-secure-key` to interact directly with the Secure Enclave/Keystore, bypassing the JavaScript layer for sensitive cryptographic operations.
* **Backend API (`/server`):** A high-performance, async Python API built with **FastAPI**. It handles payload ingestion, signature verification (via the `cryptography` library), and routing validated data to a PostgreSQL database.

## 🚀 Getting Started

### Prerequisites
* Python 3.10+
* Node.js & React Native CLI
* PostgreSQL

### Installation
Clone the repository and spin up the development environment:

```bash
git clone [https://github.com/tigers1997/secure-poll.git](https://github.com/tigers1997/secure-poll.git)
cd secure-poll

# Start the FastAPI backend
cd server
pip install -r requirements.txt
uvicorn main:app --reload

# In a new terminal, start the React Native client
cd client
npm install
npx react-native run-ios # or run-android
