# KODRA config

Public data files for the KODRA AI app builder: the model catalog, the MCP
server catalog, community templates, desktop defaults and the pnpm
`allowBuilds` list. KODRA reads them from
`https://raw.githubusercontent.com/ko-system/kodra-config/main/<path>` and
ships a copy of each file inside the app, so it keeps working offline or when
GitHub is unreachable.

The source of truth lives in the KODRABuilder repository under
`kodra-config/`. The public `ko-system/kodra-config` repository is a mirror
published by `scripts/publish-kodra-config.mjs`; do not edit it by hand.

[ภาษาไทยด้านล่าง](#ภาษาไทย)

## Files

| File                            | Consumer in KODRABuilder                                         | What it controls                                                                                                                                                                  |
| ------------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `v1/language-model-catalog.json` | `src/ipc/shared/remote_language_model_catalog.ts`                | Cloud providers and models in the model picker, model aliases (theme generator, auto mode, help bot), `codexClientVersion` for ChatGPT subscription requests. Cached for 1 hour. |
| `v1/mcp-catalog.json`           | `src/ipc/shared/remote_mcp_catalog.ts`                           | Curated MCP servers offered in the plugin catalog and by the agent's `suggest_plugin` tool (`featured: true`). Invalid entries are dropped one by one. Cached for 1 hour.        |
| `v1/templates.json`             | `src/ipc/utils/template_utils.ts`                                | Community templates on the Hub (`githubOrg`/`githubRepo` are cloned from GitHub). Cached for the app session.                                                                   |
| `v1/desktop-config.json`        | `src/ipc/shared/remote_desktop_config.ts`                        | Defaults for settings the user has not set, currently `defaults.blockUnsafeNpmPackages`. Cached for 5 minutes and persisted in userData.                                        |
| `v1/default-approve-builds.txt` | `src/ipc/utils/socket_firewall.ts`                               | npm packages whose install scripts pnpm may run in generated apps (`allowBuilds`). Must keep the `# dyad-default-allow-builds-*` header with `channel=remote`. Max 256 KB.        |

KODRA also sends a `HEAD` request for `v1/desktop-config.json` and reads the
HTTP `Date` header as a trusted clock for the free Basic Agent quota
(`src/ipc/handlers/free_agent_quota_handlers.ts`).

The app validates every file with the Zod schemas in the consumer modules.
`src/lib/kodra_config_bundle.test.ts` checks these files against the same
schemas, and the publish script checks the basic shape before pushing.

## Rules for editing

- `v1/` is the schema version. Additive, backwards compatible changes go into
  `v1/`. A breaking change needs a new `v2/` directory and an app release that
  reads it; older app versions keep reading `v1/`.
- Do not add `expiresAt` to these static files. The app treats a past
  `expiresAt` as "stale" and would refetch on every call. Bump `version`
  instead when you change a file.
- Keep `default-approve-builds.txt` sorted, one package per line, and bump its
  `dyad-default-allow-builds-data-version` when the list changes.
- Never put secrets in these files. The repository is public.

## Publishing

1. Edit the files under `kodra-config/` in KODRABuilder and merge to `main`.
   The app bundles these copies at build time.
2. Run the **Publish KODRA config** workflow (`workflow_dispatch`) in
   `ko-system/KODRABuilder`, or locally:

   ```sh
   node scripts/publish-kodra-config.mjs --dry-run
   node scripts/publish-kodra-config.mjs --remote-url https://github.com/ko-system/kodra-config.git
   ```

   The script mirrors `README.md`, `LICENSE` and `v1/`, commits
   `Sync config from KODRABuilder@<sha>`, pushes to `main`, and verifies every
   file at the pushed commit by sha256. The workflow authenticates with the
   `KODRA_CONFIG_DEPLOY_KEY` deploy key secret.

Running apps pick up the change within their cache TTL (at most 1 hour).

## Overrides

| Variable                          | When       | Effect                                                                                                                                                   |
| --------------------------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `VITE_KODRA_CONFIG_BASE_URL`      | build time | Base URL for all files (https only), for example a mirror.                                                                                              |
| `KODRA_DISABLE_REMOTE_CONFIG`     | runtime    | `true`/`1`: never fetch, use the bundled copies. `false`/`0`: fetch. Unset: follows `DYAD_DISABLE_EXTERNAL_SERVICES` (on by default with `DYAD_INTERNAL_PRO=true`). |
| `DYAD_LANGUAGE_MODEL_CATALOG_URL`, `DYAD_MCP_CATALOG_URL`, `DYAD_DESKTOP_CONFIG_URL`, `DYAD_DEFAULT_APPROVE_BUILDS_URL` | runtime | Per-file URL (tests, mirrors). Wins over everything above. |

## Provenance

The first version of these files (2026-11-13) is a snapshot of the public data
that upstream Dyad served from `https://api.dyad.sh/v1/*`, with these changes:

- `expiresAt` removed from every file (see the rules above).
- MCP catalog: `iconUrl` removed (it pointed at Dyad's API and KODRA does not
  render it), and the GitHub entry's vendored OAuth client (Dyad's own GitHub
  OAuth app) replaced with `oauthClientId` / `oauthClientSecret` inputs that
  the user fills in, like the Asana entry.
