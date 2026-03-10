# realmnet

> The peer-to-peer realm address ledger for the InterRealm protocol.

`realmnet` is an open-source distributed ledger written in Go that enables **realm registration, discovery, and verification** without any central authority. It is the reference implementation of the [InterRealm protocol](https://interrealm.io).

---

## What is a Realm?

A **realm** is a sovereign, identity-bounded container for AI capabilities, tools, and governance policies. Realms are the core primitive of the InterRealm protocol — each one has a unique address, a cryptographic identity, and a defined set of capabilities it exposes to federated peers.

```
apex.realmnet        →   Newrock's AI realm
core.realmnet        →   Realmtrix platform realm
contractor.realmnet  →   A scoped sub-realm with delegated access
```

---

## What is realmnet?

`realmnet` is a **distributed ledger** where realm addresses live. Think of it as decentralized DNS + PKI for AI infrastructure:

- Any realm can **register** its address, endpoint, and public key
- Registrations are **cryptographically signed** and **immutable**
- The ledger is **replicated across all participating nodes** — no central registry
- Anyone can **resolve** a realm address to its current endpoint and public key

---

## Realm Network Participation

Joining realmnet is **opt-in**. A realm can operate in two modes:

### Private Mode (default)
The realm runs locally — internal use only. No endpoint is exposed, no data is written to the public ledger. Suitable for development, air-gapped environments, or internal enterprise deployments.

```bash
# Realm runs with no realmnet participation
realmctl node start --mode private
```

### Public Mode (realmnet participant)
The realm explicitly joins the network. Its realm ID, endpoint, and public key are written to the ledger and visible to all peers. **This is a deliberate, informed choice** — the operator knows their endpoint is public.

```bash
# Realm opts in — endpoint becomes publicly discoverable
realmctl register \
  --realm newrock.realmnet \
  --endpoint https://realm.newrock.com \
  --keyfile ./keys/newrock.pem \
  --join-network
```

### What Gets Exposed When You Join

| Field | Public | Notes |
|-------|--------|-------|
| Realm ID | ✅ | e.g. `newrock.realmnet` |
| Endpoint URL | ✅ | Your realm's network address |
| Public Key | ✅ | Used for signature verification |
| Realm Name | ✅ | Human-readable label |
| Internal IPs | ❌ | Never written to ledger |
| Capability details | ❌ | Governed locally, not on ledger |
| User data | ❌ | Never leaves your realm |

> **The ledger is a directory, not a window into your realm.** It tells the network you exist and how to reach you — nothing more.

---

## Architecture

```
┌─────────────────┐     P2P Gossip      ┌─────────────────┐
│ newrock.realmnet│◄───────────────────►│  core.realmnet  │
│                 │                     │                 │
│  full node      │                     │  full node      │
│  local chain    │                     │  local chain    │
└─────────────────┘                     └─────────────────┘
         ▲                                       ▲
         │                                       │
         ▼                                       ▼
┌─────────────────┐                     ┌─────────────────┐
│ partner.realmnet│                     │  audit.realmnet │
│                 │                     │                 │
│  light node     │                     │  observer node  │
└─────────────────┘                     └─────────────────┘
```

### Node Types

| Type | Description |
|------|-------------|
| **Full Node** | Holds complete chain, validates all blocks, participates in consensus |
| **Light Node** | Validates only its own transactions, trusts full node peers |
| **Observer** | Read-only, suitable for compliance and audit use cases |

---

## Block Structure

Each block on the ledger represents a **realm lifecycle event**:

```json
{
  "index": 1,
  "type": "REALM_REGISTERED",
  "timestamp": 1741478400,
  "payload": {
    "realmId": "newrock.realmnet",
    "endpoint": "https://realm.newrock.com",
    "publicKey": "-----BEGIN PUBLIC KEY-----...",
    "metadata": {
      "name": "Newrock",
      "tier": "enterprise"
    }
  },
  "signature": "<signed by registering realm's private key>",
  "previousHash": "a3f9c2d8...",
  "hash": "7d82e1b4..."
}
```

### Event Types

| Event | Description |
|-------|-------------|
| `REALM_REGISTERED` | A new realm joins the network |
| `REALM_UPDATED` | Endpoint or metadata changed |
| `REALM_DEACTIVATED` | Realm signals it is no longer active |
| `TRUST_ESTABLISHED` | Two realms declare a federation agreement |
| `TRUST_REVOKED` | Federation agreement terminated |

---

## Realm Address Ownership

### First Registration Wins

Realm addresses are claimed on a **first-write basis**. Once a realm ID is written to the ledger, every peer on the network rejects any subsequent attempt to register the same ID. There is no reservation system — the ledger is the authority.

### Only the Keyholder Can Update

Registration binds a realm ID to a **public key**. All future events for that realm — updates, deactivation, trust agreements — must be **signed by the matching private key**. Peers verify this signature against the public key already on the ledger before accepting any block.

```
newrock.realmnet  →  registered with pubkey A
                           ↓
Someone attempts REALM_UPDATED signed with key B
                           ↓
Peers verify: signature does not match pubkey A
                           ↓
Block rejected network-wide
```

This means:
- **Squatting is possible** — anyone can register `newrock.realmnet` first
- **Squatting is useless** — without the private key, a squatter cannot impersonate the real realm, modify its record, or receive federated traffic meant for it
- **Key rotation** is supported via `REALM_UPDATED` — but only provable by the current keyholder

### Choosing a Realm ID

Pick something that clearly identifies your organization:

```
{org}.realmnet              →  newrock.realmnet
{service}.{org}.realmnet    →  ai.newrock.realmnet
{env}.{org}.realmnet        →  dev.newrock.realmnet
```

> Naming conventions are by agreement today. A formal namespace spec is planned for InterRealm v0.2.

---

## Getting Started

### Prerequisites

- Go 1.22+
- A keypair for your realm (or generate one with `realmctl`)

### Install

```bash
go install github.com/interrealm-io/realmnet/cmd/realmctl@latest
```

### Generate a Realm Keypair

```bash
realmctl keygen --realm newrock.realmnet --out ./keys/
```

### Register Your Realm

```bash
realmctl register \
  --realm newrock.realmnet \
  --endpoint https://realm.newrock.com \
  --keyfile ./keys/newrock.pem
```

### Resolve a Realm Address

```bash
realmctl resolve newrock.realmnet
```

### Start a Node

```bash
realmctl node start --port 7946
```

### Inspect the Chain

```bash
realmctl chain status
realmctl chain list
realmctl chain verify
```

---

## Repository Structure

```
realmnet/
├── cmd/
│   └── realmctl/        # CLI entrypoint
├── internal/
│   ├── block/           # Block types, hashing, validation
│   ├── chain/           # Chain management, fork resolution
│   ├── ledger/          # Realm registration and event logic
│   ├── p2p/             # Peer discovery and gossip protocol
│   └── crypto/          # Key generation, signing, verification
├── pkg/
│   └── realmnet/        # Public SDK — importable by other Go apps
├── genesis/
│   └── genesis.json     # Hardcoded genesis block
└── spec/                # Protocol specification documents
```

---

## Using the Go SDK

Any Go application can import `realmnet` to resolve realm addresses:

```go
import "github.com/interrealm-io/realmnet/pkg/realmnet"

client, err := realmnet.NewClient(realmnet.DefaultBootstrapPeers)
if err != nil {
    log.Fatal(err)
}

realm, err := client.Resolve("newrock.realmnet")
if err != nil {
    log.Fatal(err)
}

fmt.Println(realm.Endpoint)   // https://realm.newrock.com
fmt.Println(realm.PublicKey)  // -----BEGIN PUBLIC KEY-----...
```

---

## Relationship to InterRealm & Realmtrix

```
interrealm.io (protocol spec)
      │
      ▼
realmnet (this repo — open reference implementation)
      │
      ▼
realmtrix.com (enterprise platform built on the protocol)
```

- **InterRealm** defines the open protocol and governance model
- **realmnet** is the reference Go implementation — open source, permissively licensed
- **Realmtrix** is the commercial enterprise control plane that extends realmnet with capability packages, RBAC, Keycloak integration, and a managed SaaS offering

---

## Roadmap

- [ ] Genesis block + core block/chain types
- [ ] Realm registration and signature verification
- [ ] BoltDB-backed local chain storage
- [ ] libp2p-based P2P gossip layer
- [ ] `realmctl` CLI (register, resolve, node, chain)
- [ ] Go SDK (`pkg/realmnet`)
- [ ] Light node support
- [ ] InterRealm spec v0.1 publication
- [ ] JS/TS client SDK (`interrealm-js`)

---

## Contributing

`realmnet` is an open protocol. Contributions, RFCs, and feedback are welcome.

```bash
git clone https://github.com/interrealm-io/realmnet
cd realmnet
go mod tidy
go test ./...
```

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before opening a pull request.

---

## License

Apache 2.0 — see [LICENSE](./LICENSE)

---

> Built by [Realmtrix](https://realmtrix.com) · Protocol spec at [interrealm.io](https://interrealm.io)
