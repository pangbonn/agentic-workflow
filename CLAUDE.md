# CLAUDE.md — Agentic SDLC Team Playbook

> **สำหรับ Tech Lead และ AI Agents ทุกตัวในทีม**
> ไฟล์นี้คือ "กฎของสนาม" ที่ทุก Agent ต้องอ่านก่อนเริ่มงาน

---

## When in Doubt

ถ้าไม่แน่ใจ ให้ทำตามลำดับนี้ก่อนทุกครั้ง:

1. อ่าน CLAUDE.md อีกรอบ — คำตอบมักอยู่ที่นี่แล้ว
2. ดู existing code เป็น reference — ทำแบบเดียวกับที่ codebase ทำอยู่
3. **ถามก่อน — อย่า assume** — ถามให้ชัดดีกว่าทำผิดทิศ
4. ถ้าติดปัญหาเดิม 2 รอบแล้วยังไม่คืบ → **escalate ให้ Tech Lead ทันที**

> "A wrong assumption made silently costs more than a clarifying question."

---

## 1. Project Overview

**Repository:** `agentic-workflow`
**วัตถุประสงค์:** ฝึกและกำหนดมาตรฐาน Agentic SDLC Workflow สำหรับทีม Full Stack ที่ใช้ AI เป็น pair programmer แบบ autonomous

**แนวคิดหลัก:**
- ใช้ **ReAct Pattern** (Reason → Act → Observe → Repeat) สำหรับทุก task
- AI ทำงาน **Outcome-based** ไม่รอคำสั่งทีละขั้น
- Agent แต่ละตัวมี role ชัดเจน ไม่ก้าวก่ายกัน
- Tech Lead เป็น Orchestrator — สั่ง intent, ไม่สั่ง implementation

**เป้าหมาย SDLC Team:**
```
Tech Lead (Human)
    └── Orchestrator Agent     ← รับ intent, แจก task
         ├── PM Agent          ← วิเคราะห์ requirement, เขียน spec
         ├── Architect Agent   ← ออกแบบ system, เลือก pattern
         ├── Developer Agent   ← เขียนโค้ด, แก้ bug
         ├── QA Agent          ← เขียน test, ตรวจ quality
         └── DevOps Agent      ← deploy, CI/CD, environment
```

---

## 2. Tech Stack & Dependencies

**Backend**
- **Framework:** NestJS (TypeScript)
- **Runtime:** Node.js 20+ LTS
- **ORM:** TypeORM / Prisma
- **Validation:** class-validator + class-transformer (NestJS DTO)
- **Auth:** Passport.js + JWT
- **Queue:** BullMQ (Redis)
- **Cache:** Redis

**Frontend**
- **Framework:** Vue.js 3 (TypeScript, Composition API)
- **Build Tool:** Vite
- **State:** Pinia
- **HTTP Client:** Axios / ofetch
- **UI Library:** ตามที่ project กำหนด (PrimeVue, Vuetify, Headless UI)

**Python (Scripting / ML / Automation)**
- **Runtime:** Python 3.11+
- **Web (ถ้าต้องการ):** FastAPI
- **Data:** pandas, numpy, pydantic
- **Testing:** pytest

**Monorepo**
- **Manager:** Rush.js (Microsoft)
- **Package Manager:** pnpm (Rush default)

**Database**
- PostgreSQL 15+ (primary)
- Redis (cache + queue)
- Migrations บังคับทุกครั้งที่เปลี่ยน schema

**Tools & Quality**
| เครื่องมือ | หน้าที่ |
|---|---|
| ESLint | Linting (TypeScript + Vue) |
| Prettier | Code formatting |
| TypeScript | Type safety (strict mode) |
| Jest / Vitest | Unit & integration tests |
| Playwright | E2E tests |
| pytest | Python tests |
| GitHub Actions | CI/CD |
| Claude Code | AI pair programmer |

## 2.1 Environment Bootstrap (บังคับก่อนเริ่ม DEV/QA ทุกครั้ง)

ก่อนเขียนโค้ดหรือ test ใดๆ Agent ต้องทำขั้นตอนนี้ก่อน:

1. ตรวจสอบว่ามี `.env` หรือไม่ → ถ้าไม่มี **ถาม Tech Lead ทันที อย่าสร้างเอง**
2. ตรวจสอบ DB connection จริง:
   ```bash
   # TypeORM
   npx typeorm query "SELECT 1"

   # Prisma (ถ้าใช้ Prisma แทน TypeORM)
   npx prisma db execute --stdin <<< "SELECT 1"
   ```
   → ต้องได้ผลก่อนทำต่อ ถ้า error = DB ไม่พร้อม หยุดทันที

