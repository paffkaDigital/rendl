# Rendl — AI kuchyňský asistent (Claude Code plugin)

## Shrnutí

Rendl je Claude Code plugin, který z Claude vytvoří komplexního kuchyňského asistenta. Umí importovat recepty (z webu i textu), spravovat sklad ingrediencí, plánovat týdenní jídelníčky s ohledem na nutriční cíle a pravidla, a generovat nákupní seznamy. Struktura pluginu kopíruje vzor pompo (zahradní asistent).

## Architektura

Monolitický plugin se sdílenými daty. Jeden plugin, 5 skills, data v `~/.config/rendl/`.

```
rendl/
├── .claude-plugin/
│   └── marketplace.json
├── .claude/
│   └── settings.local.json
├── plugins/rendl/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── hooks/
│   │   ├── hooks.json
│   │   └── context-session.md
│   ├── skills/
│   │   ├── setup/SKILL.md
│   │   ├── recept/SKILL.md
│   │   ├── sklad/SKILL.md
│   │   ├── jidelnicek/SKILL.md
│   │   └── nakup/SKILL.md
│   └── sablony/
│       ├── persona-default.md
│       ├── persona.md
│       ├── profil.md
│       ├── recept.md
│       ├── ingredience.md
│       ├── jidelnicek.md
│       └── nakup.md
└── docs/
    └── superpowers/specs/
        └── 2026-03-26-rendl-design.md
```

### Data: `~/.config/rendl/`

```
~/.config/rendl/
├── persona.md
├── profil.md
├── recepty/
├── sklad/
├── jidelnicky/
└── nakupy/
```

## Datový model

### `profil.md`

```yaml
---
pocet_osob: 4
uroven: "začátečník | pokročilý | zkušený"
vybaveni:
  - trouba
  - mixér
  - pomalý hrnec
diety:
  - bezlepková
alergie:
  - ořechy
pravidla:
  - "červené maso max 1×/týden"
  - "ryba min 2×/týden"
nutricni_cile:
  kcal_den: 2000
  bilkoviny_g: 80
  sacharidy_g: 250
  tuky_g: 65
---
```

### `recepty/<nazev>.md`

Názvy souborů: kebab-case bez diakritiky (jako pompo).

```yaml
---
nazev: "Kuřecí stir-fry"
zdroj: "https://..."
porce: 4
cas_pripravy: 15
cas_vareni: 20
kategorie: "oběd"
tagy:
  - kuřecí
  - asijská
makra_na_porci:
  kcal: 450
  bilkoviny_g: 35
  sacharidy_g: 40
  tuky_g: 15
---
# Kuřecí stir-fry

## Ingredience
- 2 kuřecí prsa (cca 400g)
- 1 hrnek rýže
- 2 lžíce sójové omáčky
- 1 lžička sezamového oleje
...

## Postup
1. ...
```

Konverze jednotek při importu — preferované jednoduché míry:
- 250 ml → 1 hrnek
- 125 ml → 1/2 hrnku
- 15 ml → 1 lžíce
- 5 ml → 1 lžička
- 1-2 ml → špetka
- 125 g mouky → 1 hrnek
- Přibližné konverze (237 ml → "cca 1 hrnek"), ne dogmatické

### `sklad/<ingredience>.md`

```yaml
---
nazev: "Rýže basmati"
kategorie: "přílohy"
mnozstvi: "hodně"
posledni_nakup: 2026-03-20
---
```

Množství: `hodně | dost | málo | dochází | nemám`

### `jidelnicky/<YYYY-Wxx>.md`

```yaml
---
tyden: "2026-W14"
typ_jidel:
  - snídaně
  - oběd
  - večeře
---
# Jídelníček — týden 14/2026

## Pondělí
### Snídaně
- [Ovesná kaše s ovocem](../recepty/ovesna-kase-s-ovocem.md)
### Oběd
- [Kuřecí stir-fry](../recepty/kureci-stir-fry.md)
...
```

### `nakupy/<YYYY-MM-DD>.md`

