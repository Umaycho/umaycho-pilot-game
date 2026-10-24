# Kutu Toplayıcı — Oyun Tasarım Dokümanı

> Amaç: AI ekip akışını denemek. Kapsam bilerek çok dar tutuldu. Burada yazmayan hiçbir şey kapsama dahil değil.
> Bu dokümandaki bütün sayılar ilk tahmin; **(A)** ile işaretli olanlar ayarlanabilir parametre (`@export` olarak açılmalı).

## 1. Özet
- **Tür / perspektif:** 2D platformer, yandan görünüm, saf 2D (Godot 4 `Node2D`). 2.5D değil.
- **Platform:** Masaüstü (macOS / Windows).
- **Çözünürlük:** Viewport 1280×720 px, sabit kamera, kaydırma yok. Önerilen ayar: `display/window/stretch/mode = canvas_items`, `aspect = keep`.
- **Hedef oturum süresi:** 30–60 saniye.
- **Tek cümlelik fikir:** Oyuncu tek ekranlık bir seviyede yürüyüp zıplayarak 3 sarı kutuyu toplar, sonra bitiş bayrağına dokunarak kazanır.
- **Koordinatlar:** Godot 2D kuralı, orijin sol üst köşe, +y aşağı. Tüm konumlar px.

## 2. Çekirdek döngü
1. Oyuncu platformlar arasında yürür ve zıplar.
2. Basamak gibi dizilmiş platformlara çıkıp kutuları toplar (sayaç `0/3` → `3/3`).
3. 3 kutu toplanınca bitiş bayrağı açılır (renk değişir).
4. Bayrağa dokununca kazanır. Çukura düşerse seviye baştan başlar.

## 3. Mekanikler (toplam 3)

### 3.1 Yürüme
- **Tanım:** Oyuncu yatay eksende sola/sağa hareket eder. Yerde ve havada kontrol aynıdır.
- **Oyuncu gövdesi:** `CharacterBody2D`, dikdörtgen çarpışma 32×48 px.
- **Parametreler (A):**

