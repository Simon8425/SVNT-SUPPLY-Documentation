# Register funkcií admin panela

## Architektúra

Admin panel je chránený cez Supabase Auth a ACL whitelist. Pozostáva z dvoch vrstiev:

- **Dátové úložiská** — čisté TypeScript moduly, ktoré volajú Supabase alebo edge funkcie. Bez Reactu, testovateľné samostatne.
- **UI komponenty** — React komponenty, ktoré využívajú dátové úložiská. Prístup je obmedzený len na desktop.

Každá mutácia prechádza cez Row-Level Security. Operácie so storage používajú podpísané URL. Žiadna admin operácia nedotkne databázy bez autentifikácie a členstva v ACL.

---

## Autentifikácia a session

| Funkcia | Účel |
|---|---|
| `signInWithManagement` | Prihlásenie cez Supabase email/heslo + kontrola ACL voči whitelistu administrátorov. Volá edge funkciu na obmedzenie rýchlosti prihlasovania. V lokálnom vývoji je k dispozícii mock autentifikácia. |
| `createManagementSession` | Vytvorí session s kryptograficky náhodným tokenom, fingerprintom hostname, absolútnym TTL a časovým limitom nečinnosti. |
| `getActiveManagementSession` | Overí a obnoví session. Porovnanie fingerprintu prebieha v konštantnom čase. Ak je session neplatná alebo expirovaná, vráti null. |
| `clearManagementSession` | Vymaže session z localStorage a odhlási používateľa zo Supabase. |
| `getAttemptSnapshot` | Vráti stav prihlasovacích pokusov: počet zlyhaní v okne, zostávajúce pokusy, stav uzamknutia. |
| `registerFailedAttempt` | Zapíše neúspešný pokus a aplikuje exponenciálne spomalenie. |
| `RequireManagementSession` | Route guard. Overuje autentifikáciu + ACL pri načítaní aj pri zmene route. Pri neúspechu presmeruje na prihlásenie. |

---

## Produktový CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listProducts` | Stránkovaný zoznam produktov s vyhľadávaním podľa názvu/značky/kategórie. |
| **Načítanie bundle** | `loadProductBundle` | Kompletný produkt: riadok, preklady, farby, médiá, prezentácie, nákupné odkazy, súvisiace produkty. |
| **Vytvorenie** | `createEmptyBundle` | Koncept produktu s anglickým prekladom, card/PDP prezentáciami a jedným nákupným odkazom. |
| **Aktualizácia** | `saveProductBundle` | Upsertuje produkt, preklady, farby, nákupné odkazy a súvisiace produkty. Riadi životný cyklus médií. |
| **Publikovanie** | `publishProduct` | Overí povinné polia, nastaví status publikovaný a invaliduje cache katalógu. |
| **Archivovanie** | `archiveProduct` | Nastaví status archivovaný a invaliduje cache katalógu. |
| **Zmazanie** | `deleteProduct` | Trvale zmaže produkt a zaradí jeho médiá do fronty na oneskorené zmazanie. |

**Operácie s médiami:**

| Operácia | Funkcia | Účel |
|---|---|---|
| **Vyžiadanie upload URL** | `requestMediaUploadUrl` | Získa podpísané URL na nahratie produktového obrázka z media edge funkcie. |
| **Upload súboru** | `uploadMediaFile` | Nahraje súbor cez PUT na podpísané URL. |
| **Finalizácia** | `finalizeUploadedProductBundle` | Edge funkcia spracuje obrázok (rozmery, detekcia alfa kanála, bounding box). Pri prechodných chybách skúsi znova. |
| **Zmazanie storage** | `deleteMediaStorageObjects` | Odstráni súbory zo storage cez edge funkciu. |

**Polia formulára na úpravu produktu:**
- Slug, značka, kategória (multi-select s primárnou), ceny (EUR/USD), priradenie dropu
- Obrázok: drag-and-drop PNG upload, duálny náhľad, posúvanie/zoom/auto-fit
- Farby: mriežka klasických farebných vzoriek na prepínanie
- Nákupné odkazy: URL, popisok (viacero odkazov na produkt)
- Editorial: názov, verdict, čo to je, prečo to prešlo, výhody/nevýhody, odolnosť, pre koho je/nie je
- Prepínače: Staff Pick (výber redakcie), statusové štítky

---

