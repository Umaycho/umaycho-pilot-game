# Umaycho Pilot Game

AI üretim ekibi akışını denemek için tek seviyeli 2.5D platformer prototipi. Godot 4 ile geliştirilir; işler GitHub issue'ları olarak açılır ve [`agentic-team`](https://github.com/Umaycho/umaycho-ai-team) plugin'iyle yerel Claude Code tarafından yürütülür.

## Başlarken
1. Godot 4.x (standart sürüm) kurulu olmalı.
2. Projeyi aç: Godot → Import → bu klasördeki `project.godot`.
3. Claude Code bu klasörde ilk açıldığında `ai-team` plugin'ini etkinleştirmeyi önerir; kabul et.

## AI ekiple çalışma
1. GitHub'da **Ekip görevi** şablonuyla bir issue aç ve `type:` etiketini ekle.
2. Bu klasörde `claude` çalıştır ve `issue #<no>'i yürüt` de (veya `/agentic-team:orkestra <no>`).
3. Ekip planı issue'ya yazar ve kartı **Onay bekliyor**'a alır. Sohbette "onaylandı" de.
4. Ekip işi bir dalda yapar, `develop`'a PR açar ve kartı **İncelemede**'ye taşır. PR'ı gözden geçirip birleştirmek sana kalır.

Ayrıntılar: [`CLAUDE.md`](CLAUDE.md)
