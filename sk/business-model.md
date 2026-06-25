# Biznis model

## Ako SVNT SUPPLY zarába

### 1. Partnerské odkazy

Každý produkt má jeden alebo viac **nákupných odkazov** smerujúcich k externým predajcom (Amazon, oficiálne značkové obchody, špecializované obchody). Tieto odkazy obsahujú affiliate tagy. Ak návštevník klikne a zakúpi, SVNT SUPPLY získa províziu.

Kľúčové detaily:
- **Viacero odkazov na produkt**: Produkt môže obsahovať viacero nákupných možností (napr. Amazon US, Amazon DE, značkový obchod). Admin UI podporuje viacero odkazov s vlastnými popiskami a poznámkami o dostupnosti.
- **Žiadny sklad, žiadna pokladňa**: SVNT SUPPLY nikdy nedrží zásoby, neprocesuje platby ani nezabezpečuje dopravu. Je to čisto discovery a referral vrstva.
- **Zobrazenie cien**: Produkty zobrazujú ceny v EUR a USD. Návštevník môže prepínať menu.

### 2. Monetizácia newsletteru

Newsletter je zároveň retention nástroj a zdroj príjmov:

- **Aktuálne**: Newsletter prináša opakovanú návštevnosť. Odberatelia dostávajú upozornenia na nové dropy, editorialne highlighty a produktové tipy. Vyššia návštevnosť = viac affiliate klikov.
- **Budúcnosť**: Sponzorované umiestnenia v rámci kampaní. Značky zaplatia za to, že ich produkt bude featured v sekcii "Partner Pick" alebo ako hlavný príbeh. Infrastruktúra newsletteru (campaign builder, Canva import, analytický dashboard, integrácia s email providerom) už to podporuje — chýba len komerčná vrstva.

### 3. Multi-site potenciál

Celá platforma je postavená ako znovupoužiteľný systém:

- Backend (produkty, kategórie, dropy, značky, newsletter, wishlisty) je multi-tenant schopný
- Admin panel spravuje jeden katalóg, ale architektúra podporuje spustenie samostatných inštancií s rôznymi doménami, katalógmi a brandingom
- Potenciálne vertikály: čisto tech, home/lifestyle, fashion, gifting — každá s vlastnou doménou, katalógmi a newsletterom
- Zdieľaná infraštruktúra: jeden backend projekt, jeden hostingový účet, jedna codebase s environment-based konfiguráciou

### Čo nerobíme

- **Žiadne platené umiestnenia v katalógu**: Produkty v hlavnej mriežke sú editoriálne vyberané, nie platené. Dôvera závisí od tohto princípu.
- **Nepredávame dáta používateľov**: Emaily odberateľov, wishlist dáta, správanie návštevníkov sa nepredávajú ani nezdieľajú.
- **Žiadne reklamné siete**: Žiadne display reklamy, žiadne trackovacie pixely okrem vlastného newsletter open trackera (first-party a v súlade s ochranou súkromia).

---

## Prehľad príjmových prúdov

| Prúd | Stav | Poznámka |
|---|---|---|
| Partnerské provízie | **Live** | Viacero odkazov na produkt. Hlavný zdroj príjmov. |
| Sponzorstvá newslettera | **Plánované** | Infraštruktúra pripravená. Potrebné komerčné partnerstvá. |
| Multi-site licencovanie | **Budúcnosť** | White-label platforma pre iných kurátorov/vertikály. |

---

## Kurátorský priekop

Biznis hodnota nie je kód — je to **editoriálny filter**. Každý vie postaviť affiliate stránku. Ťažšie je:
1. Konzistentne vyberať dobré produkty
2. Písať úprimné, užitočné recenzie
3. Budovať publikum, ktoré ti verí
4. Udržiavať kvalitu aj pri raste katalógu

SVNT SUPPLY súťaží na základe vkusu, nie technológie. Tech slúži editoriálnemu hlasu — rýchle načítanie, čistý dizajn, žiadny clutter, žiadne dark patterns. To je produkt.