```yaml
---
datum: 2026-03-26
jidelnicek: "2026-W14"
stav: "aktivní | nakoupeno"
---
# Nákupní seznam — 26.3.2026

## Potřebuji
- [ ] Kuřecí prsa — cca 800g (2× recept)
- [ ] Sójová omáčka — 1 láhev (dochází)

## Mám na skladě
- Rýže basmati ✓
- Sezamový olej ✓
```

## Skills

### `rendl:setup` — Onboarding

**Allowed tools:** Read, Glob, Write, Edit

Postup:
1. Vytvoří adresářovou strukturu `~/.config/rendl/` a podadresáře (`recepty/`, `sklad/`, `jidelnicky/`, `nakupy/`)
2. Zeptá se na personu (nebo použije výchozí z `persona-default.md`)
3. Projde profil:
   - Počet osob v domácnosti
   - Úroveň kuchařských dovedností
   - Kuchyňské vybavení (trouba, mixér, pomalý hrnec, gril, sous-vide...)
   - Diety a alergie/intolerance
4. Zeptá se na frekvenční pravidla (nabídne rozumné výchozí: červené maso max 2×/týden, ryba min 1×/týden)
5. Zeptá se na nutriční cíle (nabídne výchozí dle počtu osob a standardních doporučení)
6. Uloží `profil.md`
7. Shrne co se nastavilo, navrhne další kroky (přidej první recept)

### `rendl:recept` — Import a správa receptů

**Allowed tools:** Read, Glob, Grep, Write, Edit, WebSearch, WebFetch

**Režimy:**

1. **Import z webu:**
   - Uživatel dá URL → WebFetch stáhne stránku
   - Claude extrahuje recept (ingredience, postup, porce, čas)
   - Převede jednotky na jednoduché míry
   - Dohledá makra pro ingredience přes Open Food Facts API (`https://world.openfoodfacts.org/cgi/search.pl?search_terms=<ingredience>&json=1`)
   - Pokud API nenajde → odhadne ze svých znalostí
   - Sečte makra na porci
   - Uloží do `recepty/<nazev>.md`

2. **Import z textu:**
   - Uživatel vloží text receptu
   - Stejné zpracování jako z webu

3. **Prohlížení:**
   - Seznam receptů s filtrováním dle tagů, kategorií, ingrediencí
   - Zobrazení detailu receptu včetně makro složení

4. **Úprava:**
   - Uživatel může ručně upravit cokoliv — makra, ingredience, postup
   - Rendl nabídne přepočet makro při změně ingrediencí

### `rendl:sklad` — Správa skladu ingrediencí

**Allowed tools:** Read, Glob, Grep, Write, Edit

**Režimy:**

1. **Přehled:**
   - Tabulka: Ingredience | Množství | Kategorie | Poslední nákup
   - Filtrování dle kategorie, množství ("co dochází?")

2. **Načtení účtenky:**
   - Uživatel vloží zkopírovaný text účtenky
   - Rendl identifikuje potravinové položky (ignoruje nepotravinové)
   - Přiřadí ke stávajícím ingrediencím v skladu nebo vytvoří nové
   - Aktualizuje množství (nákup → posun směrem k "hodně"/"dost")
   - Aktualizuje `posledni_nakup`

3. **Ruční úprava:**
   - Přidat/odebrat ingredienci
   - Změnit množství

4. **Po vaření:**
   - Když uživatel řekne "uvařil jsem X" → Rendl nabídne snížení skladu dle ingrediencí receptu
   - Množství snižuje heuristicky (ne přesně — odpovídá přibližnému měření)

### `rendl:jidelnicek` — Týdenní plánování

**Allowed tools:** Read, Glob, Grep, Write, Edit

**Postup:**

1. **Doptání při vytváření:**
   - Na kolik dní plánujeme?
   - Jaká jídla? (snídaně / svačina / oběd / večeře — výběr)
   - Speciální požadavky na tento týden? (host, oslava, méně času na vaření...)

2. **Načtení kontextu:**
   - Profil (pravidla, diety, alergie, nutriční cíle, vybavení, úroveň)
   - Dostupné recepty
   - Stav skladu (preferovat recepty z toho co mám)

