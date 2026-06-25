# Obchodný model

## Ako SVNT SUPPLY zarába

### 1. Partnerské odkazy (Affiliate)

Každý produkt obsahuje jeden alebo viacero **nákupných odkazov** (buy links) smerujúcich k externým predajcom (Amazon, oficiálne e-shopy značiek, špecializované obchody). Tieto odkazy obsahujú affiliate tagy. Ak návštevník klikne na odkaz a nakúpi, SVNT SUPPLY získa províziu.

Kľúčové detaily:
- **Viacero odkazov na jeden produkt**: Pre každý produkt je možné pridať viacero možností nákupu (napr. Amazon US, Amazon DE, oficiálny e-shop). Administračné rozhranie (Admin UI) podporuje viacero nákupných odkazov s vlastnými popismi a poznámkami o dostupnosti.
- **Žiadne skladové zásoby ani nákupný košík**: SVNT SUPPLY nevlastní skladové zásoby, nespracováva platby ani nerieši dopravu. Funguje výhradne ako referenčná a inšpiratívna vrstva (discovery & referral).
- **Zobrazenie cien**: Produkty zobrazujú ceny v EUR a USD. Návštevník môže meny v e-shope ľubovoľne prepínať.

### 2. Monetizácia newslettera

Newsletter slúži ako retention nástroj (na udržanie používateľov) a zároveň ako zdroj príjmov:

- **Súčasnosť**: Newsletter buduje pravidelnú návštevnosť. Odberatelia dostávajú upozornenia na nové dropy, editoriálne výbery a produktové tipy. Vyššia návštevnosť znamená viac affiliate preklikov.
- **Budúcnosť**: Sponzorovaný obsah v rámci e-mailových kampaní. Značky si budú môcť zaplatiť za to, aby bol ich produkt prednostne zobrazený v sekcii „Partner Pick“ alebo ako hlavný článok. Infraštruktúra newslettera (nástroj na tvorbu kampaní, import z Canvy, analytický prehľad, integrácia s e-mailovým poskytovateľom) je na to už pripravená – chýba len samotné komerčné spustenie.

### 3. Multi-site potenciál

Celá platforma je navrhnutá ako znovupoužiteľný systém:

- Backend (produkty, kategórie, dropy, značky, newsletter, wishlisty) je pripravený na prevádzku pre viacero nezávislých webov (multi-tenant architektúra).
- Administračný panel (Admin UI) momentálne spravuje jeden katalóg, ale celková architektúra umožňuje spustenie samostatných inštancií s odlišnými doménami, produktmi a brandingom.
- Potenciálne tematické vertikály: technológie, bývanie/lifestyle, móda, darčeky – každá s vlastnou doménou, katalógom a newsletterom.
- Zdieľaná infraštruktúra: jeden backendový projekt, jeden hostingový účet, jedna kódová základňa (codebase) konfigurovaná pomocou systémových premenných (env variables).

### Čo nerobíme

- **Žiadne platené umiestnenia v katalógu**: Produkty v hlavnom prehľade (mriežke) sú vyberané výhradne editoriálne, nie za peniaze. Na tomto princípe stojí dôvera našich používateľov.
- **Nepredávame osobné údaje**: E-mailové adresy odberateľov, dáta z wishlistov ani informácie o správaní používateľov na webe nikomu nepredávame ani nezdieľame.
- **Žiadne reklamné siete**: Stránka neobsahuje žiadne bannerové reklamy ani sledovacie pixely (s výnimkou vlastného nástroja na meranie otvorení e-mailov, ktorý je typu first-party a plne rešpektuje súkromie).

---

## Prehľad zdrojov príjmov

| Zdroj príjmov | Stav | Poznámka |
|---|---|---|
| Partnerské provízie (Affiliate) | **Aktívne (Live)** | Viacero nákupných odkazov na produkt. Hlavný zdroj príjmov. |
| Sponzorstvo newslettera | **V príprave** | Infraštruktúra je pripravená. Hľadajú sa obchodní partneri. |
| Licencovanie platformy (Multi-site) | **V budúcnosti** | White-label riešenie pre iných kurátorov a tematické weby. |

---

## Kurátorská konkurenčná výhoda (moat)

Hlavná hodnota tohto biznisu nespočíva v kóde – je ňou **editoriálny filter**. Každý dokáže vytvoriť web s affiliate odkazmi. Oveľa ťažšie je:
1. Konzistentne vyberať skvelé produkty.
2. Písať úprimné a užitočné recenzie.
3. Budovať publikum, ktoré vašim výberom dôveruje.
4. Udržiavať vysokú kvalitu aj pri raste katalógu.

SVNT SUPPLY stavia na vkuse, nie na technológii. Technologické riešenie slúži výhradne editoriálnemu zámeru – rýchle načítanie, čistý dizajn, žiadny zbytočný balast (clutter) a žiadne manipulatívne techniky (dark patterns). Práve to definuje náš produkt.
