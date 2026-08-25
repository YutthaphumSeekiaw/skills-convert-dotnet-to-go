# Prompt: Create .NET-to-Go Converter Agents and Skills

คัดลอก prompt นี้ไปใช้กับ coding agent เพื่อสร้าง project สำหรับแปลง .NET API เป็น Go แบบ reusable และย้ายไปใช้บนเครื่องอื่นได้

```text
คุณเป็น senior software architect และ coding-agent designer

จงสร้าง project สำหรับ .NET-to-Go API conversion agents และ skills ให้ใช้งานซ้ำได้บนหลายเครื่อง โดย output ต้องเป็นไฟล์จริงใน workspace ไม่ใช่เพียงคำอธิบาย

## เป้าหมาย

สร้าง orchestrator agent และ skills ที่ทำงานตามลำดับนี้:

1. วิเคราะห์ .NET API source และ API specification
2. วิเคราะห์ Go template และสร้าง template profile
3. แสดงรายการ API route ทั้งหมดจาก specification ให้ผู้ใช้เลือก
4. ให้ผู้ใช้เลือก target datastore ราย route
5. สร้าง conversion plan และรอ approval
6. clone Go template แบบเต็ม
7. prune เฉพาะ application content ที่ไม่เกี่ยวข้อง
8. แปลง route ที่ approved เป็น Go
9. บันทึกสถานะ route ลง Conversion manifest
10. แสดงรายการ route ที่ยังไม่เสร็จให้ผู้ใช้เลือกทำต่อใน project เดิม
11. สร้าง tests, parity fixtures และ validation report

## Project naming rule

กำหนด `projectName` จากชื่อ folder สุดท้ายของ `outputPath` เสมอ เช่น `D:\work\dms-ocr` ต้องใช้ `dms-ocr` เป็นชื่อ project, Go module, service name, command directory, generated output และเอกสารที่เกี่ยวข้อง

- เปลี่ยนชื่อ project/template เดิมให้เป็น `projectName` ในจุดที่เป็นชื่อของ project
- อัปเดต `go.mod` module path และ internal imports ให้สอดคล้องกับ `projectName`
- อัปเดต command folder, executable name, Docker/service name, README, Makefile และ OpenAPI metadata ตามความเหมาะสม
- คง external imports เช่น `github.com/kkpfg-kkpb/*` ไว้เหมือนเดิม ห้ามเปลี่ยนเป็นชื่อ output project
- ถ้า outputPath มี project เดิมอยู่แล้ว ให้ใช้ชื่อจาก output folder และตรวจชื่อเดิมที่ค้างอยู่ก่อนแก้

## Input contract

รองรับ input ต่อไปนี้:

- sourcePath: path ของ .NET API source
- templatePath: path ของ Go template หรือ directory ที่มี module อยู่ภายใน
- specPaths: รายการ path ของ OpenAPI/Swagger/documentation
- allowInferredContract: boolean
- outputPath: path ของ output workspace
- routeSelection: route IDs หรือ operationIds
- targetDatastore: mapping ระหว่าง route ID กับ datastore

ตรวจสอบ path ทุกตัวก่อนเริ่มงาน

ถ้า `allowInferredContract` เป็น false ต้องมี specPaths อย่างน้อยหนึ่งไฟล์ ถ้าเป็น true จึงอนุญาต `specPaths: []` และใช้ source เป็น contract โดยต้องระบุโหมด inferred ใน artifacts

## Spec-first rule

ต้องอ่านและ parse ทุกไฟล์ใน specPaths ก่อนเสนอหรือรับ routeSelection และต้องสร้าง route catalog จาก spec

- route identity ใช้ operationId เป็นหลัก
- ถ้าไม่มี operationId ให้ใช้ HTTP method + normalized path
- routeSelection ต้องมีอยู่ใน specification
- หลัง parse spec ต้องแสดง route catalog ให้ผู้ใช้เห็น โดยมี route ID, operationId, method, path, summary, dependencies และสถานะปัจจุบัน
- ถ้าไม่ได้ระบุ routeSelection ให้รอผู้ใช้เลือกจาก route catalog ห้ามเลือก route แทนผู้ใช้
- ถ้าระบุหลาย route ให้ถามยืนยันลำดับการทำงานและทำทีละ route
- spec หาย, parse ไม่ได้ หรือไม่พบ route ให้สร้าง Blocker และหยุดก่อน planning
- ใช้ source-derived contract ได้ก็ต่อเมื่อ allowInferredContract เป็น true อย่างชัดเจน
- ระบุใน artifact เสมอว่า contract มาจาก declared spec หรือ inferred source
- หาก spec กับ source ขัดแย้งกัน ให้รายงาน conflict และรอ decision ห้ามเดา

## Template-first rule

ต้อง clone Go template แบบเต็มเป็น baseline ห้ามสร้าง minimal standalone project แทน template

ต้องคงไว้เสมอ:

- root configuration และ tooling
- .gitignore
- .golangci.yml
- .pre-commit-config.yaml
- Makefile
- README.md
- STRUCTURE.md
- build/
- commitlint.config.js
- go.mod
- go.sum
- scripts/
- sqlc.yaml
- third_party/
- private libraries และ dependencies โดยเฉพาะ namespace github.com/kkpfg-kkpb/

ตรวจสอบ module root จริงจาก go.mod หาก templatePath ชี้ไปยัง directory ชั้นนอก

## Code generation and database rule

หลังเลือก Route และสร้าง/ปรับ OpenAPI document แล้ว ต้องรัน `gen-server` ของ template เพื่อ generate API server/types จาก OpenAPI ห้ามเขียน generated API files ด้วยมือ

เลือก database strategy ตาม `targetDatastore`:

- PostgreSQL: ใช้ `sqlc` ผ่าน `gen-db` ของ template, เก็บ SQL queries/migrations ตาม convention และตรวจ generated code
- SQL Server: ใช้ database access แบบปกติของ template เช่น `database/sql`, driver, repository หรือ native query ที่ template รองรับ ห้ามเรียก `sqlc` ถ้า template ไม่มี SQL Server workflow ที่รองรับ
- หาก route ไม่ใช้ database ให้ไม่สร้าง database code แต่ยังคง database tooling ที่เป็น baseline ของ template

บันทึก database strategy, generation commands, driver, query files และ migration decisions ใน Conversion plan และ scope report

## Architecture rule

โครงสร้าง output ต้องยึด hexagonal architecture และ conventions ของ Go template:

- domain/application อยู่ด้านในและไม่ผูกกับ framework หรือ database driver
- ports เป็น interfaces ที่กำหนด inbound และ outbound contracts
- adapters แยก HTTP handler, persistence, external clients และ infrastructure
- wiring อยู่ที่ composition root/command package
- dependency direction ชี้เข้าหา domain/application
- ใช้ชื่อและ folder layout ของ template เมื่อ template มี convention อยู่แล้ว

หาก template ใช้ชื่อ `handler`, `service`, `repository`, `ports` หรือ `internal/domain` ให้ map เป็น hexagonal role ใน profile และห้ามสร้าง architecture แบบใหม่โดยไม่มีเหตุผลที่บันทึกไว้

## Pruning rule

อนุญาตให้แก้ไขหรือลบเฉพาะ mutable application areas:

- cmd/
- db/
- docs/
- internal/

ต้องคง directory หลัก `cmd/`, `db/`, `docs/` และ `internal/` ไว้เสมอ แม้จะลบไฟล์ application ภายในบางส่วนได้

ก่อน prune ให้คำนวณ dependency closure ของ route ที่เลือก

ลบหรือแก้เฉพาะสิ่งที่อยู่นอก dependency closure เช่น:

- route registrations อื่น
- handlers อื่น
- generated API models/operations อื่น
- domain services อื่น
- external clients อื่น
- SQL queries อื่น
- seed data อื่น
- migrations อื่น

ห้ามลบสิ่งต่อไปนี้แม้ route จะไม่ได้ import โดยตรง:

- scripts/
- sqlc.yaml
- third_party/
- Makefile และ root tooling
- go.mod และ go.sum
- private KKP libraries/dependencies

สร้าง scope-report.md ที่แยกชัดเจนว่าไฟล์ใดเป็น preserved baseline และไฟล์ใดถูก prune

## Approval gates

หยุดรอผู้ใช้ตามลำดับ:

1. หลัง source/spec analysis: ขอ analysis approval
2. หลัง conversion plan: ขอ plan approval
3. ก่อนเขียนหรือแก้ Go implementation: ขอ implementation approval

ห้ามข้าม approval gate

## Incremental route queue

ใช้ Conversion manifest เป็น state machine ของ project เดิม:

```text
discovered -> selected -> planned -> approved -> implemented -> validated
                                                        └──────> blocked
```

หลังจบแต่ละ route ให้ update manifest แล้วแสดง completed Routes และ remaining Routes แยกกัน ผู้ใช้เลือกได้เฉพาะ remaining Routes เพื่อทำ route ถัดไปใน outputPath เดิม ห้าม clone template ใหม่หรือเขียนทับ implementation และ artifacts ของ route ที่เสร็จแล้ว

## Required agents and skills

สร้างไฟล์ต่อไปนี้:

- AGENTS.md
- CONTEXT.md
- docs/adr/0001-route-scoped-conversion-with-manifest.md
- docs/adr/0002-contract-first-with-code-evidence.md
- docs/adr/0003-preserve-template-baseline.md
- .agents/agents/dotnet-to-go-converter.agent.md
- .agents/skills/dotnet-source-analysis/SKILL.md
- .agents/skills/go-template-profile/SKILL.md
- .agents/skills/dotnet-to-go-planning/SKILL.md
- .agents/skills/api-route-conversion/SKILL.md
- .agents/skills/api-parity-validation/SKILL.md
- schemas/conversion-manifest.schema.json
- schemas/route-artifact.schema.json
- examples/conversion-manifest.example.json
- START.md

## Artifact requirements

สร้าง Conversion manifest ที่เก็บ:

- input paths และ content fingerprints
- allowInferredContract
- agent/skill versions
- selected routes
- target datastore ราย route
- approval states
- artifact paths
- validation commands/results
- blockers

แต่ละ route ต้องมีอย่างน้อย:

- analysis.md
- plan.md
- decisions.md
- route-artifact.json
- scope-report.md
- implementation.patch
- parity-fixtures/
- parity-report.md
- validation.json

Machine-readable artifacts ต้อง validate ด้วย JSON Schema

## Conversion behavior

สำหรับแต่ละ route ที่ approved:

- รักษา HTTP method และ path
- รักษา request/response schema
- รักษา status codes, headers และ serialization
- รักษา validation และ error contract
- รักษา auth, claims, roles และ policies
- รักษา transaction, side effects และ idempotency
- ใช้ target datastore ตาม decision
- หาก route ไม่มี database behavior ห้ามเพิ่ม database code เพียงเพราะ template มี database
- data migration ต้องเป็น plan/artifact แยก ไม่ execute อัตโนมัติ
- unsupported behavior หรือ evidence ไม่พอให้สร้าง Blocker และหยุด route
- route ที่ทำเสร็จแล้วต้องไม่ถูก implement ซ้ำ เว้นแต่ผู้ใช้สั่ง re-convert หลัง fingerprint เปลี่ยน

## Validation

อ่าน validation commands จาก template profile และรันเท่าที่ environment รองรับ เช่น:

- gofmt หรือ make fmt
- go test ./...
- go vet ./...
- make lint
- go build
- OpenAPI generation
- `gen-server` จาก OpenAPI
- `gen-db` และ `sqlc` เมื่อ target datastore เป็น PostgreSQL
- native SQL Server generation/access เมื่อ target datastore เป็น SQL Server
- focused route tests
- HTTP parity tests

Parity ต้องเปรียบเทียบ observable behavior:

- HTTP status
- headers
- response body
- validation/errors
- auth outcomes
- datastore side effects
- external side effects
- transaction outcome
- idempotency

ห้ามถือว่า compile ผ่านเท่ากับ parity ผ่าน

## Reproducibility and portability

- ใช้ relative paths ใน artifacts เท่าที่ทำได้
- เก็บ absolute paths เฉพาะ runtime context
- ใช้ content fingerprints ตรวจ source/spec/template เปลี่ยน
- ถ้า input เปลี่ยน ให้แสดง affected routes และขอ re-approval เฉพาะส่วนที่กระทบ
- ถ้า resume project เดิม ให้โหลด manifest เดิมและแสดง remaining route queue ก่อนทำงานต่อ
- ห้ามบันทึก secret, token, password, certificate หรือ production connection string
- redact secret ใน logs และ reports
- ผลลัพธ์ต้องมี structured artifacts และ evidence เพื่อให้เครื่องอื่น resume ได้
- ห้ามรับประกัน byte-for-byte model output แต่ต้องทำให้ workflow และ contracts เหมือนกัน

## Domain vocabulary

ใช้คำเหล่านี้เป็น canonical terms:

- Source API
- Target service
- Route
- Target datastore
- Template profile
- Conversion plan
- Conversion manifest
- Parity fixture
- Parity report
- Blocker

หลีกเลี่ยงคำว่า legacy API, Go rewrite, Go database, endpoint, TODO เมื่อหมายถึง domain concepts ข้างต้น

## Implementation process

1. อ่าน workspace และตรวจ conventions เดิม
2. สร้าง CONTEXT.md เมื่อเริ่มมีศัพท์ domain ที่ตกลงแล้ว
3. สร้าง ADR เฉพาะ decisions ที่ reverse ยากและมี trade-off
4. สร้าง agent/skills ตามรายการ
5. สร้าง schemas และ example manifest
6. ตรวจ frontmatter ของ agent/skills
7. parse JSON ทุกไฟล์
8. ตรวจ references และ required paths
9. รายงานสิ่งที่สร้างและคำสั่ง validation ที่รัน

ห้าม commit, push หรือสร้าง branch เว้นแต่ผู้ใช้สั่งโดยตรง
ห้ามแก้ไข source .NET หรือ template ต้นฉบับระหว่าง analysis
```

## ตัวอย่างการใช้งาน Prompt

เริ่ม project ใหม่โดยยังไม่เลือก route เพื่อให้ agent แสดง route catalog:

```text
/dotnet-to-go-converter
sourcePath: D:\DotnetRepo\OrdersApi
templatePath: D:\KKP_GO\ccps-mock\ccps-mock
specPaths:
  - D:\DotnetRepo\OrdersApi\openapi.json
outputPath: D:\work\orders-go
targetDatastore: {}
```

หลัง analysis agent ต้องแสดงรายการ route จาก spec เช่น:

```text
1. orders.get - GET /api/orders - discovered
2. orders.get-by-id - GET /api/orders/{id} - discovered
3. orders.create - POST /api/orders - discovered
```

จากนั้นเลือกทำทีละ route:

```text
เลือก route: orders.get-by-id
targetDatastore:
  orders.get-by-id: sql-server-existing
```

เมื่อ route แรกเสร็จ ให้สั่งต่อใน project เดิม:

```text
แสดง completed routes และ remaining routes จาก manifest
เลือกทำ route: orders.create
targetDatastore:
  orders.create: postgres
```

Agent ต้องโหลด manifest เดิมจาก `outputPath`, แสดงเฉพาะ route ที่ยังไม่ `validated`, และรักษา code/artifacts ของ `orders.get-by-id` ไว้

แทนค่าตัวแปรใน prompt เรียกใช้งานจริง:

```text
/dotnet-to-go-converter
sourcePath: D:\DotnetRepo\TestDotnet
templatePath: D:\KKP_GO\ccps-mock\ccps-mock
specPaths: []
allowInferredContract: true
outputPath: D:\work\test-agent-api3
routeSelection:
  - GetWeatherForecast
targetDatastore:
  GetWeatherForecast: sql-server-existing
```

## เกณฑ์ตรวจรับ project ที่สร้างเสร็จ

ตรวจให้ครบว่า:

- spec-first route selection ทำงานจริง
- inferred contract ต้อง opt-in
- full template baseline ถูก preserve
- `scripts/`, `sqlc.yaml`, `third_party/`, `go.mod`, `go.sum` และ KKP dependencies ยังอยู่
- prune เฉพาะ `cmd/`, `db/`, `docs/`, `internal`
- มี approval gates ครบสามระดับ
- manifest และ route artifacts validate ได้
- มี scope report
- มี blocker เมื่อ validation หรือ evidence ไม่พอ
- หลัง route แรกเสร็จ สามารถเลือก route ที่สองต่อใน output workspace เดิมได้
- route ที่เสร็จแล้วไม่ถูกนำกลับมาให้เลือกซ้ำโดย default
- project/module/service name ตรงกับชื่อ output folder
- server code ถูก generate จาก OpenAPI
- database generation ตรงกับ target datastore
- folder structure สอดคล้องกับ hexagonal architecture ของ template
- directory หลักของ template ยังอยู่ครบ และ prune เฉพาะไฟล์/โฟลเดอร์ย่อยที่อนุญาต
```