## Kategóriový CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listCategories` | Všetky kategórie zoradené podľa desktop sort order. |
| **Batch save** | `saveCategories` | Upsertuje názvy, poradie a obrázky. Pri zápise invaliduje cache landing page. |
| **Upload obrázka** | `uploadCategoryImage` | Optimalizuje a nahraje dlaždicu kategórie cez podpísané URL. Vráti verejnú URL. |

**Funkcie editora kategórií:**
- Samostatné poradie pre desktop a mobil s drag-and-drop
- Inline úprava názvu každej dlaždice
- Upload obrázka pre každú dlaždicu s náhľadom
- Prepínač Desktop/Mobil náhľad (mobil vo vnútri iPhone mockupu)
- Ukladá sa ako batch — žiadne jednotlivé vytváranie/mazanie (kategórie sú fixný kurátorský set)

---

## Dropový (vydávací) CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listDrops` | Všetky dropy s vypočítaným stavom a počtom produktov. |
| **Načítanie** | `loadDrop` | Jeden drop podľa ID. |
| **Vytvorenie** | `createDrop` | Nový drop s nedeľným dátumom vydania. Invaliduje cache katalógu. |
| **Aktualizácia** | `updateDrop` | Zmena názvu alebo dátumu naplánovaného dropu. Invaliduje cache katalógu. |
| **Soft delete** | `softDeleteDrop` | Nastaví čas zmazania. Drop sa skryje. Invaliduje cache katalógu. |
| **Obnovenie** | `restoreDrop` | Vymaže čas zmazania. Drop sa znovu zobrazí. Invaliduje cache katalógu. |
| **Zoznam produktov** | `getDropProducts` | Produkty priradené k dropu so značkou a názvom. |
| **Odstránenie produktu** | `removeProductFromDrop` | Zruší priradenie produktu k dropu. |

**Biznis pravidlá:**
- Dátumy vydania musia byť nedele v fixnom popoludňajšom čase.
- Názvy dropov sa automaticky normalizujú na `DROP 001`, `DROP 002` atď.
- Produkt môže patriť len do jedného dropu naraz.
- Upraviť sa dá len naplánovaný drop; vydané dropy sú nemenné.

---

