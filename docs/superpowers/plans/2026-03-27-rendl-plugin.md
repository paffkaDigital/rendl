# Rendl — Claude Code Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a Claude Code plugin that turns Claude into a kitchen AI assistant — managing recipes, ingredient inventory, weekly meal plans, and shopping lists.

**Architecture:** Monolithic Claude Code plugin following the pompo pattern. One plugin with 5 skills, startup hook, and MD templates. User data persists in `~/.config/rendl/`.

**Tech Stack:** Claude Code plugin system (hooks.json, SKILL.md files), Markdown with YAML frontmatter, Open Food Facts API for nutrition data.

**Reference:** The pompo plugin at `../pompo/plugins/pompo/` serves as the structural template.

---

## File Structure

```
rendl/
├── .claude-plugin/
│   └── marketplace.json              # Marketplace config
├── .claude/
│   └── settings.local.json           # Plugin permissions
├── plugins/rendl/
│   ├── .claude-plugin/
│   │   └── plugin.json               # Plugin metadata
│   ├── hooks/
│   │   ├── hooks.json                # SessionStart hook config
│   │   └── context-session.md        # Startup context injected into session
│   ├── skills/
│   │   ├── setup/SKILL.md            # Onboarding skill
│   │   ├── recept/SKILL.md           # Recipe import/management
│   │   ├── sklad/SKILL.md            # Ingredient inventory
│   │   ├── jidelnicek/SKILL.md       # Weekly meal planning
│   │   └── nakup/SKILL.md            # Shopping list generation
│   └── sablony/
│       ├── persona-default.md        # Default persona
│       ├── persona.md                # Persona template
│       ├── profil.md                 # User profile template
│       ├── recept.md                 # Recipe template
│       ├── ingredience.md            # Inventory item template
│       ├── jidelnicek.md             # Meal plan template
│       └── nakup.md                  # Shopping list template
└── docs/
    └── superpowers/
        ├── specs/
        │   └── 2026-03-26-rendl-design.md
        └── plans/
            └── 2026-03-27-rendl-plugin.md
```

Runtime data (created by setup skill):
```
~/.config/rendl/
├── persona.md
├── profil.md
├── recepty/
├── sklad/
├── jidelnicky/
└── nakupy/
```

---

### Task 1: Plugin scaffold — marketplace, plugin.json, settings, .gitignore

**Files:**
- Create: `.claude-plugin/marketplace.json`
- Create: `.claude/settings.local.json`
- Create: `plugins/rendl/.claude-plugin/plugin.json`
- Create: `.gitignore`

- [ ] **Step 1: Create `.claude-plugin/marketplace.json`**

```json
{
  "name": "rendl",
  "owner": {
    "name": "paffkaDigital"
  },
  "plugins": [
    {
      "name": "rendl",
      "source": "./plugins/rendl",
      "description": "Kuchyňský asistent — recepty, sklad, jídelníčky a nákupní seznamy"
    }
  ]
}
```

- [ ] **Step 2: Create `plugins/rendl/.claude-plugin/plugin.json`**

```json
{
  "name": "rendl",
  "version": "0.1.0",
  "description": "Kuchyňský asistent — recepty, sklad, jídelníčky a nákupní seznamy",
  "license": "MIT",
  "author": {
    "name": "paffkaDigital"
  }
}
```

- [ ] **Step 3: Create `.claude/settings.local.json`**

```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Skill(rendl:setup)",
      "Skill(rendl:recept)",
      "Skill(rendl:sklad)",
      "Skill(rendl:jidelnicek)",
      "Skill(rendl:nakup)",
      "WebSearch",
      "WebFetch(domain:world.openfoodfacts.org)"
    ]
  }
}
```

- [ ] **Step 4: Create `.gitignore`**

```
.claude/settings.local.json
```

