# เริ่มใช้งาน .NET-to-Go Converter

คู่มือนี้อธิบายโครงสร้างและวิธีใช้ชุด agents/skills สำหรับวิเคราะห์ .NET API แล้วแปลงเฉพาะ API route ที่เลือกไปเป็น Go โดยใช้ Go template ที่ผู้ใช้ระบุ

## 1. โครงสร้าง Repository

```text
.
├── START.md                         คู่มือเริ่มต้นใช้งาน
├── README.md                        ภาพรวมและข้อกำหนดหลัก
├── AGENTS.md                        กติกาสำหรับ agent ใน workspace
├── CONTEXT.md                       glossary ของคำศัพท์โครงการ
├── skills-lock.json                 versions ของ skills เดิมใน repository
├── schemas/
│   ├── conversion-manifest.schema.json
│   └── route-artifact.schema.json
├── examples/
│   └── conversion-manifest.example.json
├── docs/adr/
│   ├── 0001-route-scoped-conversion-with-manifest.md
│   └── 0002-contract-first-with-code-evidence.md
└── .agents/
    ├── agents/
    │   └── dotnet-to-go-converter.agent.md
    └── skills/
        ├── dotnet-source-analysis/SKILL.md
        ├── go-template-profile/SKILL.md
        ├── dotnet-to-go-planning/SKILL.md
        ├── api-route-conversion/SKILL.md
        └── api-parity-validation/SKILL.md
```

## 2. สิ่งที่ต้องเตรียม

ต้องมี:

- path ของ .NET API source เช่น `D:/work/orders-api`
- path ของ Go template เช่น `D:/templates/go-service`
- path ของ output workspace ซึ่งควรเป็น directory ใหม่
- OpenAPI/Swagger หรือเอกสาร API ถ้ามี
- environment สำหรับรัน tests ของ source และ target
- target datastore ต่อ route: `sql-server-existing` หรือชื่อ datastore ใหม่
- spec ที่อ่านได้จริงและมี route ที่เลือก เช่น OpenAPI/Swagger

ไม่ควรใส่ connection string, token, certificate หรือ production secret ลงใน repository หรือ artifact

## 3. วิธีติดตั้งบนเครื่องอื่น

1. Clone หรือ copy repository นี้ไปยังเครื่องปลายทาง
2. ตรวจให้แน่ใจว่าโฟลเดอร์ `.agents` และ `schemas` ถูก copy มาครบ
3. เปิด workspace นี้ด้วย agent runtime ที่รองรับ `.agents/agents` และ `.agents/skills`
4. ให้ runtime อ่าน [AGENTS.md](AGENTS.md) และ [CONTEXT.md](CONTEXT.md)
5. ตรวจว่า agent มองเห็น `dotnet-to-go-converter` และ skills ทั้ง 5 รายการ

ผลลัพธ์อาจไม่เหมือนกันแบบ byte-for-byte เพราะ model output มีความแปรผัน แต่ workflow, input fingerprints, decisions, evidence และ validation จะถูกบังคับให้มีรูปแบบเดียวกัน

## 4. วิธีเรียกใช้งาน

เรียก agent `dotnet-to-go-converter` พร้อมข้อมูลนี้:

```text
ใช้ agent dotnet-to-go-converter

sourcePath: D:/work/orders-api
templatePath: D:/templates/go-service
specPaths:
  - D:/work/orders-api/openapi.json
outputPath: D:/work/orders-api-go
routeSelection:
  - orders.get-by-id
targetDatastore:
  orders.get-by-id: sql-server-existing
```

ถ้ายังไม่ทราบ route ID ให้ใช้:

```text
วิเคราะห์ source และ spec ก่อน แล้วเสนอรายการ route พร้อม operationId,
HTTP method, path, dependencies, blockers และ target datastore ที่ต้องตัดสินใจ
```

`specPaths` เป็น required input ต้องมีไฟล์ที่อ่านและ parse ได้จริง และต้องมี route ที่เลือกอยู่ใน spec หาก spec หาย, parse ไม่ได้ หรือไม่พบ route ให้หยุดเป็น Blocker ห้ามใช้ source เดา route โดยอัตโนมัติ เว้นแต่ระบุ `allowInferredContract: true` อย่างชัดเจน

## 5. Workflow

### Stage 1: Analyze .NET source

Skill: `dotnet-source-analysis`

Agent จะอ่าน controllers/minimal APIs, DTOs, validators, middleware, auth policies, database access, integrations, configuration และ tests แล้วสร้างรายการ Route พร้อม evidence และ fingerprints

**จบ stage เมื่อ:** ทุก Route มี canonical route ID, contract, behavior evidence, dependencies และ Blocker ที่ระบุชื่อชัดเจน

### Stage 2: Profile Go template

Skill: `go-template-profile`

Agent จะอ่าน module layout, router/framework, dependency injection, configuration, logging, auth, datastore, migrations, testing และ validation commands ของ Go template

**จบ stage เมื่อ:** มี Template profile ที่มี evidence และคำสั่งตรวจสอบที่รันได้หรือถูกระบุว่า unavailable

### Stage 3: Create conversion plan

Skill: `dotnet-to-go-planning`

Agent จะเสนอ route selection และตัดสินใจต่อ route เรื่อง:

- request/response/type mapping
- handler, service, repository และ middleware
- auth, claims, roles และ policy
- target datastore
- schema mapping และ migration plan
- compatibility/read strategy
- external dependencies, transaction และ idempotency
- tests, parity fixtures และ acceptance criteria

