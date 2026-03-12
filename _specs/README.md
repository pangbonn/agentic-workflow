# _specs — MAS + Playbook + Git

> คู่มือสำหรับ mini project: research เต็มระบบ, คีย์เดียว ~20 USD, แยก branch ตามฟีเจอร์

---

## วิธีสั่งงาน (ประหยัด token)

**สั่งผ่าน:** แชทกับ Claude ใน Cursor (หรือ Cline) — ไม่ต้องใช้ tool แยก

**รูปแบบ:** พิมพ์ **trigger + goal** ในข้อความเดียว เช่น  
`PM: สร้าง API รายการ todo CRUD`  
`ARCH: ออกแบบ endpoint จาก _specs/todo-spec.md`  
`DEV: implement ตาม _specs/todo-spec.md และ todo-arch`  
`QA: ตรวจ todo flow ตาม acceptance criteria ใน spec`

**แบบประหยัด token (แนะนำ):**
- สั่ง **ทีละ role** ต่อหนึ่งข้อความ — ไม่ยิง PM+ARCH+DEV ในคำสั่งเดียว
- **อ้างอิงไฟล์เฉพาะที่ใช้:** `@CLAUDE.md` หรือ `@_specs/todo-spec.md` แทนการเปิดทั้ง repo
- ข้อความสั้น ตรงเป้า — goal ชัดเจนแล้วให้ agent ไปอ่าน spec เอง
- ครั้งถัดไปเปิดแชทใหม่ได้ แล้วสั่ง `DEV: ทำต่อจาก _specs/state.json และ _specs/todo-spec.md` เพื่อลด context ซ้ำ

---

## 1. จับคู่ MAS กับ Playbook (CLAUDE.md)

| MAS Role    | Playbook Trigger | หน้าที่หลัก | Output ไปที่ |
|-------------|------------------|-------------|--------------|
| **RESEARCHER** | `PM: [goal]`   | วิเคราะห์ requirement, เขียน spec, acceptance criteria | `_specs/<feature>-spec.md` |
| **ARCHITECT**  | `ARCH: [goal]` | ออกแบบ API contract, ERD, pattern | `_specs/<feature>-arch.md` หรืออัปเดต spec |
| **CODER**      | `DEV: [goal]`  | เขียนโค้ดตาม spec/arch, tests | branch `feature/<name>` |
| **QA**         | `QA: [goal]`   | รัน test, edge cases, รายงาน | อัปเดต state / ส่งต่อ CODER แก้ |

**ส่งงานระหว่าง agent:** ใช้ JSON ใน `_specs/` — ใส่ path ของ spec ที่สร้าง (เช่น `_specs/todo-spec.md`) ใน `state.json` หรือในไฟล์ spec ชุดเดียวกัน

---

## 2. Git — แยก branch ตามฟีเจอร์

- **main** — ห้าม commit โดยตรง ใช้ merge จาก PR เท่านั้น
- **feature/<ชื่อสั้น>** — หนึ่งฟีเจอร์ต่อหนึ่ง branch  
  ตัวอย่าง: `feature/todo-api`, `feature/auth-check`, `feature/ci-setup`
- **fix/<คำอธิบาย>** — แก้บั๊ก
- **chore/<งาน>** — เช่น chore/docs, chore/deps

**Flow สั้น ๆ:**
```bash
git checkout main && git pull
git checkout -b feature/<ชื่อ>
# ทำงาน (RESEARCHER → ARCHITECT → CODER → QA ตาม spec)
git add . && git commit -m "feat(scope): description"
git push -u origin feature/<ชื่อ>
# เปิด PR → merge หลัง review/CI ผ่าน
```

---

## 3. ควบคุม Budget (~20 USD, คีย์เดียว)

- **Research เต็มระบบแต่กระชับ:**  
  - PM (RESEARCHER): สร้าง spec สั้นชัด หนึ่งไฟล์ต่อฟีเจอร์ (User Story + Acceptance Criteria 3–5 ข้อ)  
  - ARCH: ออกแบบเฉพาะ API/ข้อมูลที่จำเป็น ไม่ over-design  
  - DEV: ทำตาม spec/arch โดยตรง ไม่ให้ agent อ่านทั้ง repo ซ้ำ  
  - QA: เช็คเฉพาะรายการใน spec + smoke test  

- **ลด token:**  
  - อ้างอิง `CLAUDE.md` + `_specs/<feature>-spec.md` แทนการยิง context ยาว  
  - เก็บ output หลักใน `_specs/` และใน branch — ครั้งถัดไปให้ agent อ่านแค่ spec + state  

- **คีย์เดียว:** ใช้ model เดียว (หรือ tier เดียว) ทั้ง RESEARCHER / ARCHITECT / CODER / QA เพื่อให้ประมาณต้นทุนง่าย

---

## 4. state.json — ใช้ทำอะไร

- `phase`: planning | research | design | build | test | done  
- `current_agent`: RESEARCHER | ARCHITECT | CODER | QA  
- `branches.active`: branch ที่กำลังทำงาน (เช่น `feature/todo-api`)  
- อัปเดตหลังจบแต่ละ phase / ส่งมือข้าม role

---

**ทำได้แน่นอน:** research เต็มระบบ + จับคู่ MAS กับ playbook + เอาขึ้น git แยก branch ตามฟีเจอร์ + ควบคุม budget ด้วย spec กระชับและ single key ~20 USD ได้ครับ