- [ ] **Step 5: Initialize git repo and commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git init
git add .claude-plugin/marketplace.json plugins/rendl/.claude-plugin/plugin.json .gitignore
git commit -m "chore: initialize rendl plugin scaffold"
```

---

### Task 2: Templates (sablony)

**Files:**
- Create: `plugins/rendl/sablony/persona-default.md`
- Create: `plugins/rendl/sablony/persona.md`
- Create: `plugins/rendl/sablony/profil.md`
- Create: `plugins/rendl/sablony/recept.md`
- Create: `plugins/rendl/sablony/ingredience.md`
- Create: `plugins/rendl/sablony/jidelnicek.md`
- Create: `plugins/rendl/sablony/nakup.md`

- [ ] **Step 1: Create `plugins/rendl/sablony/persona-default.md`**

```markdown
---
jmeno: "Rendl"
inspirace: "Kuchařský asistent s osobností"
---

# Rendl

## Osobnost
Přátelský kuchař, který miluje jednoduchost. Nesnáší zbytečné komplikace v kuchyni. Praktický, vtipný, občas trochu drzý. Ví, že nejlepší jídlo je to, které zvládneš uvařit bez stresu.

## Styl
- Tykání, neformální, přímý tón
- Občas použije kuchařské rčení nebo vtip
- Oslovuje "šéfkuchaři" nebo "šéfe"
- Stručný ale přátelský — žádné recepty na romány
```

- [ ] **Step 2: Create `plugins/rendl/sablony/persona.md`**

```markdown
---
jmeno: "{{jméno}}"
inspirace: "{{odkud postava pochází}}"
---

# {{jméno}}

## Osobnost
{{popis osobnosti, styl komunikace}}

## Styl
{{jak mluví — formálně/neformálně, humor, oslovení uživatele}}
```

- [ ] **Step 3: Create `plugins/rendl/sablony/profil.md`**

```markdown
---
pocet_osob: {{počet}}
uroven: "{{začátečník | pokročilý | zkušený}}"
vybaveni:
  - {{položka}}
diety:
  - {{dieta nebo prázdné}}
alergie:
  - {{alergie nebo prázdné}}
pravidla:
  - "{{pravidlo, např. červené maso max 1×/týden}}"
nutricni_cile:
  kcal_den: {{číslo}}
  bilkoviny_g: {{číslo}}
  sacharidy_g: {{číslo}}
  tuky_g: {{číslo}}
---

# Profil domácnosti

- **Počet osob:** {{počet}}
- **Úroveň vaření:** {{úroveň}}
- **Vybavení:** {{seznam}}
- **Diety:** {{seznam nebo žádné}}
- **Alergie:** {{seznam nebo žádné}}

## Pravidla jídelníčku
{{seznam pravidel}}

## Nutriční cíle (na osobu/den)
- **Energie:** {{kcal}} kcal
- **Bílkoviny:** {{g}} g
- **Sacharidy:** {{g}} g
- **Tuky:** {{g}} g
```

- [ ] **Step 4: Create `plugins/rendl/sablony/recept.md`**

```markdown
---
nazev: "{{název}}"
zdroj: "{{url nebo 'ruční zadání'}}"
porce: {{počet}}
cas_pripravy: {{minuty}}
cas_vareni: {{minuty}}
kategorie: "{{snídaně | oběd | večeře | svačina | dezert | příloha}}"
tagy:
  - {{tag}}
makra_na_porci:
  kcal: {{číslo}}
  bilkoviny_g: {{číslo}}
  sacharidy_g: {{číslo}}
  tuky_g: {{číslo}}
---

# {{název}}

## Ingredience
- {{množství}} {{ingredience}}

## Postup
1. {{krok}}

## Poznámky
{{volitelné — tipy, variace, skladování}}
```

- [ ] **Step 5: Create `plugins/rendl/sablony/ingredience.md`**

```markdown
---
nazev: "{{název}}"
kategorie: "{{maso | zelenina | ovoce | mléčné | pečivo | suché | koření | oleje | nápoje | ostatní}}"
mnozstvi: "{{hodně | dost | málo | dochází | nemám}}"
posledni_nakup: {{YYYY-MM-DD}}
---

# {{název}}

- **Kategorie:** {{kategorie}}
- **Množství:** {{množství}}
- **Poslední nákup:** {{datum}}
```

- [ ] **Step 6: Create `plugins/rendl/sablony/jidelnicek.md`**

```markdown
---
tyden: "{{YYYY-Wxx}}"
typ_jidel:
  - {{typ}}
---

# Jídelníček — týden {{číslo}}/{{rok}}