- Model catalog aliases keep their upstream IDs (`dyad/theme-generator/*`,
  `dyad/auto/*`, `dyad/help-bot/default`) because the app looks them up by
  those IDs.

## License

Apache-2.0. See `LICENSE`.

---

## ภาษาไทย

ไฟล์ข้อมูลสาธารณะของ KODRA AI app builder ได้แก่ รายการโมเดล รายการ MCP
server เทมเพลตจากชุมชน ค่าเริ่มต้นของแอป และรายการ `allowBuilds` ของ pnpm
แอป KODRA อ่านไฟล์เหล่านี้จาก
`https://raw.githubusercontent.com/ko-system/kodra-config/main/<path>` และมีสำเนา
ของทุกไฟล์ฝังมาในตัวแอปด้วย จึงยังใช้งานได้เมื่อออฟไลน์หรือเข้าถึง GitHub ไม่ได้

ต้นฉบับอยู่ใน repository KODRABuilder ที่โฟลเดอร์ `kodra-config/`
ส่วน repository สาธารณะ `ko-system/kodra-config` เป็นสำเนาที่ publish ด้วย
`scripts/publish-kodra-config.mjs` ห้ามแก้ไขใน repository นี้โดยตรง

### ไฟล์และผู้ใช้งาน

| ไฟล์                             | โมดูลที่อ่านไฟล์ใน KODRABuilder                       | ใช้ทำอะไร                                                                                                                                         |
| -------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `v1/language-model-catalog.json` | `src/ipc/shared/remote_language_model_catalog.ts`     | ผู้ให้บริการและโมเดลใน model picker, alias ของโมเดล (theme generator, auto mode, help bot) และ `codexClientVersion` สำหรับ ChatGPT subscription แคช 1 ชั่วโมง |
| `v1/mcp-catalog.json`            | `src/ipc/shared/remote_mcp_catalog.ts`                | MCP server ที่แนะนำในหน้า plugin catalog และในเครื่องมือ `suggest_plugin` ของ agent (`featured: true`) รายการที่ไม่ถูกต้องจะถูกตัดทิ้งทีละรายการ แคช 1 ชั่วโมง |
| `v1/templates.json`              | `src/ipc/utils/template_utils.ts`                     | เทมเพลตจากชุมชนในหน้า Hub (clone จาก GitHub ตาม `githubOrg`/`githubRepo`) แคชตลอดการเปิดแอป                                                   |
| `v1/desktop-config.json`         | `src/ipc/shared/remote_desktop_config.ts`             | ค่าเริ่มต้นของการตั้งค่าที่ผู้ใช้ยังไม่ได้ตั้งเอง ตอนนี้คือ `defaults.blockUnsafeNpmPackages` แคช 5 นาทีและบันทึกไว้ใน userData                     |
| `v1/default-approve-builds.txt`  | `src/ipc/utils/socket_firewall.ts`                    | แพ็กเกจ npm ที่ pnpm อนุญาตให้รัน install script ในแอปที่สร้าง (`allowBuilds`) ต้องมี header `# dyad-default-allow-builds-*` และ `channel=remote` ขนาดไม่เกิน 256 KB |

นอกจากนี้ KODRA ส่ง `HEAD` request ไปที่ `v1/desktop-config.json` เพื่ออ่าน
HTTP header `Date` เป็นนาฬิกาที่เชื่อถือได้สำหรับโควตา Basic Agent ฟรี
(`src/ipc/handlers/free_agent_quota_handlers.ts`)

แอปตรวจทุกไฟล์ด้วย Zod schema ในโมดูลที่อ่านไฟล์นั้น
`src/lib/kodra_config_bundle.test.ts` ตรวจไฟล์ในโฟลเดอร์นี้ด้วย schema เดียวกัน
และสคริปต์ publish ตรวจโครงสร้างพื้นฐานก่อน push

