# Prehľad funkcií administrátorského panelu

## Architektúra

Administračný panel je chránený prostredníctvom Supabase Auth a ACL whitelistu (zoznamu povolených administrátorov). Pozostáva z dvoch základných vrstiev:

- **Dátové moduly (Data stores)** – čisté TypeScript moduly, ktoré komunikujú so Supabase alebo Edge funkciami. Sú napísané bez závislosti na Reacte a dajú sa testovať samostatne.
- **UI komponenty** – React komponenty, ktoré využívajú dáta z dátových modulov. Zobrazenie rozhrania je vynútené výhradne pre desktopové zariadenia.

Každá zmena dát (mutácia) prechádza cez zabezpečenie na úrovni riadkov (Row-Level Security - RLS). Operácie s úložiskom (storage) využívajú zabezpečené (podpísané) URL adresy. Žiadna administrátorská operácia sa nedotkne databázy bez úspešného overenia totožnosti (autentifikácie) a kontroly oprávnení v zozname ACL.

---

## Autentifikácia a relácie (Auth & Sessions)

| Funkcia | Účel |
|---|---|
| `signInWithManagement` | Prihlásenie pomocou e-mailu a hesla cez Supabase + kontrola ACL voči zoznamu povolených administrátorov. Volá Edge funkciu na obmedzenie frekvencie prihlasovania (rate limiting). Pri lokálnom vývoji podporuje simulované (mock) prihlásenie. |
| `createManagementSession` | Vytvorí administrátorskú reláciu (session) s kryptograficky náhodným tokenom, odtlačkom (fingerprintom) hostiteľa, pevným časom platnosti (TTL) a časovým limitom nečinnosti. |
| `getActiveManagementSession` | Overí a predĺži aktívnu reláciu (session). Porovnanie odtlačku (fingerprintu) prebieha v konštantnom čase. Ak je relácia neplatná alebo expirovaná, vráti `null`. |
| `clearManagementSession` | Vymaže reláciu (session) z lokálneho úložiska (`localStorage`) a odhlási používateľa zo služby Supabase. |
| `getAttemptSnapshot` | Vráti aktuálny stav pokusov o prihlásenie: počet neúspešných pokusov v danom časovom intervale, zostávajúci počet pokusov a stav uzamknutia účtu. |
| `registerFailedAttempt` | Zaznamená neúspešný pokus o prihlásenie a uplatní exponenciálne predĺženie doby čakania (cooldown). |
| `RequireManagementSession` | Ochranca smerovania (Route guard). Overuje autentifikáciu a ACL oprávnenia pri prvotnom načítaní aj pri každej zmene podstránky (route). V prípade neplatnej relácie presmeruje používateľa na prihlasovaciu stránku. |

---

## Produktový CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listProducts` | Stránkovaná tabuľka produktov s podporou vyhľadávania podľa názvu, značky alebo kategórie. |
| **Načítanie dát** | `loadProductBundle` | Načíta kompletné dáta o produkte (bundle): základné informácie, preklady, farby, mediálne súbory, prezentácie (orezy), nákupné odkazy a súvisiace produkty. |
| **Vytvorenie** | `createEmptyBundle` | Vytvorí prázdny koncept (draft) produktu s kostrou anglického prekladu, nastavením pre zobrazenie na kartách a detaile produktu (PDP) a jedným predvoleným nákupným odkazom. |
| **Aktualizácia** | `saveProductBundle` | Uloží alebo aktualizuje (upsertuje) dáta o produkte, jeho preklady, farby, nákupné odkazy a odporúčania. Zároveň spravuje životný cyklus pridružených médií. |
| **Publikovanie** | `publishProduct` | Skontroluje vyplnenie povinných polí, zmení stav produktu na zverejnený a zneplatní (invaliduje) vyrovnávaciu pamäť (cache) katalógu. |
| **Archivovanie** | `archiveProduct` | Zmení stav produktu na archivovaný a zneplatní cache katalógu. |
| **Zmazanie** | `deleteProduct` | Trvale vymaže produkt z databázy a zaradí pridružené mediálne súbory do radu na oneskorené odstránenie z úložiska. |

**Operácie s médiami:**

| Operácia | Funkcia | Účel |
|---|---|---|
| **Požiadavka na nahrávanie** | `requestMediaUploadUrl` | Získa zabezpečenú (podpísanú) URL adresu z Edge funkcie pre nahrávanie produktových obrázkov. |
| **Nahranie súboru** | `uploadMediaFile` | Nahrá súbor metódou PUT na vygenerovanú podpísanú URL adresu. |
| **Dokončenie (Finalizácia)** | `finalizeUploadedProductBundle` | Volá Edge funkciu, ktorá spracuje nahraný obrázok (detekuje rozmery, alfa kanál a ohraničenie - bounding box). V prípade dočasných chýb pokus automaticky zopakuje. |
| **Odstránenie z úložiska** | `deleteMediaStorageObjects` | Vymaže príslušné súbory z cloudového úložiska prostredníctvom Edge funkcie. |

