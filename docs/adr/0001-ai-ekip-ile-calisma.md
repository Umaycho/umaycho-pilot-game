# 0001. AI ekip ile çalışma

- **Durum:** Kabul edildi
- **Tarih:** 2026-10-24
- **İlgili issue:** —

## Bağlam
Bu projede işler GitHub issue'ları olarak tanımlanıyor ve `agentic-team` plugin'indeki AI agent ekibi tarafından planlanıp uygulanıyor. İnsan; görev tanımlama, plan onayı ve PR birleştirme noktalarında devrede.

## Değerlendirilen seçenekler
1. **Tamamen manuel geliştirme** — artı: tam kontrol / eksi: yavaş.
2. **İnsan kapılı AI ekip** — artı: hız, tutarlı dokümantasyon / eksi: API maliyeti, gözden geçirme yükü.

## Karar
İnsan kapılı AI ekip, çünkü hız kazanırken plan onayı ve PR birleştirme insanın kontrolünde kalıyor.

## Sonuçlar
- Olumlu: Her değişiklik PR, test ve doküman güncellemesiyle geliyor.
- Olumsuz / bedel: API kullanımı izlenmeli; döngü sınırı 3.
- Ekip, geliştiricinin bilgisayarındaki Claude Code ile çalışır; bulutta AI workflow'u ve Claude API anahtarı yoktur. Board ve PR işlemleri yerel `gh` CLI ile yapılır.
- Takip gerektiren: Lokal 3D ortamı (Blender MCP, Hunyuan3D) sonraki aşamada eklenecek.
