# 🕶️ Shadow Room: Clandestine Zero-Knowledge Real-Time Messaging Network

[![Go Version](https://img.shields.io/badge/Go-1.24+-00ADD8?style=flat&logo=go)](https://golang.org)
[![React](https://img.shields.io/badge/React-19.2-61DAFB?style=flat&logo=react)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.5+-3178C6?style=flat&logo=typescript)](https://www.typescriptlang.org)
[![Vite](https://img.shields.io/badge/Vite-8.1-646CFF?style=flat&logo=vite)](https://vitejs.dev)
[![Gorilla WebSocket](https://img.shields.io/badge/WebSocket-v1.5-010101?style=flat&logo=websocket)](https://github.com/gorilla/websocket)
[![Redis](https://img.shields.io/badge/Redis-7+-DC382D?style=flat&logo=redis)](https://redis.io)
[![Dexie IndexedDB](https://img.shields.io/badge/Dexie.js-v4-00BFFF?style=flat)](https://dexie.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A privacy-first, zero-knowledge, end-to-end encrypted (E2EE) real-time messaging network and blind relay grid designed for clandestine, decentralized communication.

Shadow Room eliminates centralized data collection: no phone numbers, email addresses, or personal identifiable information (PII) are ever requested, logged, or persisted. Operatives generate asymmetric cryptographic identities (ECDSA and ECDH on curve P-256) locally within the browser hardware runtime via the native WebCrypto API. Network relay nodes operate completely blind—routing Layer 1 envelopes while remaining mathematically incapable of inspecting or decrypting Layer 2 secret payloads.

The platform provides end-to-end encrypted 1-to-1 communication, encrypted multi-party rooms powered by a Signal-inspired Sender Key ratchet, in-band WebSocket identity elevation via single-use cryptographic tickets, offline store-and-forward dead-drops backed by Redis/Upstash with atomic Lua evacuation, real-time presence tracking, cryptographic delivery/read receipts, and local-first persistence with Dexie.js (IndexedDB).

---

## 📑 Table of Contents

- [Architecture Overview](#-architecture-overview)
- [Repository Structure (Submodules)](#-repository-structure-submodules)
- [Key Features](#-key-features)
  - [Clandestine Identity & Hardware Enrollment](#-clandestine-identity--hardware-enrollment)
  - [Blind Relay Node & In-Band Socket Elevation](#-blind-relay-node--in-band-socket-elevation)
  - [End-to-End Encrypted 1-to-1 Channels](#-end-to-end-encrypted-1-to-1-channels)
  - [Cryptographic Group Rooms & Sender Key Ratchet](#-cryptographic-group-rooms--sender-key-ratchet)
  - [Store-and-Forward Dead-Drop System](#-store-and-forward-dead-drop-system)
  - [Presence, Cryptographic Receipts & Typing Taps](#-presence-cryptographic-receipts--typing-taps)
  - [Local-First Encrypted Vault (Dexie / IndexedDB)](#-local-first-encrypted-vault-dexie--indexeddb)
- [Security & Cryptography Model](#-security--cryptography-model)
  - [1. Double-Key Identity Architecture](#1-double-key-identity-architecture)
  - [2. Zero-Knowledge Blind Relay Protocol](#2-zero-knowledge-blind-relay-protocol)
  - [3. Single-Use Infiltration Tickets](#3-single-use-infiltration-tickets)
  - [4. Multi-Party Sender Key Ratchet](#4-multi-party-sender-key-ratchet)
  - [5. Atomic Dead-Drop Evacuation & Replay Protection](#5-atomic-dead-drop-evacuation--replay-protection)
  - [6. Ephemeral In-Memory Node Registry](#6-ephemeral-in-memory-node-registry)
- [Tech Stack](#-tech-stack)
- [Quick Start & Local Setup](#-quick-start--local-setup)
  - [Prerequisites](#prerequisites)
  - [1. Clone Repository (with Submodules)](#1-clone-repository-with-submodules)
  - [2. Backend Setup](#2-backend-setup)
  - [3. Frontend Setup](#3-frontend-setup)
- [Environment Configuration](#-environment-configuration)
- [API & Protocol Reference](#-api--protocol-reference)
  - [HTTP Infiltration & Directory Endpoints](#http-infiltration--directory-endpoints)
  - [WebSocket Entry & Command Pipeline](#websocket-entry--command-pipeline)
  - [Two-Layer Wire Framing Layout](#two-layer-wire-framing-layout)
- [Working with Git Submodules](#-working-with-git-submodules)
- [License](#-license)

---

## 🏗️ Architecture Overview

The project is structured as a parent orchestration repository that links the frontend and backend services through **Git Submodules**:

```
                                  ┌────────────────────────────────────────┐
                                  │           Client / Browser             │
                                  │       Operative Terminal / Rig         │
                                  │   (WebCrypto ECDSA / ECDH / AES-GCM)   │
                                  └──────────────────┬─────────────────────┘
                                                     │
                                       HTTPS Handshake & Full-Duplex WSS
                                                     │
                                                     ▼
                                  ┌────────────────────────────────────────┐
                                  │         Shadow Room Node (Go)          │
                                  │          shadow-room-backend           │
                                  └──────┬───────────────┬───────────────┬─┘
                                         │               │               │
                            ECDSA P-256  │    Ephemeral  │   Store-and-  │
                           In-Band Auth  │   Auth Cookie │     Forward   │
                                         │               │   Dead-Drops  │
                                         ▼               ▼               ▼
                                  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
                                  │  In-Memory  │ │ Redis /     │ │ Live Rig    │
                                  │  Registry   │ │ Upstash     │ │ Sockets     │
                                  │ (Volatile)  │ │ Vault Cache │ │ (Full-Duplex│
                                  └─────────────┘ └─────────────┘ └─────────────┘
```

---

## 📂 Repository Structure (Submodules)

This parent repository links the two standalone repositories that make up the system:

| Submodule | Description | Repository Link |
| :--- | :--- | :--- |
| **`shadow-room-backend`** | Go network relay node built with Gorilla WebSocket, Redis Vault Broker, in-memory volatile operative registry, ECDSA ticket verification, and atomic store-and-forward dead-drops. | [Backend Repo](https://github.com/gurjotsingh1774/shadow-room-backend) |
| **`shadow-room-frontend`** | React 19, TypeScript, and Vite single-page application featuring WebCrypto E2EE primitives, Signal Sender Key ratchet, Dexie.js IndexedDB storage, Zustand state management, and tactical clandestine UI. | [Frontend Repo](https://github.com/gurjotsingh1774/shadow-room-frontend) |

---

## ✨ Key Features

### 🕶️ Clandestine Identity & Hardware Enrollment
- **Zero PII Registration:** No telephone numbers, email addresses, or usernames required. Operatives enroll solely by presenting client-generated public keys.
- **Hardware-Enforced Cryptography:** Private keys never leave the client's browser hardware execution boundary (`crypto.subtle`).
- **Tactical Pseudonym Matrix:** Every operative receives an autonomously generated military-grade codename (e.g., `Phantom-Viper`, `Ghost-Cipher`) and random avatar identifier.
- **Decentralized Directory Search:** Operatives can search peers by codename or hex ID, returning authenticated public key bundles (PEM + JWK) for key agreement.

### 🛡️ Blind Relay Node & In-Band Socket Elevation
- **Two-Layer Protocol Isolation:** The node routes Layer 1 transmission envelopes (`to`, `from`, `type`, `timestamp`) while remaining 100% blind to Layer 2 encrypted payloads.
- **Guest Gatekeeper:** Unauthenticated WebSocket connections enter as isolated guest lines under a strict 15-second grace countdown timer.
- **Single-Use Ticket Elevation:** Clients fetch a volatile, high-entropy ticket via authenticated HTTP session, sign it with their local ECDSA identity key, and elevate the socket in-band via an `AUTH_REQUEST` command frame.
- **Deterministic Origin Stamping:** The relay node strips any client-provided origin claims and injects the authenticated `from` operative ID directly into the transmission envelope to thwart identity spoofing.

### 🔒 End-to-End Encrypted 1-to-1 Channels
- **Pairwise Diffie-Hellman Key Agreement:** Direct operative communications negotiate shared secrets via ECDH over curve P-256.
- **AES-GCM-256 Authenticated Encryption:** Every message is sealed with 256-bit AES-GCM using unique, non-repeating 96-bit initialization vectors (IVs).
- **Cryptographic Non-Repudiation:** Every outbound payload includes an IEEE P1363 / ASN.1 DER ECDSA digital signature over the ciphertext and IV, mathematically proving message authenticity and preventing tampering.

### 👥 Cryptographic Group Rooms & Sender Key Ratchet
- **Signal-Inspired Sender Keys:** Group rooms maintain zero-knowledge secrecy using individual Sender Key bundles. Each operative generates their own 256-bit symmetric chain key and ratchets it for each message sent.
- **Encrypted Key Distribution:** Sender keys are encrypted individually using pairwise ECDH shared secrets and distributed to room members over the blind relay mesh.
- **Password-Protected Access:** Rooms support cryptographic passphrases verified upon join requests.
- **Dynamic Room Life Cycle:** Create, join, inspect active member rosters, leave, or permanently destroy rooms with automatic key re-negotiation.

### 📬 Store-and-Forward Dead-Drop System
- **Redis Vault Broker:** When an operative is detached from the grid, outbound encrypted signals are routed into an ephemeral Redis dead-drop queue (`deaddrop:{operativeID}`) with a 7-day auto-dissolution TTL.
- **Atomic Lua Evacuation:** Upon reconnection and identity elevation, an atomic Lua script extracts and liquidates all queued dead-drops in a single indivisible operation, eliminating race conditions and duplicate delivery hazards.
- **Cloud-Ready (Upstash & Redis TLS):** Fully compatible with local Redis instances as well as Upstash Redis over enforced TLS (`rediss://`).

### ⚡ Presence, Cryptographic Receipts & Typing Taps
- **Surveillance Taps:** Operatives tap target peer presence (`SUBSCRIBE_STATUS` / `UNSUBSCRIBE_STATUS`), receiving instant status transitions (`online` / `offline`) without leaking network addresses.
- **Cryptographic Delivery Receipts:** Recipients dispatch signed `MESSAGE_DELIVERED` acknowledgments once signals are extracted from the wire.
- **Cryptographic Read Receipts:** Active viewing triggers signed `MESSAGE_READ` acknowledgments, updating message statuses seamlessly across both 1-to-1 chats and multi-party rooms.

### 💾 Local-First Encrypted Vault (Dexie / IndexedDB)
- **Local Persistence:** Chat histories, contact directories, room memberships, identity key pairs, and group sender keys are persisted locally in IndexedDB via Dexie.js.
- **Zero Cloud Storage:** If a server is seized or compromised, zero historical message logs, decrypted texts, or private keys exist on the node.
- **Full Offline Resilience:** Historical messages remain searchable and accessible even when offline.

---

## 🔒 Security & Cryptography Model

```
Operative Rig A                                 Blind Node Relay                            Operative Rig B
┌────────────────────────┐                    ┌──────────────────┐                    ┌────────────────────────┐
│ 1. Local Keygen        │                    │                  │                    │ 1. Local Keygen        │
│    - Identity (ECDSA)  │                    │                  │                    │    - Identity (ECDSA)  │
│    - Handshake (ECDH)  │                    │                  │                    │    - Handshake (ECDH)  │
│                        │                    │                  │                    │                        │
│ 2. Derive Shared Secret│                    │                  │                    │ 2. Derive Shared Secret│
│    ECDH(Priv_A, Pub_B) │                    │                  │                    │    ECDH(Priv_B, Pub_A) │
│                        │                    │                  │                    │                        │
│ 3. Encrypt & Sign      │                    │                  │                    │ 5. Verify & Decrypt    │
│    - AES-GCM-256 (IV)  │  Layer 1 Envelope  │ 4. Blind Route   │  Layer 1 Envelope  │    - ECDSA Verify Sig  │
│    - ECDSA Sign Hash   ├───────────────────►│    - Inspect To  ├───────────────────►│    - AES-GCM-256 Decrypt│
│                        │ (Opaque Layer 2)   │    - Relay / Drop│ (Opaque Layer 2)   │                        │
└────────────────────────┘                    └──────────────────┘                    └────────────────────────┘
```

### 1. Double-Key Identity Architecture
Operatives maintain two distinct elliptic curve key pairs on the **NIST P-256 (secp256r1)** curve:
- **Identity Key (ECDSA):** Dedicated strictly to non-repudiation, challenge-response authentication, ticket signing, and message tampering defense.
- **Handshake Key (ECDH):** Dedicated strictly to Diffie-Hellman key agreement for deriving symmetric encryption keys.

### 2. Zero-Knowledge Blind Relay Protocol
The system enforces strict separation between transmission layers:
- **Layer 1 (Transmission Layer):** Processed by the Go server; contains envelope metadata (`id`, `from`, `to`, `type`, `timestamp`).
- **Layer 2 (Protected Content Layer):** Opaque base64 blob containing ciphertext, IV, ephemeral public keys, and cryptographic signatures. The server cannot read, modify, or forge Layer 2 contents.

### 3. Single-Use Infiltration Tickets
WebSocket connections bypass browser authorization header limitations without exposing credentials:
1. Operative authenticates via HTTP cookies (`POST /api/auth/verify`).
2. Client requests a single-use high-entropy ticket (`POST /api/auth/ws-ticket`).
3. Client opens a raw WebSocket at `/ws` and submits an `AUTH_REQUEST` containing `{ ticket, signature }`.
4. The node liquidates the ticket atomically from Redis (single-use, 60s TTL) and verifies the signature against the operative's public ECDSA key.

### 4. Multi-Party Sender Key Ratchet
In group rooms, pairwise encryption scales poorly ($O(N^2)$). Shadow Room solves this through **Sender Keys**:
- Each sender generates a 32-byte root chain key and transmits it to active room members encrypted via pairwise ECDH.
- For each message, a message key is derived from the chain key and advanced.
- When an operative leaves or is removed, the room's sender keys are discarded and re-established to preserve forward secrecy.

### 5. Atomic Dead-Drop Evacuation & Replay Protection
- Authentication nonces are single-use and expire in 60 seconds. Nonces are destructively consumed from the in-memory registry upon first verification attempt.
- Offline message queues in Redis are extracted via an atomic Lua script (`LANGE` + `DEL`) ensuring that messages cannot be duplicated or intercepted by concurrent connections.

### 6. Ephemeral In-Memory Node Registry
- Operative registrations and active room memberships are stored exclusively in memory (`memstore.ShadowRegistry`).
- The server possesses **no relational database** and writes **zero message records to disk**. Restarting a node wipes all active session traces.

---

## 💻 Tech Stack

### Backend
- **Language:** Go 1.24+ / Go 1.26
- **Multiplexer:** Standard Library `net/http` (Go 1.22+ routing patterns)
- **WebSockets:** Gorilla WebSocket (`github.com/gorilla/websocket`)
- **Key-Value Cache / Vault:** Redis 7+ & Upstash Redis via Go-Redis (`github.com/redis/go-redis/v9`)
- **Authentication:** JWT v5 (`github.com/golang-jwt/jwt/v5`) & HTTP-only SameSite cookies
- **Cryptography:** Standard Go `crypto/ecdsa`, `crypto/elliptic`, `crypto/rand`, `crypto/sha256`, `crypto/x509`

### Frontend
- **Framework:** React 19 (`react`, `react-dom`)
- **Language:** TypeScript 5.5+
- **Build Tool:** Vite 8 with Babel React Compiler plugin
- **Routing:** React Router 8 (`react-router`)
- **State Management:** Zustand 5 (`zustand`)
- **Local Database:** Dexie.js 4 (`dexie` on IndexedDB)
- **HTTP Transport:** Axios (`axios`)
- **Styling:** SCSS & Vanilla CSS with responsive dark-mode clandestine tactical terminal aesthetics
- **Cryptography Primitives:** WebCrypto API (`window.crypto.subtle`)

---

## 🚀 Quick Start & Local Setup

### Prerequisites

Ensure you have the following installed on your machine:
- [Git](https://git-scm.com/) (version 2.13+)
- [Go](https://go.dev/dl/) (version 1.24 or newer)
- [Node.js](https://nodejs.org/) (version 18 or newer) & npm
- [Redis](https://redis.io/download/) (running locally or an [Upstash](https://upstash.com/) account)

---

### 1. Clone Repository (with Submodules)

Clone the repository and automatically initialize and clone all submodules:

```bash
# Using SSH (recommended)
git clone --recurse-submodules git@github.com:gurjotsingh1774/shadow-room.git

# Or using HTTPS
git clone --recurse-submodules https://github.com/gurjotsingh1774/shadow-room.git

cd shadow-room
```

> **Note:** If you already cloned without `--recurse-submodules`, initialize them with:
> ```bash
> git submodule update --init --recursive
> ```

---

### 2. Backend Setup

1. Navigate to the backend directory:
   ```bash
   cd shadow-room-backend
   ```

2. Create an `.env` configuration file:
   ```bash
   cp .env.example .env
   ```

3. Update the `.env` file with your environment parameters:
   ```env
   PORT=8080
   AUTH_SECRET=your-high-entropy-clandestine-auth-secret
   VAULT_ADDRESS=redis://localhost:6379
   GRID_ORIGIN=http://localhost:5173
   HANDSHAKE_TTL_SECONDS=60
   ```

4. Download dependencies and run the server:
   ```bash
   go mod download
   go run cmd/server/main.go
   ```

The backend mesh node will start on `http://localhost:8080`.

---

### 3. Frontend Setup

1. Open a new terminal window and navigate to the frontend directory:
   ```bash
   cd shadow-room-frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure environment variables in `.env`:
   ```env
   VITE_BACKEND_HTTP_URL=http://localhost:8080
   VITE_BACKEND_WS_URL=ws://localhost:8080
   ```

4. Launch the Vite development server:
   ```bash
   npm run dev
   ```

Open `http://localhost:5173` in your browser to enter the Shadow Room.

---

## ⚙️ Environment Configuration

### Backend (`shadow-room-backend/.env`)

| Variable | Required | Default | Description |
| :--- | :---: | :--- | :--- |
| `PORT` | No | `8080` | Port on which the HTTP server and WebSocket mesh gateway listen |
| `AUTH_SECRET` | Yes | fallback secret | Secret key used to sign and verify JWT authentication cookies |
| `VAULT_ADDRESS` | Yes | `redis://localhost:6379` | Redis connection URL (`redis://` for local, `rediss://` for Upstash with enforced TLS) |
| `GRID_ORIGIN` | No | `http://localhost:5173` | Authorized origin for CORS policy headers |
| `HANDSHAKE_TTL_SECONDS` | No | `60` | Lifespan of volatile single-use WebSocket authorization tickets |

### Frontend (`shadow-room-frontend/.env`)

| Variable | Required | Default | Description |
| :--- | :---: | :--- | :--- |
| `VITE_BACKEND_HTTP_URL` | No | `http://localhost:8080` | Base HTTP endpoint for operative registration, challenge, and directory sweep |
| `VITE_BACKEND_WS_URL` | No | `ws://localhost:8080` | Base WebSocket endpoint for real-time encrypted message transmission |

---

## 📡 API & Protocol Reference

### HTTP Infiltration & Directory Endpoints

| Method | Endpoint | Auth | Description |
| :--- | :--- | :---: | :--- |
| `POST` | `/api/auth/register` | Public | Enrolls a new operative profile by presenting public identity and handshake keys |
| `POST` | `/api/auth/challenge` | Public | Requests a high-entropy 32-byte authentication nonce (60s TTL) |
| `POST` | `/api/auth/verify` | Public | Verifies the signed nonce and injects an authenticated HTTP session cookie |
| `POST` | `/api/auth/ws-ticket` | Cookie | Mints a single-use ticket for in-band WebSocket elevation |
| `POST` | `/api/search` | Cookie | Sweeps the active registry for operatives and public rooms by codename or ID |

---

### WebSocket Entry & Command Pipeline

The persistent full-duplex WebSocket gateway is mounted at `/ws`.

```
Client                                                  Node Gateway
  │                                                          │
  ├────── Connects as Unauthenticated Guest ────────────────►│ (15s grace timer starts)
  │                                                          │
  ├────── Command: AUTH_REQUEST { ticket, signature } ──────►│
  │                                                          ├─ Liquidates ticket from Redis
  │                                                          ├─ Verifies ECDSA signature
  │                                                          ├─ Elevates connection
  │                                                          ├─ Flushes offline dead-drops
  │◄───── Command: AUTH_OK ──────────────────────────────────┤
  │◄───── Command: FLUSH_COMPLETE ───────────────────────────┤
  │                                                          │
  ├────── Authenticated Bi-Directional Signal Mesh ─────────►│
```

#### Command Codes Reference (`command` frame type)

| Command Code | Sender | Payload Parameters | Description |
| :--- | :---: | :--- | :--- |
| `AUTH_REQUEST` | Client | `{ ticket, signature }` | Elevates unauthenticated guest terminal to authenticated operative session |
| `AUTH_OK` | Server | Status message | Confirms successful elevation and identity synchronization |
| `SUBSCRIBE_STATUS` | Client | `targetId` | Taps presence transitions for the specified operative |
| `UNSUBSCRIBE_STATUS`| Client | `targetId` | Dismantles presence surveillance for the specified operative |
| `STATUS_UPDATE` | Server | `{ message: "online" \| "offline", targetId }` | Relays presence status change to active subscribers |
| `CREATE_ROOM` | Client | `{ targetId, passphrase, message }` | Creates a new encrypted room with an optional passphrase |
| `CREATE_ROOM_OK` | Server | `targetId` | Confirms room creation |
| `JOIN_ROOM` | Client | `{ targetId, passphrase }` | Joins an existing room; verifies passphrase and notifies members |
| `JOIN_ROOM_OK` | Server | `targetId` | Confirms room admittance |
| `LEAVE_ROOM` | Client | `targetId` | Detaches the operative from a room |
| `DELETE_ROOM` | Client | `targetId` | Room creator permanently dissolves the room |
| `GET_ROOM_MEMBERS` | Client | `targetId` | Requests current roster of member profiles in the room |
| `ROOM_MEMBERS_LIST`| Server | JSON list of operative profiles | Transmits current active room member roster |
| `MESSAGE_DELIVERED`| Client | `{ targetId, message, signature }` | Acknowledges receipt of encrypted packet |
| `MESSAGE_READ` | Client | `{ targetId, message, signature }` | Acknowledges active viewing/read of encrypted packet |
| `FLUSH_COMPLETE` | Server | Status message | Notifies client that offline dead-drop extraction is finished |

---

### Two-Layer Wire Framing Layout

#### Layer 1: Transmission Routing Envelope
```json
{
  "id": "c1f7b029-8472-4d0a-9bc1-5fe1839a9c40",
  "from": "01a4e8d2f5c9...",
  "to": ["94b7e1c8a2d0..."],
  "type": "secure_signal",
  "timestamp": 1729278000000,
  "payload": { ... }
}
```

#### Layer 2: Protected Content Payload (Opaque to Relay Node)
```json
{
  "from": {
    "operativeId": "01a4e8d2f5c9...",
    "terminalId": "rig-terminal-99"
  },
  "contentType": "text/plain",
  "encryption": {
    "alg": "AES-GCM-256",
    "iv": "vG4kL9w+2Pz0xQ==",
    "keyId": "sender-key-uuid-or-shared-ecdh"
  },
  "ciphertext": "k3N9xJ4Fq7P8zW1A5bT2...",
  "sig": {
    "alg": "ECDSA-P256-SHA256",
    "value": "MEUCIQD..."
  },
  "meta": {
    "conversationId": "94b7e1c8a2d0..."
  }
}
```

---

## 🔄 Working with Git Submodules

### Pulling the Latest Changes
To fetch latest changes for both the parent repo and all submodules:
```bash
git pull --recurse-submodules
git submodule update --remote --merge
```

### Making Changes in a Submodule
1. Enter the submodule directory:
   ```bash
   cd shadow-room-backend
   ```
2. Make your edits, commit, and push to origin:
   ```bash
   git add .
   git commit -m "feat: enhance handshake verification"
   git push origin main
   ```
3. Return to the root directory to commit the updated submodule reference:
   ```bash
   cd ..
   git add shadow-room-backend
   git commit -m "chore: bump backend submodule reference"
   git push origin main
   ```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
