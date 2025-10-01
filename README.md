# Proxmox VE API – Postman Collection

A ready-to-use Postman collection generated from the official **Proxmox VE API Viewer**. It covers the REST endpoints exposed under `/api2/json` and includes examples for common workflows on Proxmox VE 7/8 (nodes, VMs, containers, storage, users, clusters, tasks, etc.).

## Contents

- **Collection**: Importable Postman JSON with organized folders by resource.
- **(Optional) Environment**: Suggested variables to simplify switching between Proxmox hosts and auth methods.
- **Examples**: Requests demonstrate path/query params and bodies where applicable.

---

## Quick Start

1. **Prerequisites**
   - Postman (v10+ recommended)
   - A reachable Proxmox VE host (e.g., `https://pve.example.com:8006`)
   - One of:
     - **API Token** (recommended), or
     - **User + Password** (will generate a ticket/cookie)

2. **Import**
   - In Postman: **File → Import → Upload Files** and select the collection JSON from this repo.
   - (Optional) Import the provided environment JSON, or create a new environment with the variables below.

3. **Select Environment**
   - Choose your Proxmox environment in the top-right environment selector in Postman.

4. **Auth**
   - Use **API Token** *or* **Username/Password** (see details below), then send any request.

---

## Environment Variables

| Variable | Example | Notes |
|---|---|---|
| `PVE_BASE_URL` | `https://pve.example.com:8006` | Protocol + host + port (no trailing slash) |
| `PVE_TOKEN_ID` | `root@pam!mytoken` | Only for token auth (Datacenter → Permissions → API Tokens) |
| `PVE_TOKEN_SECRET` | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` | Only for token auth |
| `PVE_USERNAME` | `root@pam` | Only for ticket (cookie) auth |
| `PVE_PASSWORD` | `your-password` | Only for ticket (cookie) auth |
| `PVE_REALM` | `pam` | Usually `pam`, `pve`, or your SSO realm |
| `PVE_TICKET` | *(auto-set)* | Filled after login; used as `PVEAuthCookie` |
| `PVE_CSRF` | *(auto-set)* | Filled after login; required for write ops when using cookie auth |

> You don’t need `PVE_CSRF` with **API Tokens**. CSRF is only required for POST/PUT/DELETE when authenticating with **ticket/cookie**.

---

## Authentication

### Option A — API Token (recommended)

1. In the Proxmox UI: **Datacenter → Permissions → API Tokens**  
2. Create a token for a user (set privileges as needed; prefer least privilege).  
3. In Postman, set these environment variables:
   - `PVE_TOKEN_ID` = `USER@REALM!TOKENID`
   - `PVE_TOKEN_SECRET` = the generated secret
4. The collection adds this header automatically on every request:  


Authorization: PVEAPIToken={{PVE_TOKEN_ID}}={{PVE_TOKEN_SECRET}}


**Benefits:** stateless, no CSRF token required, easy to rotate.

---

### Option B — Username + Password (ticket/cookie)

1. Set `PVE_USERNAME`, `PVE_PASSWORD`, and `PVE_REALM`.
2. Send the request in **Auth → Login** (under the `access`/`tickets` folder).
3. A **Pre-request Script**/Test captures and stores:
- `PVE_TICKET` → used as cookie `PVEAuthCookie`
- `PVE_CSRF` → used as header `CSRFPreventionToken` for write operations
4. Subsequent requests will include:
- `Cookie: PVEAuthCookie={{PVE_TICKET}}`
- `CSRFPreventionToken: {{PVE_CSRF}}` (only for POST/PUT/DELETE)

---

## Using the Collection

- **Base URL**: All requests use `{{PVE_BASE_URL}}/api2/json/...`
- **Folders** mirror API resources (e.g., `nodes`, `cluster`, `storage`, `access`, `qemu`, `lxc`).
- **Path/Query Params** are templated where helpful (e.g., `{{node}}`, `{{vmid}}`).
- **Bodies** include minimal valid examples for create/update operations.

### Common Workflows

- List nodes: `GET {{PVE_BASE_URL}}/api2/json/nodes`
- List VMs on a node: `GET {{PVE_BASE_URL}}/api2/json/nodes/{{node}}/qemu`
- Start a VM: `POST {{PVE_BASE_URL}}/api2/json/nodes/{{node}}/qemu/{{vmid}}/status/start`
- Create a container: `POST {{PVE_BASE_URL}}/api2/json/nodes/{{node}}/lxc`
- Track a task: `GET {{PVE_BASE_URL}}/api2/json/nodes/{{node}}/tasks/{{upid}}/status`

---

## Certificates & HTTPS

Proxmox commonly uses a self-signed certificate out of the box.

- In Postman, you can enable **“SSL certificate verification”** off for local testing (**Settings → General**), or
- Add your CA/host certs in **Settings → Certificates**.

Prefer securing TLS properly in production and re-enabling verification.

---

## Troubleshooting

- **401 Unauthorized**  
- Token ID/secret mismatch or insufficient permissions  
- Ticket expired (re-run Login), wrong realm, or wrong base URL/port
- **403 Permission check failed**  
- The user or token lacks the required `Privilege` on the target path
- **CSRF token missing** (when using ticket/cookie)  
- Include `CSRFPreventionToken` header for write operations
- **Self-signed cert errors**  
- Temporarily disable verification or install proper certificates

---

## Security Notes

- Prefer **API Tokens** over password tickets for automation.
- Scope tokens with **least privilege** and **expiration/rotation** policies.
- Never commit secrets to the repo. Use Postman environments or secret stores.
- Consider read-only roles for browsing endpoints.

---

## Compatibility

- Tested against Proxmox VE 7.x and 8.x APIs.
- Endpoints come from the official API viewer; availability may vary by version and your node’s feature set.

---

## Contributing

Issues and PRs are welcome:
- Add missing examples/bodies
- Improve scripts/tests
- Expand environment presets

Please avoid including real hostnames, usernames, or secrets in examples.

---

## License

This repository contains a Postman collection derived from publicly documented endpoints of Proxmox VE.  
Choose a license that fits your needs (e.g., MIT). If omitted, defaults to “All rights reserved”.

---

## Acknowledgements

- Proxmox VE and the **PVE API Viewer**
- The Postman team and community

---

### Appendix: Example cURL (API Token)

```bash
curl -s \
-H "Authorization: PVEAPIToken=root@pam!mytoken=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" \
https://pve.example.com:8006/api2/json/nodes
```
Appendix: Example cURL (Ticket/Cookie)

# Login (get ticket and CSRF)
curl -s -k -d "username=root@pam&password=YOURPASS" \
  https://pve.example.com:8006/api2/json/access/ticket | jq

# Use the returned 'ticket' as PVEAuthCookie and 'CSRFPreventionToken' for write ops
curl -s -k \
  -H "CSRFPreventionToken: <token>" \
  --cookie "PVEAuthCookie=<ticket>" \
  -X POST https://pve.example.com:8006/api2/json/nodes/pve/qemu/100/status/start