3. ตรวจสอบว่า migration รันแล้ว:
   ```bash
   # TypeORM
   npx typeorm migration:show

   # Prisma
   npx prisma migrate status
   ```
   → ต้องไม่มี pending migration ก่อนทำต่อ
4. ถ้าขั้นตอนใดล้มเหลว → **หยุดทันที + รายงาน Tech Lead** พร้อมบอกว่าขาดอะไร

> ห้ามข้ามขั้นตอนนี้แม้จะมั่นใจว่า environment พร้อม — ตรวจสอบทุกครั้ง

---

## 3. โครงสร้าง Repository

```
agentic-workflow/                ← repo นี้ (workflow documentation)
├── CLAUDE.md                    ← ไฟล์นี้ (อ่านก่อนทุกครั้ง)
├── .clinerules                  ← Identity & codex สำหรับ Cline
├── README.md                    ← Overview สำหรับ developer ใหม่
├── info.md                      ← Setup guide, ReAct pattern, checklist
├── _specs/                      ← MAS: spec + state (จับคู่ Playbook ดู _specs/README.md)
│   ├── state.json               ← phase, current_agent, branch
│   └── README.md                ← MAS↔Playbook, Git แยก branch, budget ~20 USD
│
├── docs/
│   ├── architecture/            ← ADR, system diagrams
│   │   └── system.md            ← high-level architecture
│   ├── agents/                  ← agent-specific playbooks
│   │   ├── pm.md
│   │   ├── architect.md
│   │   ├── developer.md
│   │   ├── qa.md
│   │   └── devops.md
│   └── workflows/               ← SDLC process docs
│
└── .github/
    └── workflows/               ← GitHub Actions CI/CD
```

**Rush Monorepo Project ที่อยู่ภายใต้ workflow นี้:**
```
{monorepo}/
├── rush.json                    ← Rush configuration
├── common/
│   ├── config/rush/             ← Rush plugin config
│   └── scripts/                 ← shared scripts
├── apps/
│   ├── api/                     ← NestJS backend
│   │   ├── src/
│   │   │   ├── modules/         ← feature modules
│   │   │   ├── common/          ← shared guards, pipes, interceptors
│   │   │   └── main.ts
│   │   └── package.json
│   ├── web/                     ← Vue.js frontend
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── composables/     ← reusable logic
│   │   │   ├── stores/          ← Pinia stores
│   │   │   ├── views/
│   │   │   └── main.ts
│   │   └── package.json
│   └── python-service/          ← Python service (ถ้ามี)
│       ├── src/
│       ├── tests/
│       └── pyproject.toml
└── libs/
    └── shared-types/            ← shared TypeScript types/interfaces
```

---

## 4. Context Files

ถ้าต้องการข้อมูลเพิ่มเติม ให้อ่านไฟล์เหล่านี้ก่อน:

| ไฟล์ | อ่านเมื่อ |
|------|-----------|
| `CLAUDE.md` | เริ่มทุก session — อ่านก่อนเสมอ |
| `docs/architecture/system.md` | ออกแบบ feature ใหม่, เลือก pattern |
| `docs/agents/pm.md` | ต้องการ PM Agent rules ละเอียด |
| `docs/agents/architect.md` | ต้องการ Architect Agent rules ละเอียด |
| `docs/agents/developer.md` | ต้องการ Developer Agent rules ละเอียด |
| `docs/agents/qa.md` | ต้องการ QA Agent rules ละเอียด |
| `docs/agents/devops.md` | ต้องการ DevOps Agent rules ละเอียด |
| `docs/workflows/` | ต้องการดู SDLC process ละเอียด |
| `rush.json` | เข้าใจ monorepo structure |
| `apps/api/src/` | เขียน/แก้ backend code |
| `apps/web/src/` | เขียน/แก้ frontend code |
| `.env.example` | ตั้งค่า environment variables |

---

## 5. Coding Patterns

### TypeScript / NestJS Standards

**Controller — ให้บาง (Thin Controller)**
```typescript
// ✅ ถูก: delegate ไป Service
@Post()
async create(@Body() dto: CreateProductDto): Promise<ProductResponseDto> {
  return this.productService.create(dto);
}

// ❌ ผิด: logic ใน controller
@Post()
async create(@Body() dto: any) {
  const product = await this.productRepo.save(dto);
  // ... 50 lines of business logic
}
```

