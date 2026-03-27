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
