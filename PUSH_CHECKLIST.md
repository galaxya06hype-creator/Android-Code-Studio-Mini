# PUSH CHECKLIST — lite/ram2gb-lite (Agent-5 Verifikasi)

> Status verifikasi: 2026-09-06, branch `lite/ram2gb-lite`. JANGAN push sebelum semua kotak hijau.
> Full APK build SENGAJA tidak dijalankan (RAM free ~187–543M, disk 8.6G — pasti OOM).

## 1. Branch & working tree
- [ ] `git branch` menunjukkan `* lite/ram2gb-lite` (dibuat Agent-1 dari `dev`)
- [ ] `git status --short` bersih kecuali file yang memang mau di-commit:
  - `M core/app/build.gradle.kts`
  - `?? LITE_BUILD.md`
  - `?? gradle.properties.lite`
  - `?? ideconfigurations/.../utils/LiteMode.kt`
- [ ] `git diff --check` exit 0 (tanpa whitespace error) — TERKONFIRMASI 2026-09-06
- [ ] `git diff --stat` direview (saat ini 51 tambah / 5 hapus di `core/app/build.gradle.kts`)

## 2. Commit per agent (koordinator yang commit, BUKAN agent)
Urutan anjuran (satu commit per agent agar revert mudah):
```bash
git add gradle.properties.lite core/app/build.gradle.kts LITE_BUILD.md
# pecah per agent bila file laporan masing-masing sudah ada:
git commit -m "lite(agent-1): branch lite/ram2gb-lite dari dev"
git commit -m "lite(agent-2): audit termux (LITE_TERMUX_AUDIT.md)"
git commit -m "lite(agent-3): flavor lite/full + LiteMode + fullImplementation (LITE_FEATURES.md)"
git commit -m "lite(agent-4): gradle.properties.lite + snippet build (LITE_BUILD.md)"
git commit -m "lite(agent-5): verifikasi ringan + PUSH_CHECKLIST.md"
```

## 3. Verifikasi ringan wajib sebelum push (sudah dilakukan sebagian)
- [x] AI dep ada: `implementation("com.google.ai.client.generativeai:generativeai:0.9.0")` (line ~267, tetap `implementation`, bukan full)
- [x] Kotlin/Java path ada: `javacServices`, `lspSetup`, `tooling.api`, `tooling.pluginConfig` (semua `implementation`)
- [x] `git diff --check` bersih
- [x] `du -sh` = 243M, `core/app` = 12M
- [ ] `./gradlew :core:app:tasks --offline --dry-run --max-workers=2` — BELUM HIJAU (lingkungan ini tanpa Java/SDK: `JAVA_HOME not set`). Wajib hijau di mesin builder (JDK 17 + SDK) sebelum push.
- [ ] `assembleLiteDebug` / `assembleDebug --max-workers=2 --offline` — BELUM dicoba (OOM risk). Wajib minimal `assembleDebug` hijau di builder.

## 4. Conflict / inkonsistensi — STATUS (update pasca 5-agent audit)
1. ~~`dimension = "mode"` vs `"tier"`~~ SELESAI: snippet + flavor nyata sama-sama `"tier"`.
2. ~~Snippet duplikat `create("lite")`~~ SELESAI: snippet hanya `getByName("lite")` berkomentar.
3. ~~`LITE_BUILD.md` klaim "flavor belum ada"~~ SELESAI: flavor `lite`/`full` commit.
4. ~~`fullImplementation` tanpa guard~~ SELESAI (Stage-1 safe): kembali `implementation`; flip = Stage-2.
5. ~~Pesan error "Only arm64..."~~ SELESAI: kini "Mini ini khusus arm64-v8a".
6. `gradle.properties.lite`: `nonTransitiveRClass=true` vs asli `false` — masih perlu uji varian `full` di CI.
7. ~~Laporan belum ada~~ SELESAI: keempat `LITE_*.md` + checklist ini ada dan sinkron.

## 5. Remote & PR (JANGAN dijalankan sebelum seksi 3–4 hijau)
Remote saat ini = upstream resmi (`AndroidCSOfficial/android-code-studio`), BUKAN fork. Jangan push langsung:
```bash
# 1. Fork di GitHub dulu, lalu:
git remote add fork <FORK_URL_PLACEHOLDER — mis. https://github.com/<USER>/android-code-studio.git>
git fetch fork
git push -u fork lite/ram2gb-lite

# 2. Buat PR ke upstream dev:
gh pr create --repo AndroidCSOfficial/android-code-studio \
  --base dev --head <USER>:lite/ram2gb-lite \
  --title "lite: varian RAM 2GB (flavor lite/full + build tuning)" \
  --body-file PUSH_CHECKLIST.md
```

## 6. GPL-3.0 notice (wajib di PR body / file baru)
- `LiteMode.kt` sudah memuat header GPL-3.0 — OK.
- `gradle.properties.lite`, `PUSH_CHECKLIST.md`, `LITE_*.md` adalah config/dok — tetap di bawah GPL-3.0 repo (`LICENSE` root). Jangan tambah lisensi inkompatibel.
- Blok `// LITE-*` di `build.gradle.kts` tidak mengubah lisensi.

## 7. Yang BELUM BOLEH di-push jika build belum hijau
- [ ] `core/app/build.gradle.kts` dengan `fullImplementation` tanpa guard call-site (risiko lite tidak kompilasi).
- [ ] Snippet `abiFilters arm64-v8a` aktif (belum ada di flavor nyata; potong ABI tanpa uji).
- [ ] `gradle.properties` hasil `cp gradle.properties.lite gradle.properties` (file `.lite` boleh di-push, overwrite `gradle.properties` JANGAN).
- [ ] File `.bak`, `local.properties`, signing key, `gradle.properties.bak`.
- [ ] Klaim hemat "%" di `LITE_BUILD.md` §5 sebagai hasil ukur (itu estimasi teoritis, bukan hasil build).

## 8. Perintah final koordinator (setelah semua hijau)
```bash
git status --short
git diff --check
git log --oneline -5
# baru kemudian commit per agent (lihat §2), push ke fork, gh pr create (lihat §5)
```

_Agent-5: hanya verifikasi + checklist. Tidak push, tidak commit final._