**DTO Validation — class-validator เสมอ**
```typescript
// create-product.dto.ts
export class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(255)
  name: string;

  @IsNumber()
  @Min(0)
  price: number;
}
```

**Service Layer — Business Logic อยู่ที่นี่**
```typescript
// product.service.ts
@Injectable()
export class ProductService {
  constructor(private readonly productRepo: ProductRepository) {}

  async create(dto: CreateProductDto): Promise<Product> {
    const product = this.productRepo.create(dto);
    return this.productRepo.save(product);
  }
}
```

**Vue 3 — Composition API + Composables**
```typescript
// ✅ ถูก: logic แยกออกมาเป็น composable
// composables/useProducts.ts
export function useProducts() {
  const products = ref<Product[]>([]);
  const isLoading = ref(false);

  async function fetchProducts() {
    isLoading.value = true;
    products.value = await productApi.getAll();
    isLoading.value = false;
  }

  return { products, isLoading, fetchProducts };
}

// ❌ ผิด: logic ทั้งหมดอยู่ใน component
```

**Naming Conventions**
| Element | Convention | Example |
|---|---|---|
| Class / Interface | PascalCase | `ProductService`, `IProduct` |
| Function / Method | camelCase | `getUserById()` |
| Variable | camelCase | `productList` |
| DB Table / Column | snake_case | `product_categories` |
| Vue Component | PascalCase | `ProductCard.vue` |
| Composable | camelCase + use prefix | `useProducts()` |
| Env var | UPPER_SNAKE | `DATABASE_URL` |
| CSS class | kebab-case | `product-card` |

**Security Non-negotiables**
- ห้าม hardcode credentials — ใช้ `.env` เท่านั้น
- ตรวจ authorization ทุก endpoint ด้วย NestJS Guards
- Validate input ทุก request ผ่าน DTO + class-validator
- ไม่ใช้ `any` type ใน TypeScript (strict mode)
- ไม่ expose stack trace ใน production error response
- Sanitize output เพื่อป้องกัน XSS

---

## 6. Agentic SDLC Workflow

### Trigger Phrases — วิธีเรียก Agent

```
ORCH: [goal]    → Orchestrator Agent  (แจก task ให้ทีม)
PM: [goal]      → PM Agent            (วิเคราะห์ requirement, เขียน spec)
ARCH: [goal]    → Architect Agent     (ออกแบบ system, API contract)
DEV: [goal]     → Developer Agent     (เขียนโค้ด, แก้ bug)
QA: [goal]      → QA Agent            (ทดสอบ, ตรวจ quality)
DEVOPS: [goal]  → DevOps Agent        (deploy, CI/CD, environment)
```

**ตัวอย่าง:**
```
PM: สร้างระบบ user authentication รองรับ email + Google OAuth
ARCH: ออกแบบ API สำหรับ product catalog ที่รองรับ pagination และ filter
DEV: สร้าง NestJS module สำหรับ product CRUD พร้อม tests ให้ผ่านทั้งหมด
QA: ตรวจสอบ authentication flow ว่า edge cases ทุกกรณีได้รับการ handle
DEVOPS: ตั้งค่า GitHub Actions pipeline สำหรับ apps/api
```

### การสั่งงานแบบ Outcome-based (ReAct Pattern)

**หลักการ:** สั่ง *ผลลัพธ์* ที่ต้องการ ไม่สั่ง *วิธีทำ*

```
❌ แบบเก่า (Step-by-step):
"สร้าง entity ก่อน แล้วสร้าง repository แล้วสร้าง service..."

✅ แบบ Agentic (Outcome-based):
"DEV: สร้าง Product module ใน NestJS ให้ครบ รวม entity, repository,
service, controller, DTO, และ tests
ให้รัน npm test ผ่านทั้งหมดก่อนถือว่าเสร็จ
ถ้า error ให้แก้เองจนจบ"
```

### SDLC Flow สำหรับ Feature ใหม่

```
1. PLAN      → PM Agent: เขียน spec, user story, acceptance criteria
2. DESIGN    → Architect Agent: เลือก pattern, ERD, API contract
3. BUILD     → Developer Agent: เขียนโค้ด (TDD preferred)
4. TEST      → QA Agent: รัน test, regression, edge cases
5. REVIEW    → Tech Lead: ดู output, approve/reject
6. DEPLOY    → DevOps Agent: CI/CD pipeline, staging → prod
7. RECAP     → Orchestrator: สรุปบทเรียน, อัปเดต docs
```

