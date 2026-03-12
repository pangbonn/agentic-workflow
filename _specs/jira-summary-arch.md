# Architecture: ระบบสรุปงาน Task จาก Jira (พัฒนาต่อจากระบบเดิม)

**Role:** ARCHITECT  
**อ้างอิง:** `_specs/jira-summary-research.md`  
**หลักการ:** พัฒนาต่อจากระบบ tracking เดิมบน Supabase ไม่ทำซ้ำ sync Jira → Supabase

---

## 1. สมมติฐานจากระบบเดิม

- มี **ตารางเก็บ task ที่ sync จาก Jira** อยู่แล้วใน Supabase (ชื่อตาราง/คอลัมน์ให้ทีมแมปตามของจริง)
- เมื่อย้าย task ใน Jira → มีกระบวนการอัปเดตข้อมูลใน Supabase แล้ว (webhook หรือ polling)
- **ไม่แก้** กลไก sync เดิม แค่เพิ่มชั้นสรุปและแสดงผล

---

## 2. สิ่งที่เพิ่มเข้าไป (บนของเดิม)

| ชั้น | สิ่งที่ทำ | หมายเหตุ |
|-----|-----------|----------|
| **DB** | View(s) สรุปจากตาราง task เดิม | GROUP BY + COUNT, ไม่เก็บข้อมูลซ้ำ |
| **API / Data** | Endpoint หรือ Supabase query อ่าน view | ถ้ามี backend แยก → REST; ถ้าใช้แค่ Supabase client → อ่าน view โดยตรง |
| **UI** | หน้า dashboard/report แสดงภาพรวม | แสดงต่อ status, ต่อ assignee, overdue, high-priority |

---

## 3. Schema สมมติ (ให้แมปกับของจริง)

สมมติตาราง task ใน Supabase มีอย่างน้อย:

- `id` (PK)
- `jira_key` หรือ `issue_key` (เช่น PROJ-123)
- `status` (To Do, In Progress, Done, …)
- `assignee_id` หรือ `assignee_email` / `assignee_name`
- `priority` (High, Medium, Low หรือตัวเลข)
- `project_key` หรือ `project_name`
- `due_date` (nullable)
- `updated_at` (ใช้ตรวจว่า sync ล่าสุดเมื่อไหร่)

ถ้าชื่อคอลัมน์ต่างไป ให้แทนใน view ตามของจริง

---

## 4. View สรุป (PostgreSQL ใน Supabase)

### 4.1 สรุปแยกตาม status

```sql
-- ชื่อ view: jira_summary_by_status (หรือตาม naming ของโปรเจกต์)
CREATE OR REPLACE VIEW jira_summary_by_status AS
SELECT
  status,
  COUNT(*) AS count
FROM jira_issues   -- แก้ชื่อตารางเป็นของจริง
GROUP BY status
ORDER BY count DESC;
```

### 4.2 สรุปแยกตาม assignee

```sql
CREATE OR REPLACE VIEW jira_summary_by_assignee AS
SELECT
  COALESCE(assignee_name, assignee_id::text, 'Unassigned') AS assignee,
  status,
  COUNT(*) AS count
FROM jira_issues
GROUP BY assignee_id, assignee_name, status
ORDER BY assignee, status;
```

### 4.3 รายการที่ overdue และ high-priority (สำหรับโฟกัสทีม)

```sql
CREATE OR REPLACE VIEW jira_focus_overdue_high AS
SELECT
  id,
  jira_key,
  summary,
  status,
  assignee_name,
  priority,
  due_date,
  updated_at
FROM jira_issues
WHERE (due_date IS NOT NULL AND due_date < CURRENT_DATE)
   OR (priority IN ('High', 'Highest', 'Critical') OR priority_level >= 4)  -- แมปตามของจริง
ORDER BY due_date ASC NULLS LAST, priority DESC;
```

ทีมสามารถเพิ่ม view อื่นได้ (เช่นแยกตาม project, sprint) โดยใช้ pattern เดียวกัน

---

## 5. API Contract (ถ้ามี Backend แยก)

ถ้าใช้ NestJS/Express หน้าบน Supabase:

| Method | Path | Response | หมายเหตุ |
|--------|------|----------|----------|
| GET | `/api/summary/by-status` | `{ items: { status, count }[] }` | อ่านจาก view สรุปตาม status |
| GET | `/api/summary/by-assignee` | `{ items: { assignee, status, count }[] }` | อ่านจาก view สรุปตามคน |
| GET | `/api/summary/focus` | `{ items: Issue[] }` | รายการ overdue + high-priority (จาก view โฟกัส) |

Response DTO ตัวอย่าง:

```ts
// by-status
interface SummaryByStatusItem { status: string; count: number; }

// by-assignee
interface SummaryByAssigneeItem { assignee: string; status: string; count: number; }

// focus (ใช้ type ของ task เดิม)
interface FocusIssue { id: string; jira_key: string; summary: string; status: string; assignee_name: string; priority: string; due_date: string | null; updated_at: string; }
```

ถ้า **ไม่มี backend แยก** — ใช้ Supabase client อ่าน view โดยตรงจาก frontend (เปิด RLS ให้ view อ่านได้ตามนโยบายเดิมของตาราง task)

---

## 6. UI Scope (พัฒนาต่อจากระบบเดิม)

- **หน้าเดียวหรือแท็บเดียว:** “สรุปงาน” / “Team summary”
  - บล็อก 1: สรุปแยกตาม status (จำนวนต่อ status)
  - บล็อก 2: สรุปแยกตาม assignee (ตารางหรือการ์ด assignee × status)
  - บล็อก 3: รายการโฟกัส (overdue + high-priority) เป็นตารางคลิกไปดู key ได้
- **ไม่ทำซ้ำ:** ไม่ sync Jira ใหม่ ไม่เก็บ task ซ้ำ แค่ query จากตารางเดิม + view

---

## 7. สิ่งที่ไม่เปลี่ยน (ระบบเดิม)

- กลไก sync Jira → Supabase (webhook / job ที่มีอยู่)
- โครงสร้างตาราง task เดิม (ถ้าไม่จำเป็นไม่ต้องเพิ่มคอลัมน์)
- Auth / RLS ของ Supabase ใช้ต่อได้ — view ใช้สิทธิ์อ่านตามตารางต้นทาง

---

## 8. Checklist ก่อนให้ DEV implement

- [ ] แมปชื่อตาราง/คอลัมน์จริงกับที่ใช้ใน view (jira_issues → ชื่อจริง, assignee_name, priority_level ฯลฯ)
- [ ] ตกลงรายการสรุปที่ต้องการ (ถ้าไม่ใช้ 3 view ด้านบน ให้เพิ่ม/ลด view ตามนี้)
- [ ] ตกลงว่าสรุปจะอ่านผ่าน Backend API หรือ Supabase client โดยตรง
- [ ] ถ้ามี backend: กำหนด endpoint และ DTO ตาม section 5

---

**ส่งมอบ:** เอกสารนี้ + research (`jira-summary-research.md`) พอให้ DEV implement ต่อได้ (view + API หรืออ่าน view + UI)