### กติกาการแก้ไข

- `v1/` คือเวอร์ชันของ schema การเปลี่ยนแปลงที่เข้ากันได้กับของเดิมให้ใส่ใน `v1/`
  ถ้าเปลี่ยนแบบไม่เข้ากัน ให้สร้างโฟลเดอร์ `v2/` และออกแอปเวอร์ชันใหม่ที่อ่าน `v2/`
  แอปเวอร์ชันเก่ายังอ่าน `v1/` ต่อไป
- ห้ามใส่ `expiresAt` ในไฟล์เหล่านี้ เพราะเมื่อเวลาผ่านไปแอปจะถือว่าข้อมูลหมดอายุ
  และดึงใหม่ทุกครั้ง ให้เปลี่ยน `version` แทนเมื่อแก้ไฟล์
- `default-approve-builds.txt` ต้องเรียงตามตัวอักษร หนึ่งแพ็กเกจต่อหนึ่งบรรทัด
  และต้องเพิ่ม `dyad-default-allow-builds-data-version` เมื่อแก้รายการ
- ห้ามใส่ข้อมูลลับใด ๆ เพราะ repository นี้เป็นสาธารณะ

### การ publish

1. แก้ไฟล์ใน `kodra-config/` ของ KODRABuilder แล้ว merge เข้า `main`
   (แอปจะฝังสำเนาเหล่านี้ตอน build)
2. รัน workflow **Publish KODRA config** (`workflow_dispatch`) ใน
   `ko-system/KODRABuilder` หรือรันบนเครื่อง:

   ```sh
   node scripts/publish-kodra-config.mjs --dry-run
   node scripts/publish-kodra-config.mjs --remote-url https://github.com/ko-system/kodra-config.git
   ```

   สคริปต์จะคัดลอก `README.md`, `LICENSE` และ `v1/` แล้ว commit ด้วยข้อความ
   `Sync config from KODRABuilder@<sha>` push ไปที่ `main` และตรวจ sha256
   ของทุกไฟล์ที่ commit นั้น workflow ใช้ deploy key จาก secret
   `KODRA_CONFIG_DEPLOY_KEY`

แอปที่เปิดอยู่จะเห็นข้อมูลใหม่เมื่อแคชหมดอายุ (ไม่เกิน 1 ชั่วโมง)

### ตัวแปรสำหรับปรับแต่ง

- `VITE_KODRA_CONFIG_BASE_URL` (ตอน build): base URL ของทุกไฟล์ (https เท่านั้น) เช่น mirror ภายในองค์กร
- `KODRA_DISABLE_REMOTE_CONFIG` (ตอนรัน): `true`/`1` ไม่ดึงจากเครือข่าย ใช้สำเนาที่ฝังในแอป,
  `false`/`0` ดึงจากเครือข่าย, ถ้าไม่ตั้งจะตามค่า `DYAD_DISABLE_EXTERNAL_SERVICES`
  (เปิดเป็นค่าเริ่มต้นเมื่อ `DYAD_INTERNAL_PRO=true`)
- `DYAD_LANGUAGE_MODEL_CATALOG_URL`, `DYAD_MCP_CATALOG_URL`, `DYAD_DESKTOP_CONFIG_URL`,
  `DYAD_DEFAULT_APPROVE_BUILDS_URL` (ตอนรัน): URL เฉพาะไฟล์ (ใช้ในการทดสอบหรือ mirror) มีผลเหนือค่าอื่นทั้งหมด

### ที่มาของข้อมูล

ไฟล์ชุดแรก (2026-11-13) คัดลอกจากข้อมูลสาธารณะที่ Dyad ต้นทางให้บริการที่
`https://api.dyad.sh/v1/*` โดยมีการเปลี่ยนแปลงดังนี้

- ลบ `expiresAt` ออกจากทุกไฟล์
- MCP catalog: ลบ `iconUrl` (ชี้ไปที่ API ของ Dyad และ KODRA ไม่ได้แสดงผล)
  และเปลี่ยน OAuth client ที่ฝังไว้ในรายการ GitHub (เป็น OAuth app ของ Dyad)
  ให้ผู้ใช้กรอก `oauthClientId` / `oauthClientSecret` เอง แบบเดียวกับรายการ Asana
- alias ของโมเดลยังใช้ ID เดิม (`dyad/theme-generator/*`, `dyad/auto/*`,
  `dyad/help-bot/default`) เพราะแอปค้นหาด้วย ID เหล่านี้

### สัญญาอนุญาต

Apache-2.0 ดูไฟล์ `LICENSE`