**Polia formulára na úpravu produktu:**
- Identifikátor (slug), značka, kategórie (multi-select výber s označením primárnej kategórie), ceny v EUR a USD, priradenie k nedeľnému dropu.
- Obrázok: nahrávanie PNG súborov pomocou drag-and-drop, duálny náhľadový panel, nástroje na posúvanie, približovanie (zoom) a automatické prispôsobenie (auto-fit).
- Farby: mriežka klasických farebných vzoriek, ktoré je možné jednoducho prepínať.
- Nákupné odkazy: webová adresa (URL), štítok odkazu (s podporou viacerých odkazov pre jeden produkt).
- Redakčná časť (Editorial): názov produktu, celkové zhodnotenie (verdict), popis produktu, dôvod zaradenia do výberu („Why it made the cut“), výhody a nevýhody, skóre odolnosti, sekcia pre koho produkt je a pre koho nie je určený.
- Prepínače a značky: príznak „Staff Pick“ (odporúčanie redakcie) a stavové štítky (status badges).

---

## Kategóriový CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listCategories` | Načíta všetky kategórie zoradené podľa určeného poradia pre desktop. |
| **Hromadné uloženie** | `saveCategories` | Hromadne uloží alebo aktualizuje (upsertuje) názvy, poradie a priradené obrázky. Zápis zároveň zneplatní (invaliduje) cache úvodnej stránky. |
| **Nahrávanie obrázka** | `uploadCategoryImage` | Optimalizuje a nahrá obrázok pre dlaždicu kategórie cez zabezpečené (podpísané) URL. Vracia verejný odkaz na obrázok. |

**Funkcie editora kategórií:**
- Samostatné nastavenie poradia pre desktopové a mobilné zobrazenie pomocou drag-and-drop.
- Inline úprava textu názvu priamo na dlaždici kategórie.
- Nahrávanie a výmena obrázka pre každú dlaždicu s okamžitým náhľadom.
- Možnosť prepínania medzi desktopovým a mobilným náhľadom (mobilné zobrazenie je zasadené do verného mockupu iPhonu).
- Hromadné ukladanie (batch) – chýba možnosť vytvárať alebo mazať kategórie jednotlivo (kategórie tvoria pevne definovaný kurátorský set).

---

## Dropový (vydávací) CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Prehľad** | `listDrops` | Zobrazí zoznam všetkých dropov spolu s ich vypočítaným stavom a počtom priradených produktov. |
| **Načítanie** | `loadDrop` | Načíta podrobné informácie o konkrétnom drope na základe jeho ID. |
| **Vytvorenie** | `createDrop` | Vytvorí nový nedeľný drop s určeným dátumom vydania. Zápis zneplatní (invaliduje) cache katalógu. |
| **Aktualizácia** | `updateDrop` | Umožňuje zmeniť názov alebo dátum naplánovaného dropu. Zápis zneplatní cache katalógu. |
| **Dočasné zmazanie** | `softDeleteDrop` | Označí drop príznakom zmazania (soft-delete), čím ho skryje pred verejnosťou. Zápis zneplatní cache katalógu. |
| **Obnovenie** | `restoreDrop` | Zruší príznak zmazania, vďaka čomu sa drop opäť zviditeľní. Zápis zneplatní cache katalógu. |
| **Zoznam produktov** | `getDropProducts` | Načíta produkty priradené k danému dropu vrátane ich značky a názvu. |
| **Odstránenie produktu** | `removeProductFromDrop` | Zruší prepojenie medzi produktom a jeho aktuálnym dropom. |

**Biznis pravidlá:**
- Plánované dátumy zverejnenia musia pripadať vždy na nedeľu v presne stanovenom popoludňajšom čase.
- Názvy dropov sa v databáze automaticky formátujú do podoby `DROP 001`, `DROP 002` atď.
- Produkt môže byť v rovnakom čase priradený najviac k jednému nedeľnému dropu.
- Upravovať je možné iba naplánované dropy (Scheduled), už vydané dropy (Released) sú z bezpečnostných dôvodov nemenné.

---

