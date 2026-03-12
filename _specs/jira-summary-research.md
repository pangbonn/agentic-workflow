# Research: ระบบสรุปงาน Task จาก Jira ให้ทีม

**Role:** PM (RESEARCHER)  
**สถานะ:** มีระบบ tracking บน Supabase แล้ว + ย้าย task ใน Jira แล้วข้อมูลอัปเดตที่ Supabase

---

## ปัญหา

- Task ใน Jira เยอะมาก ทีมดูไม่ทั่ว
- ต้องการวิธีสรุป/เห็นภาพรวมงานให้ทีม

---

## ทางเลือกที่ทำได้

### 1. ใช้ของ built-in ใน Jira (ไม่ต้องพัฒนาต่อ)

- **Summary view (Jira):** แสดงสถิติ completed/updated/created/due, แยกตาม status/priority/assignee, workload, recent activity  
  - บางส่วนต้อง Jira Cloud Premium/Enterprise (overview cards: unassigned, high-priority, overdue, blocked)
- **Reports & Dashboards:** สร้าง dashboard ใน Jira ใส่ gadgets ตาม filter ที่ต้องการ

**ข้อดี:** ไม่ต้องเขียนโค้ด  
**ข้อเสีย:** ข้อมูลอยู่แค่ใน Jira ไม่ได้อยู่ที่ Supabase ที่ทีมใช้อยู่แล้ว

---

### 2. สรุปจากข้อมูลใน Supabase (แนะนำ เพราะมี tracking อยู่แล้ว)

ข้อมูล task มาอยู่ที่ Supabase อยู่แล้ว และมีการอัปเดตเมื่อย้าย task ดังนั้นสามารถ:

- **สร้าง View / Query สรุปใน DB**
  - Group by `status`, `assignee`, `priority` (หรือ field ที่ sync มาจาก Jira)
  - ใช้ PostgreSQL: `GROUP BY`, `COUNT`, `json_agg` หรือ view สำหรับ summary
- **สร้าง UI อ่านจาก Supabase**
  - หน้า dashboard / report อ่านจาก view หรือ query สรุป (โดยไม่ต้องเรียก Jira ซ้ำ)
  - แสดงเช่น: จำนวนแยกตาม status, แยกตามคน, รายการ overdue / high-priority

**ข้อดี:** ใช้ข้อมูลเดียวกับที่ทีมใช้อยู่แล้ว, ควบคุม format การสรุปได้, ลดการพึ่งพา Jira UI  
**ข้อเสีย:** ต้องออกแบบ view + UI (หรือ report) เล็กน้อย

---

### 3. ดึงจาก Jira API โดยตรง (เสริมถ้าต้องการ)

- Jira Cloud: `GET /rest/api/3/search/jql` ใช้ JQL filter (project, assignee, status, created, etc.)
- Pagination ใช้ `nextPageToken`
- ใช้เมื่อต้องการข้อมูล real-time จาก Jira โดยไม่รอ sync มาที่ Supabase

**ใช้เมื่อ:** อยากได้ตัวเลขจาก Jira ทันที หรือ sync ยังไม่ครอบคลุมทุก event

---

### 4. Sync ให้ครบ (Jira → Supabase)

- **Jira Webhook:** ตั้งค่าให้ส่งเมื่อ "Issue updated" / "Issue moved" ไปที่ endpoint ของเรา แล้วให้ backend อัปเดต/insert ลง Supabase
- ถ้าทำแล้ว (ย้าย task แล้วอัปเดต Supabase) แค่ให้แน่ใจว่า event ที่ต้องการ (สร้าง/แก้/ย้าย/ลบ) ครบ

---

## แนวทางที่แนะนำ

- **หลัก:** ทำ **ระบบสรุปจากข้อมูลใน Supabase** (ทางเลือก 2)  
  - เพราะมี tracking และการอัปเดตเมื่อย้าย task อยู่แล้ว
- **ขั้นตอนย่อย:**
  1. กำหนดว่าต้องการสรุปอะไร (เช่น แยกตาม status, assignee, โปรเจกต์, overdue, high-priority)
  2. ออกแบบ/สร้าง **DB view หรือ query สรุป** ใน Supabase (GROUP BY + aggregate)
  3. สร้าง **dashboard/report** (หรือ API อ่าน view) ให้ทีมเปิดดูภาพรวมได้
  4. (ถ้าต้องการ) ใช้ **Jira webhook** ให้ครบทุก event ที่ส่งผลต่อการสรุป

---

## สิ่งที่ต้องชัดก่อนออกแบบ (สำหรับ ARCH/DEV)

- ชุด field ที่ sync จาก Jira ลง Supabase (ชื่อตาราง/คอลัมน์ เช่น status, assignee, priority, due_date, project)
- รายการ "สรุป" ที่ทีมต้องการ (เช่น สรุปต่อคน, ต่อ status, ต่อโปรเจกต์, รายการ overdue)
- ว่า UI จะเป็นแค่ internal report หรือต้องแชร์ลิงก์ให้คนนอกทีม

---

## Out of scope (ในขั้น research นี้)

- เปลี่ยนวิธี sync Jira → Supabase (สมมติว่ามีอยู่แล้วและใช้ได้)
- รายละเอียด Jira Summary view / Premium features
- การ implement จริง (ส่งต่อให้ ARCH แล้ว DEV ตาม spec)

---

## อ้างอิง

- Jira Summary view: [Atlassian docs](https://support.atlassian.com/jira-software-cloud/docs/what-is-the-summary-view)
- Jira REST API search: `/rest/api/3/search/jql` (JQL, pagination ด้วย nextPageToken)
- Jira webhooks: Issue updated / custom transition → HTTP POST ไป endpoint เรา
- Supabase: aggregate + GROUP BY ใน PostgreSQL หรือ view แล้ว query ผ่าน client