3. **Generování návrhu — respektuje:**
   - Frekvenční pravidla (červené maso max X×/týden)
   - Vylučovací pravidla (alergie, diety)
   - Nutriční cíle (denní průměr makro přes celý týden)
   - Pestrost (neopakovat stejný recept, střídat kategorie)
   - Kuchyňské vybavení (nenavrhovat sous-vide, když nemám)
   - Úroveň dovedností (začátečníkům jednodušší recepty)
   - Stav skladu (preferovat co mám doma)

4. **Prezentace a úpravy:**
   - Ukáže návrh den po dni s makro souhrnem
   - Uživatel může vyměnit jednotlivá jídla ("v úterý nechci rybu")
   - Rendl přepočítá a nabídne alternativu

5. **Uložení:** `jidelnicky/<YYYY-Wxx>.md`

### `rendl:nakup` — Nákupní seznam

**Allowed tools:** Read, Glob, Grep, Write, Edit

**Postup:**

1. Načte aktuální/zvolený jídelníček
2. Projde všechny recepty v jídelníčku → extrahuje ingredience a množství
3. Sečte ingredience napříč recepty (2× kuřecí prsa → cca 800g)
4. Porovná se skladem:
   - `hodně/dost` → nepotřebuji
   - `málo/dochází` → potřebuji (méně)
   - `nemám` / neexistuje → potřebuji
5. Seskupí dle kategorií (maso, zelenina, mléčné, pečivo, suché, koření...)
6. Uloží: `nakupy/<YYYY-MM-DD>.md`
7. Po nákupu: uživatel označí co koupil → Rendl aktualizuje sklad

## Startup hook — Proaktivní chování

Při startu session (pokud existuje profil):

1. **Dnešní jídelníček** — pokud existuje jídelníček na aktuální týden, ukáže co je dnes na programu
2. **Aktivní nákupní seznam** — pokud existuje nenakoupený seznam, připomene
3. **Dochází ingredience** — pokud je něco `dochází` nebo `málo` s `posledni_nakup` starší než 30 dní, upozorní
4. **Doptání** — "Vařil jsi něco od minula?" → nabídne aktualizaci skladu

## Aktivace skills dle přirozeného jazyka

| Záměr | Skill | Příklady |
|-------|-------|----------|
| První kontakt, nastavení, profil | `rendl:setup` | "nastav", "změň diety", "kolik nás je", "přidej vybavení" |
| Import receptu, hledání receptu | `rendl:recept` | "přidej recept", "tady je URL", "najdi recept na X" |
| Sklad, účtenka, co mám doma | `rendl:sklad` | "co mám doma?", "nakoupil jsem", "tady je účtenka", "dochází mi mouka" |
| Plánování jídel, co vařit | `rendl:jidelnicek` | "naplánuj týden", "co vařit?", "jídelníček" |
| Nákupní seznam | `rendl:nakup` | "co nakoupit?", "nákupní seznam", "co chybí?" |

## Persona

- Výchozí persona v `persona-default.md` — výchozí osobnost kuchaře (přátelský, vtipný, motivující)
- Uživatel si může při setupu změnit
- Komunikace česky, v tónu persony

## Konvence názvů souborů

Stejné jako pompo — kebab-case bez diakritiky:
- Mezery → pomlčka
- Diakritika → základní znak (á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z)
- Vše malými písmeny
- Příklad: "Kuřecí stir-fry" → `kureci-stir-fry.md`

## Nutriční data — Open Food Facts API

Primární zdroj makro dat pro ingredience. Endpoint:
```
GET https://world.openfoodfacts.org/cgi/search.pl?search_terms={ingredience}&search_simple=1&action=process&json=1&page_size=5
```

Fallback: odhad ze znalostí Clauda (explicitně označený jako odhad v souboru receptu).

## Mimo scope (YAGNI)

- Fotografie jídel
- Sdílení receptů mezi uživateli
- Integrace s chytrými spotřebiči
- Propojení s rozvážkovými službami
- Počítání kalorií za den (pouze plánování, ne tracking)
- OCR účtenek z obrázků (jen text)
