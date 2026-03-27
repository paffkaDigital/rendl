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
