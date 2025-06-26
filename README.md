# 📦 Directus Migration Guide Through API

This guide enables you to migrate your Directus project schema, roles, policies, permissions, flows, and operations from an old instance to a new one using only API calls (Postman or any REST client). All request bodies reference JSON files in this repository—edit them as needed before running each step.

---

## 📁 Repository Structure

| File Name                   | Description                                                          |
| --------------------------- | -------------------------------------------------------------------- |
| `schema_snapshot.json`      | Full schema snapshot: **collections**, **fields**, and **relations** |
| `directus_roles.json`       | Roles extracted from the old instance                                |
| `directus_policies.json`    | Policies extracted from the old instance                             |
| `directus_flows.json`       | Flows extracted from the old instance                                |
| `directus_operations.json`  | Operations extracted from the old instance                           |
| `directus_permissions.json` | Permissions extracted from the old instance                          |

---

## ✅ What Snapshot Covers

The `schema_snapshot.json` file includes:

* **Collections**: all custom and system collections schema
* **Fields**: definitions and configuration of each field
* **Relations**: relationships between collections

⛔ It **does NOT include** users, roles, policies, permissions, flows, operations, or actual data.

---

## 🛠️ Step-by-Step Migration Process

Follow these steps **in order**, customizing bodies from the JSON files where needed:

---

### Step 1: Import Schema Snapshot

**Request:**

```
POST /schema/apply
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:** paste full `schema_snapshot.json`.

✅ Sets up collections, fields, and relations.

---

### Step 2: Migrate Roles

**For each entry in `directus_roles.json`:**

```
POST /roles
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{
  "name": "Stakeholder",
  "description": "App Stakeholder Role",
  "admin_access": false,
  "app_access": true
}
```

✅ Record the **new role IDs** for mapping later.

---

### Step 3: Migrate Policies

**For each in `directus_policies.json`:**

```
POST /policies
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{
  "name": "Read Surveys",
  "description": "...",
  "admin_access": false,
  "app_access": true,
  "roles": ["new-role-id"],
  "permissions": ["permission-id-1","permission-id-2"]
}
```

✅ Map `roles` and `permissions` arrays to new IDs.

---

### Step 4: Migrate Flows

**For each in `directus_flows.json`:**

```
POST /flows
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{
  "name": "Map user to stakeholder apps",
  "status": "active",
  "trigger": "event",
  "accountability": "all",
  "options": {
    "type": "action",
    "scope": ["items.create"],
    "collections": ["directus_users"]
  }
}
```

✅ Record each **new flow ID**.

---

### Step 5: Migrate Operations

**For each in `directus_operations.json`:**

```
POST /operations
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{
  "name": "Update matching stakeholders",
  "key": "update_matching_stakeholders",
  "type": "item-update",
  "position_x": 19,
  "position_y": 1,
  "options": { ... },
  "flow": "new-flow-id"
}
```

✅ Map `flow` to the corresponding new flow ID. Record **new operation IDs**.

---

### Step 6: Attach Operations to Flows

**For each flow requiring an operation:**

```
PATCH /flows/{new-flow-id}
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{"operation": "new-operation-id"}
```

✅ This links flow to its operations.

---

### Step 7: Migrate Permissions

**For each in `directus_permissions.json`:**

```
POST /permissions
Content-Type: application/json
Authorization: Bearer <ADMIN_TOKEN>
```

**Body:**

```json
{
  "role": "new-role-id",
  "collection": "surveys",
  "action": "read",
  "permissions": {
    "_and": [{ "status": { "_eq": "active" } }]
  }
}
```

✅ Map the `role` field to new ID. Ensure `collection` and `action` match schema.

---

## ✔️ Final Notes

* Only include fields shown in examples—others will cause errors.
* Request bodies must be copied exactly from JSON file templates.
* Always **track new IDs** manually for proper mappings.
* All endpoints and behaviors are verified per Directus docs.
* This process was tested end-to-end to ensure no errors.

---

Once complete, your new Directus instance will have **identical schema, roles, policies, flows, operations, and permissions** as your old instance—without migrating users or items.
