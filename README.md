# InterRealm

**The semantic network for AI agents.**

---

We're building the infrastructure layer that lets AI agents discover, invoke, and trust capabilities across boundaries.

```
Agents → Semantic Artifacts → Realms → Execution
```

**Semantic Artifacts** — Typed AI capabilities (entities, prompts, tools) that translate to/from MCP  
**Realms** — Logical namespaces with membership, policy, and federation  
**Routing** — Semantic resolution that abstracts MCP, HTTP, and cross-realm invocation  

---

## Repositories

| Repo | Description | Status |
|------|-------------|--------|
| [**semantic-artifacts**](https://github.com/interrealm-io/semantic-artifacts) | Specification for typed AI capabilities | 📄 v1 Draft |
| **interrealm-spec** | Realm routing & federation protocol | 🔜 Coming |
| **interrealm-sdk** | TypeScript/Python agent SDK | 🔜 Coming |
| **mcp-bridge** | Bidirectional MCP ↔ Artifact translation | 🔜 Coming |

---

## The Problem

AI tooling is fragmented:
- MCP servers are point-to-point
- No semantic typing across tools  
- No governance layer
- No federation across organizations
- Every agent is an island

## The Solution

InterRealm introduces:
- **Entities as first-class types** — Prompts and tools bind to schemas
- **Realms as logical boundaries** — Namespace capabilities with membership and policy
- **Semantic routing** — Agents request capabilities, not endpoints
- **Federation** — Cross-realm trust and invocation

---

## Quick Example

```yaml
# A Semantic Artifact
apiVersion: interrealm.io/semantic-artifacts/v1
kind: Artifact
metadata:
  name: invoice-processor
  version: 1.0.0

entities:
  - name: Invoice
    schema:
      type: object
      properties:
        vendor: { type: string }
        amount: { type: number }

prompts:
  - name: extract-invoice
    output:
      entity: Invoice  # ← Typed output
```

```typescript
// Agent SDK
const realm = await InterRealm.connect("realm:acme:finance");
const tools = await realm.discoverTools();

// LLM sees typed tools, SDK handles routing
const invoice = await realm.invoke("invoice-processor.extract-invoice", {
  document: pdfResource
});
```

---

## Backed By

<a href="https://realmtrix.com">
  <img src="https://realmtrix.com/logo.svg" alt="Realmtrix" height="32">
</a>

**Realmtrix** — Enterprise AI infrastructure  

InterRealm is the open specification. [Realmtrix Edge](https://realmtrix.com) is the enterprise platform with managed realms, LDAP/OIDC integration, audit dashboards, and support.

---

## Get Involved

- 📖 [Read the Semantic Artifacts spec](https://github.com/interrealm-io/semantic-artifacts-spec)
- 🐦 Follow updates (Twitter/X coming)

---

<p align="center">
  <i>Just as the Internet connected networks, InterRealm connects AI capability domains.</i>
</p>