## Pondělí
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Úterý
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Středa
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Čtvrtek
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Pátek
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Sobota
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Neděle
### {{typ jídla}}
- [{{název receptu}}](../recepty/{{soubor}}.md)

## Nutriční souhrn
| Den | kcal | Bílkoviny | Sacharidy | Tuky |
|-----|------|-----------|-----------|------|
| Po  | {{}} | {{}} g    | {{}} g    | {{}} g |
```

- [ ] **Step 7: Create `plugins/rendl/sablony/nakup.md`**

```markdown
---
datum: {{YYYY-MM-DD}}
jidelnicek: "{{YYYY-Wxx}}"
stav: "{{aktivní | nakoupeno}}"
---

# Nákupní seznam — {{datum}}

## Potřebuji
{{seskupeno dle kategorií}}

### Maso
- [ ] {{položka}} — {{množství}} ({{důvod}})

### Zelenina
- [ ] {{položka}} — {{množství}}

### Mléčné
- [ ] {{položka}} — {{množství}}

### Suché / trvanlivé
- [ ] {{položka}} — {{množství}}

### Koření
- [ ] {{položka}} — {{množství}}

### Ostatní
- [ ] {{položka}} — {{množství}}

## Mám na skladě
- {{položka}} ✓
```

- [ ] **Step 8: Commit templates**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/sablony/
git commit -m "feat: add all data templates (sablony)"
```

---

### Task 3: Startup hook — hooks.json + context-session.md

**Files:**
- Create: `plugins/rendl/hooks/hooks.json`
- Create: `plugins/rendl/hooks/context-session.md`

- [ ] **Step 1: Create `plugins/rendl/hooks/hooks.json`**

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "cat ${CLAUDE_PLUGIN_ROOT}/hooks/context-session.md"
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Create `plugins/rendl/hooks/context-session.md`**

```markdown
# Rendl — kuchyňský asistent

## Cesty

- **RENDL_HOME:** `~/.config/rendl` — uživatelská data (přežijí aktualizace pluginu)
- **Plugin root:** `${CLAUDE_PLUGIN_ROOT}` — kód pluginu (šablony, skills, hooky)

## Persona

Přečti `$RENDL_HOME/persona.md` pro jméno a styl komunikace. Pokud neexistuje, použij `${CLAUDE_PLUGIN_ROOT}/sablony/persona-default.md`. Komunikuj česky, v tónu a stylu dané persony.

Pokud neexistuje ani persona ani profil → spusť onboarding (skill `rendl:setup`).

## Datové soubory

Všechna data jsou v `$RENDL_HOME/`:
- `persona.md` — persona asistenta (jméno, osobnost, styl komunikace)
- `profil.md` — profil domácnosti (počet osob, diety, alergie, vybavení, nutriční cíle)
- `recepty/` — .md soubory receptů
- `sklad/` — .md soubory ingrediencí na skladě
- `jidelnicky/` — .md soubory týdenních jídelníčků
- `nakupy/` — .md soubory nákupních seznamů

Šablony pro vytváření nových souborů jsou v `${CLAUDE_PLUGIN_ROOT}/sablony/`.

## Konvence názvů souborů

Názvy souborů používají kebab-case bez diakritiky:
- Mezery → pomlčka
- Diakritika → základní znak (á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z)
- Vše malými písmeny
- Příklad: "Kuřecí stir-fry" → `kureci-stir-fry.md`

## Při startu session

Pokud existuje profil a onboarding je dokončen:

### 1. Dnešní jídelníček
Zjisti aktuální týden (YYYY-Wxx). Zkontroluj `$RENDL_HOME/jidelnicky/` — existuje jídelníček na tento týden?
- Pokud ano → zobraz co je dnes na programu (v tónu persony)

### 2. Aktivní nákupní seznam
Zkontroluj `$RENDL_HOME/nakupy/` — existuje seznam se stavem `aktivní`?
- Pokud ano → připomeň: "Máš nenakoupený seznam z {{datum}}."

### 3. Dochází ingredience
Projdi `$RENDL_HOME/sklad/` — jsou ingredience s množstvím `dochází` nebo `málo` a `posledni_nakup` starší než 30 dní?
- Pokud ano → upozorni v tónu persony

### 4. Doptání na aktivitu
Zeptej se v tónu persony: "Vařil jsi něco od minula?"
- Pokud uživatel odpoví → nabídni aktualizaci skladu

## Aktivace skills podle přirozeného jazyka

| Záměr | Skill | Příklady |
|-------|-------|----------|
| První kontakt, nastavení, profil | `rendl:setup` | "nastav", "změň diety", "kolik nás je", "přidej vybavení" |
| Import receptu, hledání receptu | `rendl:recept` | "přidej recept", "tady je URL", "najdi recept na X" |
| Sklad, účtenka, co mám doma | `rendl:sklad` | "co mám doma?", "nakoupil jsem", "tady je účtenka", "dochází mi mouka" |
| Plánování jídel, co vařit | `rendl:jidelnicek` | "naplánuj týden", "co vařit?", "jídelníček" |
| Nákupní seznam | `rendl:nakup` | "co nakoupit?", "nákupní seznam", "co chybí?" |

## Pravidla

- Před vytvořením nového .md souboru vždy zkontroluj (Glob), zda už neexistuje.
- Pokud chybí `$RENDL_HOME/profil.md`, upozorni uživatele, že je potřeba nejdřív spustit nastavení.
- Při konverzi jednotek preferuj jednoduché míry: hrnek, lžíce, lžička, špetka, hrst.
- Nutriční data hledej přes Open Food Facts API, fallback na vlastní znalosti.
```