## CRUD značiek

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listStorefrontBrands` | Všetky značky abecedne zoradené. |
| **Batch save** | `saveStorefrontBrands` | Overí duplicity (slugy/názvy) a upsertuje pole značiek. |
| **Zmazanie** | `deleteStorefrontBrands` | Zmaže značky podľa ID. Zároveň vyčistí logo súbory zo storage. |
| **Upload loga** | `uploadBrandLogo` | Optimalizuje a nahraje logo cez podpísané URL. Vráti URL loga a storage metadata. |
| **Zmazanie loga** | `deleteBrandLogoFiles` | Odstráni logo súbory zo storage. |

**UI funkcie:**
- Responzívna mriežka značkových kariet
- Inline úprava názvu s automatickým ukladaním
- Upload loga kliknutím alebo drag-and-drop na kartu
- Overlay "Replace" pre existujúce logá
- Návrhy značiek z katalógu produktov

---

## Newsletter campaign CRUD

| Operácia | Funkcia | Účel |
|---|---|---|
| **Zoznam** | `listCampaigns` | Stránkované kampane s filtrom statusu a textovým vyhľadávaním. |
| **Načítanie** | `loadCampaign` | Jedna kampaň s plným HTML obsahom. |
| **Uloženie** | `saveCampaign` | Vytvorí alebo aktualizuje kampaň v backende (s fallbackom na localStorage). |
| **Zmazanie** | `deleteCampaign` | Zmaže kampaň, jej storage assety a lokálne cache. |
| **Označenie ako odoslaná** | `markCampaignSent` | Nastaví status na odoslanú s počtom príjemcov. |
| **Posledná odoslaná** | `getLatestSentCampaignAt` | Čas poslednej odoslanej kampane. |

**Canva import pipeline:**

| Krok | Funkcia | Účel |
|---|---|---|
| **Import priečinka** | `importCanvaFolder` | Vyberie HTML vstup, vyrieši obrázkové assety, optimalizuje na WebP, vráti analýzu. |
| **Analýza** | `analyzeCanvaHtmlDocument` | Sanitizuje HTML a extrahuje odkazy na obrázky s varovaniami. |
| **Sanitizácia** | `sanitizeCanvaHtmlDocument` | Odstráni nebezpečné bloky/skripty a spevní kompatibilitu s email klientmi. |
| **Prepísanie URL** | `rewriteCanvaHtmlImageUrls` | Nahradí URL assety v HTML za verejné storage URL. |
| **Náhľad** | `buildCanvaPreviewHtml` | Vytvorí náhľad HTML s vyriešenými URL obrázkov. |
| **Perzistencia** | `persistCanvaImportedAssets` | Uloží optimalizované assety lokálne pre neskoršie uloženie. |

**UI builder kampane:**
- Drag-and-drop import priečinka alebo directory picker
- Desktop/Mobil náhľad emailu
- Štatistický panel assety
- Save modal: názov kampane, predmet, odosielateľ

---

## Newsletter analytika (iba na čítanie)

| Operácia | Funkcia | Účel |
|---|---|---|
| **Snapshot** | `getSnapshot` | Agregované metriky za časové obdobie: celkoví odberatelia, aktívni, odhlásení, bounced. |
| **Prehľad** | `getOverview` | Rýchle štatistiky: odberatelia, počet kampaní, open rate, bounce rate. |
| **Stránka odberateľov** | `getSubscribersPage` | Stránkovaná tabuľka odberateľov so statusom, dátumami, vyhľadávaním. |
| **Stránka kampaní** | `getCampaignsPage` | Stránkovaný výkon kampaní s doručovacími štatistikami. |

**Časové bucketovanie:** Metriky sa zoskupujú do štandardných časových okien. Rate série sa počítajú ako bucketované aj kumulatívne.

---

## Edge funkcie

| Funkcia | Účel |
|---|---|
| `management-login-throttle` | IP-hashed rate limiting pri prihlasovaní. |
| `product-media-upload` | Podpísané URL na upload/zmazanie produktových obrázkov. |
| `product-media-cleanup-worker` | Na pozadí maže osirelé produktové assety. |
| `product-media-public` | Cachovací proxy pre produktové obrázky. |
| `category-media-upload` | Optimalizovaný upload dlaždíc kategórií. |
| `brand-logo-upload` | Upload a zmazanie loga značky. |
| `newsletter-campaign-save` | Uloženie konceptu kampane s HTML obsahom. |
| `newsletter-campaign-finalize` | Uzamknutie kampane pred odoslaním. |
| `newsletter-campaign-send` | Odoslanie kampane cez email providera. |
| `newsletter-analytics-snapshot` | Analytické dáta kampaní. |
| `newsletter-bounce-retry-worker` | Opakovaný pokus o doručenie soft bounces. |
| `wishlist-share-create` / `wishlist-share-read` | Vytvorenie a čítanie anonymných zdieľacích odkazov wishlistu. |

---

## Dátové entity

| Entita | Účel |
|---|---|
| `management_admins` | ACL whitelist admin emailov s active flagom. Chránené RLS. |
| `management_login_attempts` | Audit log prihlasovacích pokusov s hashovanými IP. |
| `products` | Produktový katalóg: slug, značka, cena, status, staff pick, popularita, priradenie dropu. |
| `product_translations` | Anglický editorial obsah pre každý produkt. |
| `product_colors` | Farebné vzorky produktu. |
| `product_media_assets` | Metadata obrázkov a storage referencie. |
| `product_media_presentations` | Crop/scale/trim pre card a hero zobrazenia. |
| `product_buy_links` | Partnerské nákupné odkazy. |
| `product_related_products` | Kurátorské "mohlo by sa ti páčiť" vzťahy. |
| `product_categories` | Dlaždice kategórií na landing page s poradím a obrázkami. |
| `product_drops` | Sunday Drop vydania so soft-delete podporou. |
| `storefront_brands` | Admin-riadené metadata značiek s logami. |
| `newsletter_subscribers` | Odberatelia so statusom a unsubscribe tokenmi. |
| `newsletter_campaigns` | Koncepty/odoslané kampane s HTML a doručovacími štatistikami. |
| `newsletter_campaign_recipients` | Sledovanie doručenia po jednotlivých príjemcoch. |

---

## Bezpečnostný model

1. **Autentifikácia**: Supabase email/heslo. Žiadne magic links.
2. **ACL**: Admin whitelist sa kontroluje pri každej validácii session. Prechádzajú len aktívni členovia.
3. **Session**: Absolútny TTL, timeout nečinnosti, hostname fingerprint, kryptograficky náhodný token.
4. **Rate limiting**: Client-side exponenciálne spomalenie + server-side IP-hashed throttle.
5. **Storage**: Všetky uploady používajú podpísané URL. Verejný prístup k bucketu prebieha cez edge funkcie s dlhodobými cache hlavičkami.
6. **RLS**: Každá tabuľka má Row-Level Security. Admin tabuľky sú prístupné len autentifikovaným používateľom v ACL.
