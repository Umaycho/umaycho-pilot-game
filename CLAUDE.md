# Umaycho Pilot Game

Bu repo, `ai-team` marketplace'indeki **agentic-team** plugin'iyle çalışan bir projedir. Bu dosya ekibe projeye özgü bilgileri verir; ekibin rolleri ve çalışma yöntemi plugin'dedir.

## Proje
- **Ne:** AI ekip akışını denemek için tek seviyeli bir 2.5D platformer prototipi.
- **Tür:** Oyun (Godot 4, GDScript)
- **Dil:** Doküman ve yorumlar Türkçe, kod tanımlayıcıları İngilizce.

## Çalışma modu: LOKAL
- Ekip **geliştiricinin kendi bilgisayarındaki Claude Code** ile çalışır. Bulutta GitHub Actions ile çalışan bir AI workflow'u yoktur ve Claude API anahtarı kullanılmaz.
- GitHub'a erişim yerel `gh` CLI ile yapılır (`gh auth status` giriş yapılmış olmalı; board için `project` kapsamı gerekir).
- Görev başlatma: kullanıcı bu klasörde Claude Code'a "issue #N'i yürüt" der veya `/agentic-team:orkestra N` çağırır → **`orkestra` skill'ini izle**.
- **Onay:** Plan, issue'ya yorum olarak yazılır ve kullanıcıya sohbette de özetlenir. Kullanıcı sohbette "onaylandı" derse veya issue'ya `@claude onaylandı` yazarsa uygulamaya geç. Onay gelmeden kod yazma.
- `type:3d` işleri ve Blender/Hunyuan3D araçları henüz kurulu değil; bu tür işleri planla ama uygulama, "lokal 3D ortamı kurulmadı" diye raporla.

## Board (GitHub Projects)
- **owner:** Umaycho
- **Proje numarası:** 1
- **Statüler (sırayla):** Backlog, Planlanıyor, Onay bekliyor, Devam ediyor, İncelemede, Tamam
- **Döngü sınırı:** 3 (QA/inceleme turu; aşılırsa insana devredilir)
- Etiketler: `type:code`, `type:design`, `type:docs`, `type:3d`

## Dallar ve commit'ler
- Akış: `ai/<issue-no>-<kisa-ad>` dalı → **`develop`'a PR**. `develop` → `main` birleştirmesini (sürüm) yalnızca insan yapar.
- Commit: `<tür>(<kapsam>): <özet> (#<issue>)` — tür: feat, fix, refactor, test, docs, chore
- `main` korumalıdır. PR'ları yalnızca insanlar birleştirir.

## Godot
- **Sürüm:** Godot 4.x, standart (GDScript) sürüm. Proje kökü bu klasör (`project.godot`).
- **Çalıştırılabilir dosya (macOS):** `/Applications/Godot.app/Contents/MacOS/Godot` — kısaca `$GODOT`. Farklı bir yerdeyse kullanıcıya sor ve bu satırı güncelle.
- **Başsız doğrulama (her değişiklikten sonra):** `"$GODOT" --headless --path . --quit` → çıktıda `ERROR` veya `SCRIPT ERROR` olmamalı.
- **Editörü açma (insan için):** `"$GODOT" --path . -e`
- **Test:** Henüz test altyapısı yok. İlk `type:code` işinde GUT (Godot Unit Test) eklenmesi bir alt iş olarak planlanmalı; o zamana kadar QA başsız doğrulama + elle kontrol listesi kullanır.
- **Export:** Henüz yok.
- Godot'un ürettiği `.godot/` klasörü commit'lenmez. İlk açılışta Godot `project.godot`'u kendi sürümüne göre güncelleyebilir; bu değişiklik commit'lenebilir.

## Klasör yapısı
- `scenes/` — sahneler (`main.tscn` giriş sahnesi)
- `scripts/` — GDScript dosyaları
- `assets/models/`, `assets/textures/`, `assets/audio/` — Git LFS ile takip edilir
- `docs/adr/` — mimari kararlar
- `docs/design/GDD.md` — oyun tasarım dokümanı (henüz yok)
- `docs/CHANGELOG.md` — değişiklik günlüğü

## Projeye özgü kurallar
- Hedef platform: masaüstü (macOS/Windows). Kontroller: klavye + gamepad.
- Görsel stil: basit geometrik şekiller (yer tutucu), gerçek modeller lokal 3D aşamasında gelecek.