- [ ] **Step 3: Commit hooks**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/hooks/
git commit -m "feat: add startup hook with session context"
```

---

### Task 4: Skill — rendl:setup (onboarding)

**Files:**
- Create: `plugins/rendl/skills/setup/SKILL.md`

- [ ] **Step 1: Create `plugins/rendl/skills/setup/SKILL.md`**

```markdown
---
name: setup
description: Použij při prvním kontaktu s uživatelem (onboarding), když chybí profil, nebo když uživatel chce změnit nastavení (persona, diety, vybavení, pravidla).
allowed-tools: Read, Glob, Write, Edit
user-invocable: true
---

# Setup — onboarding a nastavení

## Kdy se aktivuje

- První spuštění (chybí `$RENDL_HOME/profil.md`)
- "nastav", "změň diety", "přidej vybavení", "kolik nás je"
- "změň personu", "kdo jsi?"

## Postup — první onboarding

### 1. Inicializace adresářů

Vytvoř adresářovou strukturu (pokud neexistuje):
```bash
mkdir -p ~/.config/rendl/{recepty,sklad,jidelnicky,nakupy}
```

### 2. Persona

- Zkontroluj, zda existuje `$RENDL_HOME/persona.md`
- Pokud ne → zkopíruj `${CLAUDE_PLUGIN_ROOT}/sablony/persona-default.md` do `$RENDL_HOME/persona.md`
- Zeptej se: "Jsem Rendl, tvůj kuchyňský asistent. Chceš si mě přizpůsobit, nebo ti vyhovuju takhle?"
- Pokud uživatel chce změnu → zeptej se na jméno a styl, vytvoř novou personu dle šablony `${CLAUDE_PLUGIN_ROOT}/sablony/persona.md`

### 3. Profil domácnosti

Ptej se postupně (jedna otázka po druhé):

1. **Počet osob:** "Pro kolik lidí obvykle vaříš?"
2. **Úroveň vaření:** "Jak se hodnotíš v kuchyni?"
   - a) Začátečník — zvládnu základy
   - b) Pokročilý — vařím pravidelně, nebojím se experimentovat
   - c) Zkušený — kuchyně je moje hřiště
3. **Vybavení:** "Co máš v kuchyni? Vyber co máš:"
   - trouba, sporák, mikrovlnka, mixér, tyčový mixér, robot, pomalý hrnec, tlakový hrnec, gril, sous-vide, fritéza, horkovzdušná fritéza, vaflovač, sendvičovač
   - (volný vstup — uživatel může dopsat vlastní)
4. **Diety:** "Držíš nějakou dietu nebo stravovací styl?"
   - bezlepková, bezlaktózová, vegetariánská, veganská, low-carb, keto, žádná
   - (volný vstup)