## CRUD značiek

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listStorefrontBrands` | Zobrazí zoznam všetkých značiek zoradených abecedne. |
| **Hromadné uloženie** | `saveStorefrontBrands` | Skontroluje duplicity (v názvoch alebo sluggách) a hromadne uloží alebo aktualizuje (upsertuje) pole značiek. |
| **Zmazanie** | `deleteStorefrontBrands` | Vymaže zvolené značky na základe ich ID a súčasne vyčistí pridružené súbory log zo cloudového úložiska. |
| **Nahrávanie loga** | `uploadBrandLogo` | Optimalizuje a nahrá súbor loga značky prostredníctvom zabezpečeného (podpísaného) URL. Vracia URL odkaz na logo a metadáta úložiska. |
| **Vymazanie loga** | `deleteBrandLogoFiles` | Odstráni súbory loga konkrétnej značky z cloudového úložiska. |

**UI funkcie:**
- Responzívna mriežka s kartami jednotlivých značiek.
- Inline úprava názvu značky s funkciou automatického ukladania zmien.
- Nahrávanie loga kliknutím na kartu alebo jednoduchým pretiahnutím súboru (drag-and-drop).
- Grafické prekrytie (overlay) s možnosťou nahradenia („Replace“) pre existujúce logá.
- Inteligentné návrhy na doplnenie značiek automaticky generované z databázy produktov.

---

## CRUD e-mailových kampaní (Newsletter)

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listCampaigns` | Stránkovaný prehľad e-mailových kampaní s filtrom stavu a fultextovým vyhľadávaním. |
| **Načítanie** | `loadCampaign` | Načíta detail vybranej kampane vrátane jej kompletného HTML obsahu. |
| **Uloženie** | `saveCampaign` | Vytvorí alebo aktualizuje kampaň na serveri (s automatickým záložným ukladaním do `localStorage` prehliadača). |
| **Zmazanie** | `deleteCampaign` | Trvale vymaže kampaň, jej súbory (assety) v úložisku a prečistí lokálnu cache. |
| **Odoslanie** | `markCampaignSent` | Zmení stav kampane na odoslanú a zaznamená presný počet adresátov. |
| **Posledné odoslanie** | `getLatestSentCampaignAt` | Vráti časovú pečiatku poslednej úspešne odoslanej kampane. |

**Canva import pipeline:**

| Krok | Funkcia | Účel |
|---|---|---|
| **Import priečinka** | `importCanvaFolder` | Načíta vstupný HTML súbor, spracuje obrázky (assety), optimalizuje ich do formátu WebP a vráti kompletnú analýzu. |
| **Analýza** | `analyzeCanvaHtmlDocument` | Vyčistí (sanitizuje) HTML kód a zistí všetky odkazy na obrázky, prípadne upozorní na chýbajúce či chybné položky. |
| **Sanitizácia** | `sanitizeCanvaHtmlDocument` | Odstráni potenciálne nebezpečné prvky alebo skripty a optimalizuje kód pre maximálnu kompatibilitu s e-mailovými klientmi. |
| **Prepísanie adries** | `rewriteCanvaHtmlImageUrls` | Nahradí lokálne cesty obrázkov v HTML kóde za verejné odkazy na cloudové úložisko. |
| **Náhľad** | `buildCanvaPreviewHtml` | Vygeneruje náhľadový HTML kód so správne prepojenými obrázkami. |
| **Trvalé uloženie** | `persistCanvaImportedAssets` | Lokálne uloží optimalizované obrázky pre ich neskoršie odoslanie na server. |

**UI builder kampane:**
- Podpora nahrávania priečinka pomocou drag-and-drop alebo výberom zo súborového systému (directory picker).
- Interaktívny náhľad e-mailu pre desktopové a mobilné zobrazenie.
- Bočný štatistický panel so zoznamom obrázkov a assetov.
- Ukladací formulár (save modal): nastavenie názvu kampane, predmetu správy a mena odosielateľa.

---

## Analytika newslettera (iba na čítanie)

| Operácia | Funkcia | Účel |
|---|---|---|
| **Súhrnný prehľad** | `getSnapshot` | Agreguje kľúčové metriky za vybrané obdobie: celkový počet odberateľov, počet aktívnych používateľov, odhlásenia a nedoručené e-maily (bounces). |
| **Rýchle štatistiky** | `getOverview` | Poskytuje okamžitý prehľad: počet odberateľov, počet odoslaných kampaní, priemernú mieru otvorenia (open rate) a mieru nedoručenia (bounce rate). |
| **Zoznam odberateľov** | `getSubscribersPage` | Zobrazí stránkovaná tabuľka odberateľov s možnosťou vyhľadávania, filtrovania podľa stavu a zobrazenia dátumov registrácie. |
| **Výkon kampaní** | `getCampaignsPage` | Stránkovaný zoznam odoslaných kampaní s podrobnými štatistikami doručenia a čítanosti. |

**Časové zoskupovanie (bucketing):** Získané metriky sa zoskupujú do definovaných časových intervalov. Trendy a časové rady mier (rate series) sa počítajú ako intervalové aj kumulatívne.

---

## Edge funkcie