### Git Workflow

```bash
# Branch naming
feature/short-description     # งานใหม่
fix/issue-description         # bug fix
refactor/component-name       # refactor
chore/task-description        # maintenance

# Commit message format (Conventional Commits)
type(scope): short description

# Types: feat, fix, refactor, test, chore, docs, perf
# Examples:
feat(product): add CRUD module with NestJS service and tests
fix(auth): handle JWT expiry edge case in guard
```

**กฎ Git:**
- ห้าม commit โดยตรงที่ `main`
- PR ต้องผ่าน CI (tests + lint + type-check) ก่อน merge
- Commit message ต้องสื่อความหมาย

**Mini project + MAS + Git (แยก branch ตามฟีเจอร์):**  
ใช้ `_specs/` สำหรับ MAS (RESEARCHER→ARCHITECT→CODER→QA), ส่งงานผ่าน spec ใน `_specs/`. แยก branch เป็น `feature/<ชื่อ>` ต่อหนึ่งฟีเจอร์. รายละเอียด + วิธีควบคุม budget ~20 USD ดู **`_specs/README.md`**

---

## 7. Agent Rules

### 7.1 Orchestrator Agent
**Role:** รับ intent จาก Tech Lead, แจก task ให้ agent ที่เหมาะสม, ติดตามสถานะ

**Trigger:** `ORCH: [goal]`

**Rules:**
- อ่าน CLAUDE.md ก่อนทุก session
- แปลง high-level goal เป็น structured task สำหรับแต่ละ agent
- ไม่ implement เอง — delegate เท่านั้น
- รายงาน progress และ blocker ให้ Tech Lead ทราบ
- เมื่อจบ task: สรุปสิ่งที่ทำ, decision ที่เลือก, สิ่งที่ต้องระวัง

---

### 7.2 PM Agent (Product Manager)
**Role:** วิเคราะห์ requirement, เขียน spec ที่ developer implement ได้

**Trigger:** `PM: [goal]`

**Rules:**
- เขียน User Story format: `As a [role], I want [feature], so that [benefit]`
- กำหนด Acceptance Criteria ที่ testable (Given/When/Then)
- ถ้า requirement ไม่ชัด: ถามคำถาม clarifying ก่อน — ไม่ assume
- ส่งมอบ: `spec.md` ที่ architect และ developer อ่านแล้วเข้าใจได้ทันที
- ไม่ตัดสินใจ technical — ส่งต่อให้ Architect

**Output format:**
```markdown
## Feature: [ชื่อ]
**User Story:** As a ... I want ... so that ...
**Acceptance Criteria:**
- [ ] Given ... When ... Then ...
**Out of scope:** ...
**Dependencies:** ...
```

---

### 7.3 Architect Agent
**Role:** ออกแบบ system, เลือก pattern, กำหนด API contract

**Trigger:** `ARCH: [goal]`

**Rules:**
- อ่าน spec จาก PM Agent และ `docs/architecture/system.md` ก่อนออกแบบ
- ออกแบบ ERD สำหรับทุก database change
- กำหนด API contract (endpoint, request DTO, response DTO, error codes)
- เลือก pattern ที่ maintainable ก่อน ไม่ over-engineer
- เขียน Architecture Decision Record (ADR) สำหรับ decision สำคัญ
- ตรวจว่า design ไม่ขัดกับ security rules ใน section 5

**Output format:**
```markdown
## Architecture: [Feature]
**Pattern:** Module / Service / Repository
**ERD:** (diagram หรือ table list)
**API Contract:**
  POST /api/products
  Request:  CreateProductDto { name: string, price: number }
  Response: ProductResponseDto { id: string, name: string, ... }
**ADR:** ทำไมถึงเลือก pattern นี้
```

---

### 7.4 Developer Agent
**Role:** เขียนโค้ดตาม spec และ architecture, แก้ bug ให้จบในตัวเอง

**Trigger:** `DEV: [goal]`

**Rules:**
- อ่าน spec (PM) และ architecture (Architect) ก่อนเริ่มเขียนโค้ด
- ทำตาม coding patterns ใน section 5 อย่างเคร่งครัด
- ใช้ TypeScript strict mode — ห้ามใช้ `any`
- เขียน test ควบคู่กับ implementation (TDD preferred)
- รัน `npm test` และแก้จนผ่านทั้งหมดก่อนส่งมอบ
- รัน `npm run lint` และ `npm run format` ก่อน commit
- ถ้า error: วิเคราะห์ → แก้ → รันใหม่ → วนจนจบ (ไม่หยุดกลางทาง)
- ห้าม hardcode, ห้าม magic number — ใช้ constants หรือ config