5. **Alergie:** "Máš potravinové alergie nebo intolerance?"
   - ořechy, lepek, laktóza, vejce, ryby, sója, žádné
   - (volný vstup)
6. **Pravidla:** "Chceš nastavit pravidla pro jídelníček? Nabízím rozumné výchozí:"
   - červené maso max 2×/týden
   - ryba min 1×/týden
   - luštěniny min 2×/týden
   - sladké max 2×/týden
   - Uživatel může upravit, přidat, odebrat
7. **Nutriční cíle:** "Chceš sledovat nutriční hodnoty? Nabízím výchozí pro {{počet_osob}} osoby:"
   - Nabídni výchozí hodnoty odvozené z běžných doporučení (2000 kcal, 50-80g bílkovin, 250-300g sacharidů, 60-80g tuků na osobu)
   - Uživatel může upravit nebo přeskočit

### 4. Uložení profilu

1. Přečti šablonu `${CLAUDE_PLUGIN_ROOT}/sablony/profil.md`
2. Vyplň nasbíraná data
3. Zapiš do `$RENDL_HOME/profil.md`

### 5. Shrnutí

Zobraz co se nastavilo a navrhni další kroky:
- "Přidej první recept" → `rendl:recept`
- "Načti účtenku z nákupu" → `rendl:sklad`

## Změna persony

Aktivace: "změň personu", "kdo jsi?", "chci jiného asistenta"

1. Zeptej se na nové jméno a styl
2. Vytvoř novou personu dle šablony
3. Zapiš do `$RENDL_HOME/persona.md` (přepíše existující)

## Úprava profilu

Aktivace: "změň diety", "přidej vybavení", "kolik nás je", "změň pravidla"

1. Přečti existující `$RENDL_HOME/profil.md`
2. Uprav požadované pole
3. Zapiš zpět

## Mapování diakritiky pro názvy souborů

á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z
```

- [ ] **Step 2: Commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/skills/setup/
git commit -m "feat: add rendl:setup skill (onboarding)"
```

---

### Task 5: Skill — rendl:recept (import a správa receptů)

**Files:**
- Create: `plugins/rendl/skills/recept/SKILL.md`

- [ ] **Step 1: Create `plugins/rendl/skills/recept/SKILL.md`**

```markdown
---
name: recept
description: Použij když uživatel chce přidat recept (z URL nebo textu), prohlížet recepty, nebo upravit existující recept.
allowed-tools: Read, Glob, Grep, Write, Edit, WebSearch, WebFetch
user-invocable: true
---

# Recept — import a správa receptů

## Kdy se aktivuje

- "přidej recept", "tady je recept", "ulož recept"
- "tady je URL: https://..."
- "najdi recept na kuřecí"
- "ukaž recepty", "co umím vařit?"
- "uprav recept X"

## Režimy

### 1. Import z webu

Aktivace: uživatel vloží URL nebo řekne "najdi recept na X"

Postup:
1. Pokud URL → WebFetch stáhne stránku
2. Pokud text "najdi recept" → WebSearch, pak WebFetch nejlepšího výsledku
3. Extrahuj z obsahu stránky:
   - Název receptu
   - Seznam ingrediencí s množstvím
   - Postup přípravy
   - Počet porcí
   - Čas přípravy a vaření
4. **Konverze jednotek** na jednoduché míry:
   - 250 ml → 1 hrnek
   - 125 ml → 1/2 hrnku
   - 60 ml → 1/4 hrnku
   - 15 ml → 1 lžíce
   - 5 ml → 1 lžička
   - 1-2 ml → špetka
   - 125 g mouky → cca 1 hrnek
   - 200 g cukru → cca 1 hrnek
   - 250 g másla → cca 1 hrnek
   - Přibližné konverze označit "cca" (např. "cca 1 hrnek")
   - Pokud je přesné množství důležité (pečení, droždí) → ponechat původní + uvést jednoduchý ekvivalent v závorce
5. **Nutriční data** pro každou ingredienci:
   - WebFetch na Open Food Facts API: `https://world.openfoodfacts.org/cgi/search.pl?search_terms={{ingredience}}&search_simple=1&action=process&json=1&page_size=3&fields=product_name,nutriments`
   - Z odpovědi extrahuj na 100g: kcal, bílkoviny, sacharidy, tuky
   - Přepočti na množství v receptu
   - Pokud API nenajde → odhadni ze svých znalostí, označ `(odhad)` v poznámce
   - Sečti makra na porci → ulož do frontmatter `makra_na_porci`
