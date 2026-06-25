# Systém nedeľných dropov (Sunday Drops)

## Filozofia

SVNT SUPPLY nie je trhovisko s nekonečnou ponukou. Je to **kurátorovaná editoriálna publikácia**, ktorá každý týždeň prináša malý, starostlivo zostavený výber produktov. Nedeľný drop (Sunday Drop) je základnou jednotkou tohto cyklu – každú nedeľu vychádza nová várka ručne vybraných produktov.

Názov je zvolený zámerne: **dropy**, nie kolekcie či vydania. Evokuje to kultúru limitovaných edícií zo sveta streetwearu a dizajnových produktov, kde zohrávajú kľúčovú rolu očakávanie a správne načasovanie. SVNT SUPPLY si neprezeráte bezcieľne – v nedeľu sa sem vraciate preto, aby ste zistili, čo je nové.

---

## Ako to funguje

### Rytmus vydávania

- Dropy vychádzajú **každú nedeľu v pevne stanovenom popoludňajšom čase**.
- Každý drop má svoj názov (automaticky formátovaný ako `DROP 001`, `DROP 002` atď.) a dátum vydania.
- Produkt patrí práve do jedného dropu (prípadne do žiadneho, ak bol do e-shopu pridaný ešte pred zavedením tohto systému).

### Stavy dropu

| Stav | Význam | Viditeľný verejne? |
|---|---|---|
| **Scheduled** | Budúci dátum vydania. Produkty sú skryté až do plánovaného času vydania. | Nie |
| **Released** | Čas vydania už prešiel. Produkty sú verejne viditeľné. | Áno |
| **Hidden** | Dočasne vymazané adminom (soft-delete). Odstránené z verejnej časti webu, no zachované v databáze. | Nie |

Stav dropu sa dynamicky vypočítava pri načítaní (neukladá sa priamo do databázy). Odvíja sa od plánovaného času vydania a príznaku soft-delete.

### Časové obmedzenie viditeľnosti produktov (Gating)

Verejný katalóg filtruje produkty podľa času zverejnenia. Naplánované produkty sú pre návštevníkov e-shopu neviditeľné, kým príslušný drop oficiálne nevyjde. To znamená, že administrátor môže drop pripravovať aj týždne vopred (pridávať produkty, písať sprievodné texty, nahrávať obrázky) a všetko sa automaticky zverejní v nedeľu bez nutnosti akéhokoľvek manuálneho zásahu.

---

## Admin workflow

1. **Vytvorenie dropu**: Zadáte názov (systém automaticky navrhne `DROP 00X`) a vyberiete nedeľný dátum.
2. **Priradenie produktov**: V editore produktov zvolíte príslušný drop z rozbaľovacieho zoznamu. Produkt tým prevezme plánovaný čas zverejnenia daného dropu.
3. **Náhľad**: Drop je v stave „scheduled“ (naplánovaný). Produkty sú na verejnom webe skryté, ale v administrácii sú normálne viditeľné.
4. **Zverejnenie (Vydanie)**: V nedeľu sa časová závora automaticky uvoľní a produkty sa okamžite objavia v katalógu.
5. **Archivácia**: Staršie dropy zostávajú dostupné v archíve. Algoritmus úvodnej stránky (landing page) dáva najnovšiemu vydanému dropu extra bonifikáciu v skóre popularity, vďaka čomu najnovšie produkty dominujú na popredných miestach mriežky počas prvého týždňa od vydania.

### Drop management UI

Administračný nástroj pre správu dropov poskytuje:
- Prehľadný zoznam s vyhľadávaním, stavovými štítkami (Scheduled/Released/Hidden) and počtom priradených produktov.
- Formulár pre vytvorenie a úpravu s názvom a výberom dátumu (ktorý validuje, či ide o nedeľu).
- Možnosť dočasného zmazania (Hide) a opätovného obnovenia (Restore) pre každý drop.
- Prehľad priradených produktov: zobrazenie všetkých produktov v danom drope a možnosť ich rýchleho odstránenia.
- Usporiadanie dropov chronologicky, od najnovšieho po najstarší.

---

## Integrácia s landing page

Algoritmus odporúčaní na úvodnej stránke výrazne zvýhodňuje produkty z najnovšieho vydaného dropu. V kombinácii s dátumom pridania to zabezpečuje, že novinky dominujú hornej časti mriežky počas celého prvého týždňa. S príchodom novších dropov staršie produkty prirodzene klesajú v rebríčku popularity.

### Návrh dočasného zmazania (Soft delete)

Dropy sa z databázy nikdy neodstraňujú natrvalo (hard-delete). Funkcia soft-delete iba nastaví časovú pečiatku zmazania namiesto vymazania celého riadka. Tým sa zachováva referenčná integrita dát a uľahčuje prípadné obnovenie. Produkty v skrytých dropoch zostávajú v databáze, ale sú vynechané z verejných dopytov.

---

## Budúci rozvoj

- **Viacero dropov za týždeň**: Databázový systém to už plne podporuje. Administračné rozhranie by sa dalo rozšíriť napríklad o kalendárový náhľad.
- **Témy dropov**: Dropy by mohli okrem číselného označenia niesť aj tematické štítky (napr. „Summer Essentials“ alebo „Gift Guide“).
- **Notifikácie**: Keďže dropy vychádzajú v pevne stanovenom čase, notifikačný systém by mohol odberateľov automaticky upozornit na spustenie novej várky produktov.