### กฎ "No Fake Green" — บังคับเคร่งครัด
- ห้ามรายงานว่า test ผ่าน ถ้าไม่ได้รัน `npm test` จริง
- ห้าม mock DB ใน integration test — ใช้ test database จริงเท่านั้น
- ถ้า DB ไม่พร้อม → หยุด + ถาม อย่าเดา อย่า mock ไปก่อน
- ต้องแนบ actual output จาก terminal มาด้วยทุกครั้ง:
  ```
  ✅ "Tests: 5 passed, 0 failed (output จาก jest)"
  ❌ "น่าจะผ่านเพราะ logic ถูกต้อง"
  ```

**Workflow (NestJS):**
```
1.  อ่าน spec + architecture
2.  สร้าง migration / schema (ถ้ามี DB change)
2.5 ✅ ยืนยัน Bootstrap (section 2.1) ผ่านแล้ว — ถ้ายังไม่ผ่าน หยุดที่นี่
3.  สร้าง entity / model
4.  สร้าง repository (ถ้าใช้ Repository pattern)
5.  สร้าง service (business logic)
6.  สร้าง DTO (request + response)
7.  สร้าง controller (บาง)
8.  register module
9.  เขียน tests (unit + integration)
10. รัน tests → แก้จนผ่าน
11. รัน lint + format → commit
```

**Workflow (Vue.js):**
```
1. อ่าน spec + API contract
2. สร้าง composable (business logic + API calls)
3. สร้าง Pinia store (ถ้าต้องการ shared state)
4. สร้าง component (ดึงจาก composable)
5. เขียน tests (Vitest)
6. รัน tests → แก้จนผ่าน
7. รัน lint + format → commit
```

---

### 7.5 QA Agent
**Role:** ตรวจสอบ quality, รัน test scenarios, หา edge case

**Trigger:** `QA: [goal]`

**Rules:**
- รัน full test suite และรายงาน coverage
- ทดสอบ edge cases ที่ developer อาจมองข้าม
- ตรวจ security: injection, XSS, unauthorized access, broken auth
- ตรวจ response format ตรงกับ API contract ที่ Architect กำหนด
- ถ้าพบ bug: รายงานพร้อม reproduction steps และส่ง Developer แก้
- ไม่ approve ถ้า test ยังไม่ผ่านหรือ coverage ต่ำกว่า 80%

**Checklist (ต้องผ่านทั้งหมด):**
```
□ npm test — ผ่านทุก test
□ Happy path ทำงานถูกต้อง
□ Validation reject invalid input พร้อม error message ที่ถูกต้อง
□ Unauthorized / unauthenticated user ถูก block
□ Edge cases (empty data, max length, null, special characters)
□ API response format ตรงกับ contract
□ ไม่มี N+1 query (ตรวจด้วย query logging)
□ TypeScript ไม่มี type error (tsc --noEmit ผ่าน)
```

---

### 7.6 DevOps Agent
**Role:** จัดการ CI/CD, environment, deployment

**Trigger:** `DEVOPS: [goal]`

**Rules:**
- ทุก push ต้อง trigger CI: test → lint → type-check → build
- ไม่ deploy branch ที่ test ไม่ผ่าน
- จัดการ `.env` แยกตาม environment (local/staging/production)
- ห้าม production credentials อยู่ใน repository
- ใช้ GitHub Secrets สำหรับ sensitive values
- Rollback plan ต้องมีก่อน deploy production ทุกครั้ง
- Monitor หลัง deploy: ตรวจ error log 15 นาที

**GitHub Actions Template:**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm test -- --coverage
      - run: npm run build