6. Zeptej se na kategorii (snídaně/oběd/večeře/svačina/dezert) pokud není zřejmá
7. Navrhni tagy (kuřecí, asijská, rychlé, vegetariánské...)
8. Vygeneruj kebab-case název souboru bez diakritiky
9. Zkontroluj (Glob) zda `$RENDL_HOME/recepty/{{filename}}.md` už neexistuje
10. Přečti šablonu `${CLAUDE_PLUGIN_ROOT}/sablony/recept.md`, vyplň, zapiš
11. Zobraz uživateli shrnutí: název, ingredience, makra na porci

### 2. Import z textu

Aktivace: uživatel vloží text receptu

Postup: stejný jako import z webu od bodu 3 (extrakce) — vstupem je text místo HTML.

### 3. Prohlížení receptů

Aktivace: "ukaž recepty", "co umím vařit?", "recepty s kuřecím"

Postup:
1. Glob všechny soubory v `$RENDL_HOME/recepty/`
2. Přečti frontmatter každého souboru
3. Pokud uživatel filtruje (tag, kategorie, ingredience) → Grep pro odpovídající
4. Zobraz tabulku:

| Recept | Kategorie | Čas | kcal/porce | Tagy |
|--------|-----------|-----|------------|------|

### 4. Úprava receptu

Aktivace: "uprav recept X", "změň ingredience v Y"

Postup:
1. Najdi recept (Glob + Grep)
2. Přečti soubor
3. Uživatel řekne co změnit
4. Uprav (Edit)
5. Pokud se změnily ingredience → nabídni přepočet makro hodnot

## Mapování diakritiky pro názvy souborů

á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z
```

- [ ] **Step 2: Commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/skills/recept/
git commit -m "feat: add rendl:recept skill (recipe import and management)"
```

---

### Task 6: Skill — rendl:sklad (správa skladu ingrediencí)

**Files:**
- Create: `plugins/rendl/skills/sklad/SKILL.md`

- [ ] **Step 1: Create `plugins/rendl/skills/sklad/SKILL.md`**

