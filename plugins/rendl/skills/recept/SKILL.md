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
