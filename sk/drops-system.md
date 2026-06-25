# Systém Sunday Dropov

## Filozofia

SVNT SUPPLY nie je trhovisko s nekonečnou ponukou. Je to **kurátorská editoriálna publikácia**, ktorá každý týždeň vydáva malý, zámerne vybraný výber produktov. Sunday Drop je atomická jednotka tohto rytmu — každú nedeľu ide live nová várka ručne vybraných produktov.

Názov je zámerný: **dropy**, nie kolekcie ani vydania. Evokuje kultúru limitovaných edícií zo streetwearu a dizajnových objektov, kde záleží na očakávaní a načasovaní. SVNT SUPPLY neprehliadaš bezcieľne — v nedeľu si prídeš pozrieť, čo je nové.

---

## Ako to funguje

### Vydávací rytmus

- Dropy vychádzajú **každú nedeľu v fixnom popoludňajšom čase**.
- Každý drop má názov (automaticky normalizovaný na `DROP 001`, `DROP 002` atď.) a dátum vydania.
- Produkt patrí práve do jedného dropu (alebo do žiadneho, ak pochádza z obdobia pred dropovým systémom).

### Stavy dropu

| Stav | Význam | Viditeľný verejne? |
|---|---|---|
| **Scheduled** | Budúci dátum vydania. Produkty sú skryté do času vydania. | Nie |
| **Released** | Čas vydania už prešiel. Produkty sú verejne viditeľné. | Áno |
| **Hidden** | Soft-delete adminom. Odstránené z verejného výhľadu, ale zachované. | Nie |

Stav sa počíta pri čítaní, nie sa ukladá. Drop state sa odvíja od vydávacieho času a príznaku soft-delete.

### Gating viditeľnosti produktov

Verejný katalóg filtruje produkty podľa času vydania. Naplánované produkty sú pre návštevníkov storefrontu neviditeľné, kým ich drop nevyjde. To znamená, že admin môže drop pripravovať týždne dopredu — pridávať produkty, písať editorial, nahrávať obrázky — a všetko pôjde live automaticky v nedeľu bez manuálneho kroku.

---

## Admin workflow

1. **Vytvorenie dropu**: Nastavíš názov (automaticky `DROP 00X`) a nedeľný dátum.
2. **Priradenie produktov**: V produktovom editore vyberieš drop z dropdownu. Produkt získa čas vydania dropu.
3. **Náhľad**: Drop je "scheduled". Produkty sú skryté verejne, ale viditeľné v admine.
4. **Vydanie**: V nedeľu sa zruší časová brána. Produkty sa automaticky objavia v katalógu.
5. **Archivácia**: Staré dropy zostávajú v archíve. Landing page boostuje najnovší vydaný drop v popularity score, takže najnovší drop dominuje mriežke počas svojho vydávacieho týždňa.

### Drop management UI

Drop manager poskytuje:
- Zoznam s vyhľadávaním, state badge (Scheduled/Released/Hidden) a počtom produktov
- Create/Edit pohľad s názvom a date pickerom (validuje nedeľu)
- Soft delete (Hide) a Restore pre každý drop
- Pohľad priradených produktov: zobrazí všetky produkty v drope, odstráni produkt z dropu
- Dropy sú zoradené chronologicky, najnovší prvý

---

## Integrácia s landing page

Algoritmus odporúčaní na landing page dáva významný boost produktom v najnovšom vydanom drope. V kombinácii so signálom novosti to znamená, že produkty z nového dropu dominujú vrchnej časti mriežky počas svojho vydávacieho týždňa. Ako vychádzajú novšie dropy, staršie produkty prirodzene klesajú v popularity rankingu.

### Soft delete dizajn

Dropy sa nikdy nehard-deletujú. Soft-delete nastaví časovú značku namiesto odstránenia riadku. Zachováva sa tak referenčná integrita a umožňuje obnovenie. Produkty v hidden dropoch zostávajú v databáze, ale sú vylúčené z verejných dotazov.

---

## Budúce zváženia

- **Viacero dropov za týždeň**: Systém to už podporuje. UI by sa dalo rozšíriť o kalendárový pohľad.
- **Témy dropov**: Dropy by mohli niesť tematický label ("Summer Essentials", "Gift Guide") okrem číslovaného názvoslovia.
- **Notifikácie**: Keďže dropy vychádzajú v známom čase, notifikačný systém by mohol upozorniť odberateľov na nový drop.