```markdown
---
name: sklad
description: Použij když uživatel chce zobrazit sklad ingrediencí, načíst účtenku, přidat/odebrat položky, nebo aktualizovat po vaření.
allowed-tools: Read, Glob, Grep, Write, Edit
user-invocable: true
---

# Sklad — správa ingrediencí

## Kdy se aktivuje

- "co mám doma?", "sklad", "ingredience"
- "nakoupil jsem", "tady je účtenka"
- "dochází mi X", "přidej X do skladu"
- "uvařil jsem X" (nabídka snížení skladu)

## Režimy

### 1. Přehled skladu

Aktivace: "co mám doma?", "sklad", "co mám na skladě?"

Postup:
1. Glob všechny soubory v `$RENDL_HOME/sklad/`
2. Přečti frontmatter každého souboru
3. Zobraz tabulku seskupenou dle kategorií:

**Maso:**
| Ingredience | Množství | Poslední nákup |
|-------------|----------|----------------|

**Zelenina:**
| Ingredience | Množství | Poslední nákup |
|-------------|----------|----------------|

...atd.

4. Zvýrazni položky s množstvím `dochází` nebo `málo`
5. Pokud sklad prázdný → "Sklad je prázdný. Načti účtenku z nákupu nebo přidej ingredience ručně."

Pokud uživatel filtruje ("co dochází?", "co mám z masa?") → filtruj odpovídajícím způsobem.

### 2. Načtení účtenky

Aktivace: "nakoupil jsem", "tady je účtenka", uživatel vloží text účtenky

Postup:
1. Uživatel vloží zkopírovaný text účtenky
2. Parsuj text — identifikuj potravinové položky:
   - Ignoruj nepotravinové položky (igelitky, DPH řádky, celkové součty, čísla účtenky)
   - Rozpoznej názvy potravin i z zkrácených forem na účtenkách (např. "KUR PRSA" → "Kuřecí prsa")
3. Pro každou identifikovanou položku:
   - Normalizuj název
   - Přiřaď kategorii (maso, zelenina, mléčné, suché, koření, oleje, nápoje, ostatní)
   - Zkontroluj (Glob) zda existuje v `$RENDL_HOME/sklad/`
   - Pokud existuje → aktualizuj množství směrem nahoru (málo→dost, dochází→dost), aktualizuj `posledni_nakup`
   - Pokud neexistuje → vytvoř nový soubor dle šablony `${CLAUDE_PLUGIN_ROOT}/sablony/ingredience.md` s množstvím `dost` a dnešním datem
4. Zobraz souhrn: co bylo aktualizováno, co bylo přidáno nového
5. Zeptej se, zda je vše správně — uživatel může opravit

### 3. Ruční úprava

Aktivace: "přidej X do skladu", "dochází mi X", "nemám X"

Postup:
1. Identifikuj ingredienci a požadovanou akci
2. Vygeneruj kebab-case název souboru
3. Pokud existuje → Edit frontmatter (množství, datum)
4. Pokud neexistuje → vytvoř nový soubor dle šablony
5. Potvrď změnu

### 4. Po vaření

Aktivace: "uvařil jsem X", "dnes jsem dělal Y"

Postup:
1. Najdi recept v `$RENDL_HOME/recepty/` (Glob + Grep)
2. Pokud nalezen → přečti ingredience receptu
3. Pro každou ingredienci:
   - Najdi v skladu (Glob)
   - Navrhni snížení množství (heuristicky: pokud bylo `hodně`→`dost`, `dost`→`málo`, `málo`→`dochází`)
4. Zobraz návrh změn a zeptej se zda souhlasí
5. Pokud uživatel souhlasí → aktualizuj sklad

## Mapování diakritiky pro názvy souborů

á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z
```

- [ ] **Step 2: Commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/skills/sklad/
git commit -m "feat: add rendl:sklad skill (ingredient inventory)"
```

---

### Task 7: Skill — rendl:jidelnicek (týdenní plánování)

**Files:**
- Create: `plugins/rendl/skills/jidelnicek/SKILL.md`

- [ ] **Step 1: Create `plugins/rendl/skills/jidelnicek/SKILL.md`**

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/skills/jidelnicek/
git commit -m "feat: add rendl:jidelnicek skill (weekly meal planning)"
```

---

### Task 8: Skill — rendl:nakup (nákupní seznam)

**Files:**
- Create: `plugins/rendl/skills/nakup/SKILL.md`

- [ ] **Step 1: Create `plugins/rendl/skills/nakup/SKILL.md`**