```

---

## 8. Definition of Done

Feature ถือว่า **Done** เมื่อผ่าน **ทุกข้อ** ต่อไปนี้:

### Code Quality
- [ ] โค้ดตาม patterns ใน section 5 (thin controller, service layer, DTO)
- [ ] ไม่มี hardcoded credentials, secrets, หรือ magic numbers
- [ ] TypeScript strict — ไม่มี `any`, ไม่มี type error (`tsc --noEmit` ผ่าน)
- [ ] `npm run lint` ผ่านไม่มี error
- [ ] `npm run format` รันแล้วโค้ดไม่เปลี่ยน (Prettier-compliant)

### Testing
- [ ] Unit tests ครอบคลุม business logic หลัก (services, utilities)
- [ ] Integration tests ครอบคลุม API endpoints ทุกตัว
- [ ] `npm test` ผ่าน 100% — ไม่มี skipped หรือ failed
- [ ] Test coverage ≥ 80% สำหรับ code ที่เขียนใหม่

### Functionality
- [ ] Acceptance criteria ทุกข้อจาก PM Agent ผ่าน
- [ ] Edge cases ที่ QA Agent กำหนดผ่าน
- [ ] API response format ตรงกับ contract จาก Architect

### Security
- [ ] Authorization guard ผ่านทุก endpoint ที่ต้องการ
- [ ] Input validation ผ่าน DTO + class-validator
- [ ] ไม่มี injection vulnerability
- [ ] ไม่มี sensitive data ใน response ที่ไม่จำเป็น

### Documentation & Git
- [ ] Commit message อยู่ใน Conventional Commits format
- [ ] `.env.example` อัปเดตถ้ามี env var ใหม่
- [ ] PR description อธิบาย what และ why
- [ ] Migration file ถูก commit พร้อมกับ code (ถ้ามี DB change)

### Deployment
- [ ] CI pipeline ผ่านบน GitHub Actions (test + lint + type-check + build)
- [ ] รัน migration บน staging และ verify แล้ว
- [ ] Tech Lead review และ approve PR แล้ว

### Deployment Auto-Steps (DevOps Agent ทำทุกครั้งที่เสร็จ)

**ตรวจก่อนว่ามี Docker หรือไม่:**
```bash
docker info 2>/dev/null && echo "Docker available" || echo "No Docker"
```

**ถ้ามี Docker:**
- [ ] รัน `docker compose up --build -d`
- [ ] ตรวจ health check: `curl http://localhost:3000/health` → ต้องได้ 200
- [ ] รัน smoke test กับ URL จริง (ไม่ใช่ mock)

**ถ้าไม่มี Docker (Node + PostgreSQL local):**
- [ ] รัน `npm run start:dev` (NestJS) และ `npm run dev` (Vue)
- [ ] ตรวจ health check: `curl http://localhost:3000/health` → ต้องได้ 200
- [ ] ยืนยัน DB connection ด้วย Bootstrap ขั้นตอน 2 (section 2.1)

**ส่ง deploy report ให้ Tech Lead ทุกกรณี:**
```
✅ Deploy สำเร็จ  [Docker / Local — ระบุด้วย]
API:      http://localhost:3000
Frontend: http://localhost:5173
DB Admin: http://localhost:5050 (pgAdmin — ถ้ามี Docker)
```

---

## Quick Reference

**Rush / Node.js**
| คำสั่ง | ทำอะไร |
|---|---|
| `rush install` | Install all dependencies |
| `rush build` | Build all projects |
| `rush test` | Run all tests |
| `cd apps/api && npm test` | Run API tests only |
| `cd apps/web && npm test` | Run web tests only |
| `npm run lint` | Run ESLint |
| `npm run format` | Run Prettier |
| `npm run type-check` | TypeScript check (no emit) |

**NestJS**
| คำสั่ง | ทำอะไร |
|---|---|
| `npm run start:dev` | Start NestJS dev server |
| `npx nest generate module products` | Generate NestJS module |
| `npx typeorm migration:generate` | Generate migration |
| `npx typeorm migration:run` | Run migrations |

**Vue.js**
| คำสั่ง | ทำอะไร |
|---|---|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Production build |
| `npm run preview` | Preview production build |

**Python**
| คำสั่ง | ทำอะไร |
|---|---|
| `pytest` | Run all tests |
| `pytest tests/test_product.py` | Run specific test file |
| `ruff check .` | Lint Python code |
| `black .` | Format Python code |

---

## สื่อสาร

- **ภาษา:** อธิบายและสรุปเป็นภาษาไทย, โค้ดและ technical terms เป็นภาษาอังกฤษ
- **รายงาน:** ทุก agent รายงาน output ชัดเจน — ทำอะไรไป, ผลลัพธ์คืออะไร, มี blocker ไหม
- **Escalate:** ถ้า agent ติดปัญหาที่แก้เองไม่ได้ใน 2 รอบ — escalate ให้ Tech Lead ทันที อย่าฝืนทำต่อ

---

*ไฟล์นี้ maintain โดย Tech Lead — Agent ทุกตัวอ่าน แต่ไม่แก้ไขเอง*
*อัปเดตล่าสุด: 2026-03-12*