| Funkcia | Účel |
|---|---|
| `management-login-throttle` | Ochrana prihlasovania pred zahltením (rate limiting) na základe hashovanej IP adresy. |
| `product-media-upload` | Generovanie zabezpečených (podpísaných) URL pre nahrávanie a mazanie produktových obrázkov. |
| `product-media-cleanup-worker` | Asynchrónna úloha, ktorá na pozadí odstraňuje nepoužívané (osirelé) produktové obrázky. |
| `product-media-public` | Vyrovnávacia pamäť (caching proxy) na rýchle doručovanie produktových obrázkov. |
| `category-media-upload` | Zabezpečené a optimalizované nahrávanie obrázkov pre dlaždice kategórií. |
| `brand-logo-upload` | Nahrávanie a odstraňovanie súborov s logami značiek. |
| `newsletter-campaign-save` | Ukladanie rozpracovaných konceptov e-mailových kampaní vrátane ich HTML obsahu. |
| `newsletter-campaign-finalize` | Uzamknutie a príprava kampane tesne pred jej odoslaním. |
| `newsletter-campaign-send` | Distribúcia kampane adresátom prostredníctvom e-mailového poskytovateľa (email provider). |
| `newsletter-analytics-snapshot` | Agregácia a spracovanie analytických dát o úspešnosti kampaní. |
| `newsletter-bounce-retry-worker` | Automatické opätovné pokusy o doručenie e-mailov pri dočasných chybách (soft bounces). |
| `wishlist-share-create` / `wishlist-share-read` | Vytvorenie a prezeranie anonymných odkazov na zdieľanie zoznamu prianí (wishlistu). |

---

## Dátové entity

| Entita | Účel |
|---|---|
| `management_admins` | Zoznam oprávnených administrátorov (ACL whitelist) s príznakom aktivity. Chránené pomocou RLS. |
| `management_login_attempts` | Záznam (audit log) pokusov o prihlásenie s hashovanými IP adresami. |
| `products` | Produktový katalóg: slug, značka, ceny, stav zverejnenia, staff pick, skóre popularity a priradený nedeľný drop. |
| `product_translations` | Anglické redakčné texty (editorial) pre jednotlivé produkty. |
| `product_colors` | Priradené vzorkovníky farieb pre produkty. |
| `product_media_assets` | Metadáta obrázkov a priame odkazy do cloudového úložiska. |
| `product_media_presentations` | Konfigurácia orezov, mierky a formátu (crop/scale/trim) pre zobrazenie v kartách a hero sekciách. |
| `product_buy_links` | Partnerské nákupné odkazy (affiliate links). |
| `product_related_products` | Kurátorsky určené odporúčania pre súvisiace produkty („Mohlo by sa vám páčiť“). |
| `product_categories` | Nastavenia dlaždíc kategórií na úvodnej stránke vrátane ich poradia a pozadí. |
| `product_drops` | Evidencia nedeľných dropov (Sunday Drops) s podporou dočasného zmazania (soft-delete). |
| `storefront_brands` | Administráciou spravované dáta o značkách a ich logách. |
| `newsletter_subscribers` | Databáza odberateľov noviniek s ich aktuálnym stavom a tokenmi pre odhlásenie. |
| `newsletter_campaigns` | Rozpracované a odoslané e-mailové kampane vrátane HTML obsahu a štatistík. |
| `newsletter_campaign_recipients` | Sledovanie doručenia e-mailov pre jednotlivých adresátov. |

---

## Bezpečnostný model

1. **Overenie totožnosti (Autentifikácia)**: Prihlasovanie cez e-mail a heslo prostredníctvom služby Supabase. Jednorazové odkazy na prihlásenie (magic links) nie sú podporované.
2. **Riadenie prístupu (ACL)**: Whitelist administrátorov sa overuje pri každej kontrole aktívnej relácie. Prístup je povolený výhradne aktívnym členom zoznamu.
3. **Zabezpečenie relácie (Session)**: Vynútené pevné trvanie relácie (absolútne TTL), automatické odhlásenie pri nečinnosti, overenie odtlačku hostiteľa (hostname fingerprint) a kryptograficky náhodný token.
4. **Ochrana pred zahltovaním (Rate limiting)**: Klientska aplikácia uplatňuje exponenciálne narastajúci čas čakania po chybách, server zase obmedzuje frekvenciu pokusov na základe hashovanej IP adresy.
5. **Úložisko súborov (Storage)**: Všetky nahrávania (uploady) prebiehajú výhradne cez jednorazové zabezpečené (podpísané) URL. Verejné sťahovanie z úložiska (bucketu) je sprostredkované cez Edge funkcie, ktoré k odpovediam pridávajú dlhodobé HTTP cache hlavičky.
6. **Zabezpečenie databázy (RLS)**: Každá tabuľka má striktne definované pravidlá prístupu na úrovni riadkov (Row-Level Security). Administračné tabuľky sú uzamknuté a povolené iba pre prihlásených používateľov, ktorí sa zároveň nachádzajú v zozname ACL.
