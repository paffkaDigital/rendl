---
name: jidelnicek
description: Použij když uživatel chce naplánovat týdenní jídelníček, podívat se na aktuální plán, nebo upravit naplánovaná jídla.
allowed-tools: Read, Glob, Grep, Write, Edit
user-invocable: true
---

# Jídelníček — týdenní plánování

## Kdy se aktivuje

- "naplánuj týden", "jídelníček", "co vařit?"
- "ukaž jídelníček", "co je tento týden?"
- "změň úterý", "v pátek nechci rybu"

## Režimy

### 1. Nový jídelníček

Aktivace: "naplánuj týden", "jídelníček na příští týden"

Postup:

#### Fáze 1 — Doptání
Zeptej se postupně:
1. "Na který týden plánujeme?" (výchozí: příští týden)
2. "Kolik dní? Celý týden, nebo jen pracovní dny?"
3. "Jaká jídla chceš plánovat?"
   - a) Jen obědy
   - b) Obědy a večeře
   - c) Snídaně, obědy, večeře
   - d) Vše (snídaně, svačiny, obědy, svačiny, večeře)
   - e) Vlastní výběr
4. "Něco speciálního tento týden?" (hosté, méně času na vaření, oslava...)

#### Fáze 2 — Načtení kontextu
1. Přečti `$RENDL_HOME/profil.md` — pravidla, diety, alergie, nutriční cíle, vybavení, úroveň
2. Glob + přečti všechny recepty v `$RENDL_HOME/recepty/`
3. Glob + přečti sklad v `$RENDL_HOME/sklad/` (preferovat recepty z toho co mám)
4. Glob předchozí jídelníčky v `$RENDL_HOME/jidelnicky/` (pro pestrost — neopakovat minulý týden)

#### Fáze 3 — Generování návrhu
Navrhni jídelníček s ohledem na:

**Frekvenční pravidla:**
- Respektuj všechna pravidla z profilu (např. "červené maso max 1×/týden")
- Počítej výskyty dle tagů receptů

**Vylučovací pravidla:**
- Vyřaď recepty obsahující alergeny z profilu
- Respektuj diety (vegetariánská → žádné maso, bezlepková → žádný lepek v ingrediencích)

**Nutriční cíle:**
- Snaž se, aby denní součet makro hodnot odpovídal cílům z profilu (±15%)
- Zobraz denní i týdenní průměr

**Pestrost:**
- Neopakuj stejný recept v jednom týdnu
- Střídat kategorie (ne 3× těstoviny za sebou)
- Neopakuj recepty z předchozího jídelníčku

**Praktičnost:**
- Respektuj vybavení (nenavrhovat sous-vide když nemám)
- Respektuj úroveň dovedností
- Preferovat recepty ze skladu (méně nákupů)

#### Fáze 4 — Prezentace a úpravy
1. Zobraz návrh den po dni:

```
## Pondělí
### Oběd: Kuřecí stir-fry (450 kcal, 35g B, 40g S, 15g T)
### Večeře: Zeleninová polévka (280 kcal, 12g B, 35g S, 8g T)
**Denní souhrn: 730 kcal, 47g B, 75g S, 23g T**
```

2. Na konci zobraz nutriční souhrn za týden (průměr na den)
3. Zobraz dodržení pravidel: "✓ Červené maso: 1×/týden (max 2×)" atd.
4. Zeptej se: "Vyhovuje ti to? Můžeš vyměnit jakékoliv jídlo."

#### Fáze 5 — Úpravy
Pokud uživatel chce změnit:
- "V úterý nechci rybu" → navrhni alternativu se stejnou nutriční hodnotou
- "Přidej víc bílkovin" → uprav recepty směrem k vyššímu obsahu bílkovin
- "Zjednodušit středu" → navrhni rychlejší recept

#### Fáze 6 — Uložení
1. Přečti šablonu `${CLAUDE_PLUGIN_ROOT}/sablony/jidelnicek.md`
2. Vyplň finální data
3. Zapiš do `$RENDL_HOME/jidelnicky/{{YYYY-Wxx}}.md`
4. Nabídni vygenerování nákupního seznamu → `rendl:nakup`

### 2. Zobrazení jídelníčku

Aktivace: "ukaž jídelníček", "co je tento týden?", "co je dnes?"

Postup:
1. Zjisti aktuální týden (nebo požadovaný)
2. Najdi odpovídající soubor v `$RENDL_HOME/jidelnicky/`
3. Zobraz obsah
4. Pokud uživatel se ptá na konkrétní den ("co je dnes?") → zobraz jen ten den

### 3. Úprava existujícího jídelníčku

Aktivace: "změň úterý", "v pátek místo X dej Y"

Postup:
1. Najdi aktuální jídelníček
2. Přečti
3. Proveď požadovanou změnu (Edit)
4. Přepočti nutriční souhrn
5. Zobraz aktualizovaný den + nový souhrn