**จุดอนุมัติ:** ผู้ใช้ต้อง approve analysis และ plan ก่อนเริ่มเขียน code

### Stage 4: Convert one Route

Skill: `api-route-conversion`

Agent จะ clone Go template แบบเต็มไปยัง `outputPath` แล้วคง baseline ทั้งหมดไว้ เช่น root tooling/configuration, `Makefile`, build, `go.mod`, `go.sum`, `scripts`, `sqlc.yaml`, `third_party` และ libraries ของ `kkpfg-kkpb` จากนั้นคำนวณ dependency closure ของ Route ที่เลือก และ prune เฉพาะพื้นที่ mutable คือ `cmd`, `db`, `docs` และ `internal` ก่อน implement ทีละ Route ที่ approved โดยรักษา HTTP contract, validation, errors, auth, transactions, side effects และ idempotency

route อื่น, handler อื่น, generated model ที่ไม่เกี่ยวข้อง, domain service, external client, SQL query, seed และ migration ที่ไม่อยู่ใน dependency closure ต้องถูกตัดออกจาก outputเมื่ออยู่ในพื้นที่ mutable เท่านั้น ห้ามลบ root tooling, `scripts`, `sqlc.yaml`, `third_party` หรือ library/dependency ของ `kkpfg-kkpb`

**จุดอนุมัติ:** ต้อง approve ก่อน implementation ของแต่ละ Route

### Stage 5: Validate parity

Skill: `api-parity-validation`

Agent จะรัน formatter, linter, static analysis, build, unit/integration tests และ parity fixtures ใน non-production environment

เปรียบเทียบ:

- HTTP status และ headers
- serialized response body
- validation และ error contract
- auth outcomes
- datastore side effects
- transaction outcome
- external side effects
- idempotency

**จบ stage เมื่อ:** Route เป็น `validated` เมื่อ checks และ fixtures ผ่านทั้งหมด หากไม่ผ่านจะเป็น `blocked` พร้อม evidence ที่ reproduce ได้

## 6. Approval Gates

มี approval 3 ระดับ:

1. **Analysis approval**: ยืนยันว่า source/spec analysis ถูกต้อง
2. **Plan approval**: ยืนยัน route selection, mappings และ target datastore
3. **Implementation approval**: อนุญาตให้เขียน code ของ Route ที่เลือก

หาก source กับ spec ขัดแย้งกัน หรือพบ behavior ที่ mapping ไม่ได้ ให้สร้าง Blocker และหยุด Route นั้นจนกว่าจะมี decision

## 7. Artifacts และสถานะ

Conversion manifest ต้องอ้างอิง:

- source/template/spec paths และ content fingerprints
- agent และ skill versions
- route selection และ target datastore
- approval status
- artifact paths
- validation commands และผลลัพธ์
- blockers และ parity reports

สถานะ Route:

```text
proposed -> planned -> approved -> implemented -> validated
                                      └────────-> blocked
```

แต่ละ Route ควรมี artifact อย่างน้อย:

```text
routes/<route-id>/
├── analysis.md
├── plan.md
├── decisions.md
├── implementation.patch
├── parity-fixtures/
├── parity-report.md
└── validation.json
```

ตรวจรูปแบบด้วย:

- [conversion-manifest.schema.json](schemas/conversion-manifest.schema.json)
- [route-artifact.schema.json](schemas/route-artifact.schema.json)
- [conversion-manifest.example.json](examples/conversion-manifest.example.json)

## 8. Resume และเปลี่ยนเครื่อง

ก่อนเริ่มแต่ละ stage ให้ agent คำนวณ fingerprint ใหม่ หาก source, spec หรือ template เปลี่ยน:

1. แสดง affected Routes
2. เปรียบเทียบ diff กับ analysis/plan เดิม
3. ขอ approval ใหม่เฉพาะส่วนที่ได้รับผลกระทบ
4. รักษา artifact ของ Route ที่ไม่เกี่ยวข้อง

ใช้ path แบบ relative ใน artifact เพื่อให้ manifest ย้ายเครื่องได้ และเก็บ absolute path ไว้เฉพาะ runtime context

## 9. คำสั่งสำหรับผู้ใช้ที่พบบ่อย

```text
แสดง route ทั้งหมดที่ค้นพบจาก source และ spec
```

```text
วางแผนเฉพาะ GET /api/orders/{id} และเสนอ target datastore 2 ทางเลือก
```

```text
แปลง route orders.get-by-id ตาม plan ที่ approved เท่านั้น
```

```text
รัน parity validation ของ route orders.get-by-id และรายงาน blocker ทั้งหมด
```

```text
ตรวจว่า source หรือ template เปลี่ยนจาก manifest เดิมหรือไม่ แล้วเสนอ routes ที่ต้อง re-approve
```

## 10. หลักการสำคัญ

- วิเคราะห์ก่อนแก้ไข
- ใช้ spec เป็น public contract และ code เป็น behavioral evidence
- แปลงทีละ Route
- ทุก decision ต้องมี evidence หรือ approval
- เลือก target datastore ราย Route
- clone template และสร้าง patch แทนการแก้ template ต้นฉบับ
- data migration เป็น plan แยกจาก route conversion
- compilation อย่างเดียวไม่ถือว่า parity ผ่าน
- secret ใช้ผ่าน reference และต้อง redact ใน output