| Ad | Değer | Birim |
|---|---|---|
| `move_speed` | 240 | px/s |
| `acceleration` | 2000 | px/s² (0'dan tam hıza ~0,12 s) |
| `deceleration` | 2400 | px/s² (tam hızdan durmaya ~0,10 s) |
| `stick_deadzone` | 0,2 | eksen değeri (0–1) |

- **Kabul kriterleri:**
  - [ ] `move_right` basılı tutulunca yatay hız en fazla 240 px/s olur (±1 px/s).
  - [ ] Tuş bırakılınca oyuncu 0,15 s içinde durur.
  - [ ] Sola ve sağa aynı anda basılırsa yatay hız 0'a iner.
  - [ ] Ekranın sol (x = 0) ve sağ (x = 1280) kenarlarında görünmez duvarlar vardır; oyuncu ekrandan yatay olarak çıkamaz.
  - [ ] Gamepad sol çubuğu ile D-pad, klavyeyle aynı sonucu verir.

### 3.2 Zıplama
- **Tanım:** Oyuncu yalnızca yerdeyken (`is_on_floor()`) zıplar. Zıplama yüksekliği her zaman aynıdır. Değişken yükseklik, çift zıplama ve coyote time yok.
- **Parametreler (A):**

| Ad | Değer | Birim |
|---|---|---|
| `jump_velocity` | 600 | px/s (yukarı, kodda -600) |
| `gravity` | 1500 | px/s² |
| `max_fall_speed` | 900 | px/s |

- **Türetilen değerler (hesap):**
  - Zıplama yüksekliği: h = v² / (2g) = 600² / (2 × 1500) = **120 px**
  - Tepeye çıkış süresi: t = v / g = 600 / 1500 = **0,40 s**. Düz zeminde havada kalma süresi: **0,80 s**
  - Düz zeminde en uzun yatay atlayış: 240 px/s × 0,80 s = **192 px**
  - Tasarım kuralı: platformlar arası yükseklik farkı en fazla **96 px** (120 px'in %80'i, 24 px pay). Yatay boşluk en fazla **128 px**.
  - 96 px yukarıdaki platforma iniş: 96 = 600t − 750t² ⇒ inişte t ≈ 0,58 s ⇒ bu sürede 240 × 0,58 ≈ **139 px** yatay mesafe gider. 80 px boşluk rahatça geçilir.
- **Kabul kriterleri:**
  - [ ] Düz zeminde zıplanınca oyuncunun en yüksek noktası başlangıçtan 120 px (±4 px) yukarıdadır.
  - [ ] Havadayken `jump` tuşuna basmak hiçbir şey yapmaz.
  - [ ] Düşüş hızı 900 px/s'yi geçmez.
  - [ ] Seviyedeki her basamak platformuna (3.3 / bölüm 4) zıplayarak çıkılabilir.
  - [ ] Çukur (120 px) koşarak zıplanınca geçilebilir.

### 3.3 Toplama
- **Tanım:** Kutular `Area2D`'dir. Oyuncu kutuya değince kutu yok olur ve sayaç 1 artar. 3 kutunun hepsi toplanınca bitiş bayrağı açılır.
- **Parametreler (A):**

| Ad | Değer | Birim |
|---|---|---|
| `collectible_size` | 20×20 | px |
| `collectible_count` | 3 | adet (seviyedeki kutu sayısından otomatik sayılır) |
| `goal_locked_color` | `#808080` (gri) | renk |
| `goal_open_color` | `#30C050` (yeşil) | renk |

- **Kabul kriterleri:**
  - [ ] Kutuya değince kutu aynı karede kaybolur ve HUD sayacı 1 artar (`1/3`).
  - [ ] Aynı kutu iki kez sayılmaz.
  - [ ] 3/3 olunca bayrak gri renkten yeşile döner.
  - [ ] Bayrak griyken ona değmek hiçbir şey yapmaz.
  - [ ] Seviye yeniden başlayınca sayaç `0/3` olur ve 3 kutu geri gelir.

## 4. Seviye (tek seviye, tek ekran)

| Seviye | Amaç | Yeni öğe | Tahmini süre |
|---|---|---|---|
| 1 | 3 kutuyu topla, bayrağa dokun | Hepsi | 30–60 s |

### 4.1 Platformlar
Konum = sol üst köşe (x, y). Hepsi `StaticBody2D` + `ColorRect`/`Polygon2D`. Yüzen platformlar (P3–P5) **tek yönlü** çarpışır (`one_way_collision = true`): alttan geçilebilir, üstüne basılabilir.

| Ad | Konum (x, y) | Boyut (g×y) | Üst yüzey y | Bir öncekiyle fark | Not |
|---|---|---|---|---|---|
| P1 Zemin-sol | (0, 680) | 480×40 | 680 | — | Başlangıç burada |
| P2 Zemin-sağ | (600, 680) | 680×40 | 680 | 0 (120 px yatay çukur) | Bayrak burada |
| P3 Basamak-1 | (200, 584) | 160×20 | 584 | P1'den 96 px yukarı | Tek yönlü |
| P4 Basamak-2 | (440, 488) | 160×20 | 488 | P3'ten 96 px yukarı, 80 px yatay boşluk | Tek yönlü, çukurun üstünde |
| P5 Basamak-3 | (680, 392) | 160×20 | 392 | P4'ten 96 px yukarı, 80 px yatay boşluk | Tek yönlü |

- **Çukur:** x = 480–600 arası (120 px). Zemin yok.
- **Doğrulama:** Her yükseklik farkı 96 px ≤ 120 px; her yatay boşluk (80 / 120 px) ≤ 128 px. P3'ün altı ile zemin arasında 76 px boşluk var, 48 px boyundaki oyuncu altından geçebilir.

### 4.2 Nesneler
| Ad | Konum (merkez x, y) | Not |
|---|---|---|
| Başlangıç (oyuncu merkezi) | (80, 656) | Ayaklar y = 680'de |
| Kutu C1 | (280, 554) | P3'ün ortası, yüzeyin 30 px üstü |
| Kutu C2 | (520, 458) | P4'ün ortası |
| Kutu C3 | (760, 362) | P5'in ortası |
| Bitiş bayrağı | Merkez (1200, 640); sol üst (1180, 600), 40×80 | P2 üstünde, alt kenarı y = 680 zeminde, sağ kenarda. `Area2D` |
| HUD sayacı | (16, 16) | `Label`, metin `0/3` |

### 4.3 Kazanma / kaybetme
- **Kazanma:** 3/3 kutu toplanmışken oyuncu bayrağa değer. Ekranın ortasında `Label` "Kazandın!" görünür, oyuncu hareketi durur (girdi yok sayılır). `restart` ile seviye baştan başlar.
- **Kaybetme (ekrandan düşme):** Oyuncunun merkezi y > **820 px** olunca (ekranın 100 px altı) seviye hemen baştan yüklenir (`get_tree().reload_current_scene()`). Can, ceza ve "kaybettin" ekranı yok.
- **Elle yeniden başlatma:** `restart` her an seviyeyi baştan yükler.
- Kabul kriterleri:
  - [ ] Çukura düşünce 1 s içinde oyuncu başlangıç noktasında ve sayaç `0/3`.
  - [ ] 3/3 + bayrağa değince "Kazandın!" yazısı çıkar ve sağ/sol/zıplama tepki vermez.
  - [ ] `restart` her durumda seviyeyi başlangıç haline getirir.

## 5. Kontroller
Input Map'te (`project.godot`) tanımlanacak action'lar:

| Action | Eylem | Klavye | Gamepad |
|---|---|---|---|
| `move_left` | Sola yürü | A, Sol ok | Sol çubuk sol (eksen 0, −), D-pad sol |
| `move_right` | Sağa yürü | D, Sağ ok | Sol çubuk sağ (eksen 0, +), D-pad sağ |
| `jump` | Zıpla | Space, W, Yukarı ok | A / Cross (`JOY_BUTTON_A`, 0) |
| `restart` | Seviyeyi baştan başlat | R | Start / Options (`JOY_BUTTON_START`, 6) |

- Kabul kriteri: [ ] Her action hem klavyeyle hem gamepad ile tetiklenir.

## 6. Asset listesi (yer tutucu, hepsi kodla/sahnede çizilir)
Görsel dosya yok; hepsi `ColorRect` veya `Polygon2D`. Ses yok.

| Ad | Tür | Bütçe (px) | Stil notu | Öncelik |
|---|---|---|---|---|
| `player` | ColorRect | 32×48 | Mavi `#3070E0`, düz dikdörtgen | P0 |
| `platform_ground` | ColorRect | P1 480×40, P2 680×40 | Koyu kahve `#5A3E2B` | P0 |
| `platform_step` | ColorRect | 160×20 | Açık kahve `#A0703C` | P0 |
| `collectible` | Polygon2D (kare, 45° döndürülmüş = elmas) | 20×20 | Sarı `#F0C020` | P0 |
| `goal_flag` | ColorRect | 40×80 | Kapalı gri `#808080`, açık yeşil `#30C050` | P0 |
| `background` | ColorRect | 1280×720 | Açık gök mavisi `#BFE3F5` | P1 |
| `hud_counter` | Label | varsayılan font, 24 px | Siyah metin | P0 |
| `win_label` | Label | varsayılan font, 48 px | Siyah, ekran ortası | P0 |

## 7. Kapsam dışı / sonraki sürüm
- Ses ve müzik
- Ana menü, duraklatma menüsü, ayarlar
- Skor, süre, skor kaydı / kalıcı kayıt
- Birden fazla seviye, seviye geçişi
- Düşmanlar, tehlikeler (çukur hariç), can sistemi
- Kamera kaydırma, ekrandan büyük seviye
- Değişken zıplama yüksekliği, çift zıplama, coyote time, zıplama tamponu, duvar zıplama
- Animasyon, parçacık, ekran sarsıntısı
- Gerçek sprite/model (lokal 3D/sanat aşaması), 2.5D görünüm
- Export / dağıtım paketi
- Tuş yeniden atama, dokunmatik kontrol

## 8. Notlar (mimar / dokümantasyon için)
- `scenes/main.tscn` şu an `Node3D` iskeleti (Camera3D, DirectionalLight3D, CSGBox3D, Label3D). Saf 2D karar gereği `Node2D` köküne çevrilmeli; 3D düğümler kaldırılmalı.
- `CLAUDE.md`, `README.md` ve `project.godot` (`config/description`) projeyi "2.5D" diye tanımlıyordu; 2D kararıyla tutarlı olması için bu iş (#2) kapsamında güncellendi.

## 9. İlk `type:code` işleri
Her iş tek başına birleştirilebilir; her birinden sonra `"$GODOT" --headless --path . --quit` çıktısında `ERROR` / `SCRIPT ERROR` olmamalı.

1. **GUT test altyapısı kur**
   - GUT eklentisi `addons/gut/` altına eklenir, `tests/` klasörü ve örnek bir test (`tests/test_smoke.gd`) yazılır.
   - Kabul: GUT başsız komut satırından çalışır, örnek test geçer; komut `CLAUDE.md` "Test" bölümüne yazılmak üzere PR'da belirtilir.
2. **`main.tscn`'yi Node2D'ye çevir + Input Map**
   - Kök `Node2D`, 3D düğümler silinir; bölüm 5'teki 4 action `project.godot`'a eklenir; stretch ayarı (bölüm 1) uygulanır.
   - Kabul: Oyun açılınca 1280×720 boş 2D sahne görünür; başsız doğrulama temiz; bir GUT testi 4 action'ın `InputMap`'te var olduğunu doğrular.
3. **Seviye geometrisi**
   - Bölüm 4.1'deki 5 platform, ekran kenarı duvarları ve arka plan (bölüm 6 renkleri) `main.tscn`'ye eklenir.
   - Kabul: Platform konum/boyutları tabloyla birebir aynı; P3–P5 `one_way_collision = true`; GUT testi platform sayısını (5) ve konumlarını doğrular.
4. **Oyuncu: yürüme + zıplama**
   - `scripts/player.gd` (`CharacterBody2D`), bölüm 3.1 ve 3.2 parametreleri `@export` olarak.
   - Kabul: 3.1 ve 3.2 kabul kriterleri sağlanır; GUT testi `jump_velocity² / (2 × gravity) = 120` hesabını ve hız sınırını doğrular; elle kontrolde P3→P4→P5 basamakları ve çukur geçilir.
5. **Toplama, bitiş, düşme ve HUD**
   - Kutular (`scripts/collectible.gd`), bayrak (`scripts/goal.gd`), HUD sayacı, "Kazandın!" yazısı, y > 820 düşme kuralı ve `restart`.
   - Kabul: 3.3 ve 4.3 kabul kriterlerinin hepsi sağlanır; GUT testi sayaç artışını ve bayrağın 3/3'te açıldığını doğrular.

## Değişiklik günlüğü
| Tarih | Bölüm | Değişiklik | Issue |
|---|---|---|---|
| 2026-10-24 | Tümü | İlk sürüm: tek ekranlı 2D platformer, 3 mekanik (yürüme, zıplama, toplama) | #2 |