```markdown
---
name: nakup
description: Použij když uživatel chce vygenerovat nákupní seznam z jídelníčku, zobrazit existující seznam, nebo označit nákup jako dokončený.
allowed-tools: Read, Glob, Grep, Write, Edit
user-invocable: true
---

# Nákup — nákupní seznam

## Kdy se aktivuje

- "co nakoupit?", "nákupní seznam", "co chybí?"
- "nakoupeno", "mám nakoupeno"
- "ukaž nákupní seznam"

## Režimy

### 1. Generování nákupního seznamu

Aktivace: "co nakoupit?", "nákupní seznam", "vygeneruj nákup"

Postup:

1. **Najdi jídelníček:**
   - Pokud uživatel specifikuje týden → použij ten
   - Pokud ne → hledej nejnovější jídelníček v `$RENDL_HOME/jidelnicky/`
   - Pokud žádný neexistuje → "Nemáš naplánovaný jídelníček. Chceš nejdřív naplánovat týden?"

2. **Extrahuj ingredience:**
   - Přečti jídelníček → identifikuj všechny recepty (linky na soubory)
   - Pro každý recept přečti soubor v `$RENDL_HOME/recepty/`
   - Extrahuj všechny ingredience a množství ze sekce "Ingredience"

3. **Agreguj napříč recepty:**
   - Sečti stejné ingredience (2× kuřecí prsa v různých receptech → celkový součet)
   - Zachovej jednoduché míry (2 lžíce + 1 lžíce = 3 lžíce, ne 45 ml)

4. **Porovnej se skladem:**
   - Pro každou ingredienci zkontroluj `$RENDL_HOME/sklad/` (Glob)
   - Rozděl na:
     - **Potřebuji:** ingredience kde sklad je `nemám`, `dochází`, nebo neexistuje
     - **Málo, možná dokup:** ingredience kde sklad je `málo` a recept potřebuje větší množství
     - **Mám na skladě:** ingredience kde sklad je `dost` nebo `hodně`

5. **Seskup dle kategorií:**
   - Maso a ryby
   - Zelenina a ovoce
   - Mléčné výrobky
   - Pečivo
   - Suché a trvanlivé
   - Koření a dochucovadla
   - Oleje a tuky
   - Nápoje
   - Ostatní

6. **Zapiš:**
   - Přečti šablonu `${CLAUDE_PLUGIN_ROOT}/sablony/nakup.md`
   - Vyplň data
   - Zapiš do `$RENDL_HOME/nakupy/{{YYYY-MM-DD}}.md`
   - Zobraz uživateli výsledný seznam

### 2. Zobrazení seznamu

Aktivace: "ukaž nákupní seznam", "co mám nakoupit?"

Postup:
1. Najdi nejnovější seznam se stavem `aktivní` v `$RENDL_HOME/nakupy/`
2. Zobraz obsah
3. Pokud žádný aktivní neexistuje → "Nemáš žádný aktivní nákupní seznam."

### 3. Dokončení nákupu

Aktivace: "nakoupeno", "mám nakoupeno", "nakoupil jsem"

Postup:
1. Najdi aktivní nákupní seznam
2. Zeptej se: "Nakoupil jsi všechno ze seznamu, nebo chceš upřesnit co jsi koupil?"
   - Pokud vše → aktualizuj celý sklad (všechny položky ze seznamu na `dost`, aktualizuj `posledni_nakup`)
   - Pokud částečně → projdi položky, uživatel řekne co koupil/nekoupil
3. Aktualizuj sklad pro nakoupené položky
4. Změň stav seznamu na `nakoupeno`
5. Potvrď v tónu persony

## Mapování diakritiky pro názvy souborů

á→a, č→c, ď→d, é→e, ě→e, í→i, ň→n, ó→o, ř→r, š→s, ť→t, ú→u, ů→u, ý→y, ž→z
```

- [ ] **Step 2: Commit**

```bash
cd /home/paffka/Workspace/digital/rendl
git add plugins/rendl/skills/nakup/
git commit -m "feat: add rendl:nakup skill (shopping list)"
```

---

### Task 9: Move design spec and plan into repo, final commit

**Files:**
- Already exists: `docs/superpowers/specs/2026-03-26-rendl-design.md`
- Already exists: `docs/superpowers/plans/2026-03-27-rendl-plugin.md`

- [ ] **Step 1: Commit docs**

```bash
cd /home/paffka/Workspace/digital/rendl
git add docs/
git commit -m "docs: add design spec and implementation plan"
```

- [ ] **Step 2: Verify complete structure**

```bash
find /home/paffka/Workspace/digital/rendl -type f | sort
```

Expected output:
```
.claude-plugin/marketplace.json
.claude/settings.local.json
.gitignore
docs/superpowers/plans/2026-03-27-rendl-plugin.md
docs/superpowers/specs/2026-03-26-rendl-design.md
plugins/rendl/.claude-plugin/plugin.json
plugins/rendl/hooks/context-session.md
plugins/rendl/hooks/hooks.json
plugins/rendl/sablony/ingredience.md
plugins/rendl/sablony/jidelnicek.md
plugins/rendl/sablony/nakup.md
plugins/rendl/sablony/persona-default.md
plugins/rendl/sablony/persona.md
plugins/rendl/sablony/profil.md
plugins/rendl/sablony/recept.md
plugins/rendl/skills/jidelnicek/SKILL.md
plugins/rendl/skills/nakup/SKILL.md
plugins/rendl/skills/recept/SKILL.md
plugins/rendl/skills/setup/SKILL.md
plugins/rendl/skills/sklad/SKILL.md
```
