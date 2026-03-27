# Rendl — kuchyňský asistent

Claude Code plugin, který z Claude vytvoří komplexního AI asistenta do kuchyně. Spravuje recepty, sklad ingrediencí, týdenní jídelníčky a nákupní seznamy. Defaultně vystupuje jako Rendl — přátelský kuchař s vtipem.

## Co Rendl umí

- **Persona** — defaultně Rendl (přátelský, vtipný kuchař), měnitelná
- **Onboarding** — při prvním spuštění provede nastavením: persona, profil domácnosti, vybavení, diety, pravidla, nutriční cíle
- **Import receptů** — z webu (URL) nebo textu, automatická konverze na jednoduché míry (hrnek, lžíce, lžička, špetka...)
- **Nutriční data** — makra (kcal, bílkoviny, sacharidy, tuky) z Open Food Facts API, s fallbackem na odhad
- **Sklad ingrediencí** — přibližné množství (hodně/dost/málo/dochází/nemám), načtení z účtenky
- **Týdenní jídelníček** — plánování s ohledem na diety, alergie, frekvenční pravidla, nutriční cíle a stav skladu
- **Nákupní seznam** — automaticky z jídelníčku vs. sklad, seskupený dle kategorií
- **Proaktivní chování** — při startu session připomene dnešní jídelníček, aktivní nákupní seznam a dochízející ingredience

## Instalace

```bash
claude plugin marketplace add paffkaDigital/rendl
claude plugin install rendl
```

Aktualizace po nové verzi:

```bash
claude plugin marketplace refresh
```

## Pouziti

Rendl reaguje na přirozený jazyk. Příklady:

```
# Onboarding a nastavení
Nastav mi profil.
Změň diety.
Přidej vybavení.

# Recepty
Přidej recept: https://example.com/kureci-stir-fry
Tady je recept: [vložit text]
Ukaž recepty.
Najdi recept na těstoviny.

# Sklad
Co mám doma?
Tady je účtenka: [vložit text]
Dochází mi mouka.
Uvařil jsem kuřecí stir-fry.

# Jídelníček
Naplánuj týden.
Co je dnes?
V úterý nechci rybu.

# Nákupní seznam
Co nakoupit?
Nakoupeno.
```

## Struktura dat

Uživatelská data se ukládají do `~/.config/rendl/` (persistentní, přežijí aktualizace pluginu):

```
~/.config/rendl/
  persona.md         # persona asistenta (jméno, styl)
  profil.md          # profil domácnosti (osoby, diety, alergie, vybavení, cíle)
  recepty/           # recepty (.md)
  sklad/             # ingredience na skladě (.md)
  jidelnicky/        # týdenní jídelníčky (.md)
  nakupy/            # nákupní seznamy (.md)
```

## Licence

MIT
