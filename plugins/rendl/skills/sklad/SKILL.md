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
