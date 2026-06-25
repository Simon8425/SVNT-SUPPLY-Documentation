# Zobrazovanie produktov a systém aktualizácie

## Mriežka na úvodnej stránke

Úvodná stránka (landing page) zobrazuje publikované produkty v jednej mriežke. Predvolené usporiadanie je **Doporučené** (Featured, na webe označené ako *Editor's choice*).

Poradie doporučených produktov (Featured) riadi skóre popularity (`popularity`), ktoré sa prepočítava na strane servera v pravidelných intervaloch pre každý zverejnený produkt. Toto skóre v sebe spája päť signálov:

| Signál | Váha | Zdroj |
|---|---|---|
| **Novosť (Newness)** | ~35% | Sekvenčné počítadlo (counter) priradené pri prvom zverejnení produktu. Vyššia hodnota = novší produkt. |
| **Staff pick** | ~20% | Manuálny príznak (flag) nastavený administrátorom v editore produktov. |
| **Čerstvosť (Freshness)** | ~10% | Časový pokles (decay) skóre od dátumu vydania alebo publikovania. Čím novší produkt, tým vyššie skóre. |
| **Najnovší drop** | ~15% | Produkt patrí do najnovšieho vydaného nedeľného dropu (Sunday Drop). |
| **Periodický posun (Jitter)** | ~20% | Deterministický hash ID produktu a aktuálneho časového okna. Zabezpečuje obmenu poradia pri každom načítaní bez toho, aby bolo náhodné. |

Novosť a najnovší drop tvoria dohromady približne polovicu celkového skóre, takže novinky prirodzene dominujú hornej časti mriežky počas celého týždňa od vydania. Príznaky „Staff pick“ umožňujú manuálny editoriálny zásah a prepísanie poradia. Jitter zase zabezpečuje, že ponuka na webe pôsobí neustále živo a dynamicky.

## Periodický refresh job

Naplánovaná databázová úloha (cron job) beží niekoľkokrát denne. Kontroluje pomocnú jednoradovú tabuľku (guard table) a ak bol posledný refresh spustený len nedávno, proces sa preskočí. Každé spustenie:
1. Prepočíta skóre popularity pre každý publikovaný produkt.
2. Identifikuje najnovší vydaný drop a aplikuje naň bonifikáciu (boost).
3. Pregeneruje odporúčania súvisiacich produktov (related products) pre každý produkt.
4. Aktualizuje časovú pečiatku v pomocnej tabuľke (guard table).

## Aktualizácia na strane klienta

Klientsky web (storefront) v krátkych intervaloch dopytuje (polluje) backend a odoberá zmeny v katalógu v reálnom čase. Stav katalógu v operačnej pamäti (in-memory) rýchlo expiruje. Ak sa správa o zmene v reálnom čase stratí, nasledujúci dopyt alebo opätovná aktivácia okna prehliadača (focus) stiahne čerstvé dáta. Typická latencia je pod jednu sekundu, v najhoršom možnom prípade (fallback) sa dáta aktualizujú do minúty.

## Sekvenčné usporiadanie

Databázový spúšťač (trigger) reaguje na pridanie (insert) nového produktu a na jeho prvé zverejnenie. Priradí mu ďalšiu hodnotu zo sekvenčného počítadla, ktorá už zostáva produktu natrvalo. To zaručuje, že signál novosti sa nikdy nestratí – aj keď sa produkt archivuje a neskôr opäť publikuje, jeho pôvodné poradie sa zachová.

## Produktové obrázky

### Odstránenie pozadia

Všetky obrázky s orezaným pozadím (cutouts) začínajú ako neupravené (raw) fotografie. Pozadie sa odstraňuje prostredníctvom nástroja **PhotoRoom**, ktorý je momentálne pre tento workflow bezplatný. Orezaný produkt sa nahrá do administrácie ako súbor PNG s priehľadnosťou. V budúcnosti sa plánuje priame prepojenie cez API na PhotoRoom (alebo podobnú službu), čím sa tento krok plne automatizuje.

### Nahrávanie a optimalizácia

Po odstránení pozadia prechádzajú obrázky v administrácii trojkrokovým procesom:
1. **Požiadavka na nahrávanie (Upload URL)** – získanie zabezpečeného (podpísaného) URL z Edge funkcie pre médiá.
2. **Nahranie (Upload)** – priamy zápis (PUT) do objektového úložiska (object storage).
3. **Dokončenie (Finalizácia)** – Edge funkcia spracuje obrázok: zistí rozmery, alfa kanál (priehľadnosť), ohraničenie (bounding box) a optimalizuje súbor na rýchle doručenie používateľom.

Doručovanie obrázkov používateľom prebieha cez cachovaciu proxy službu, ktorá ku každému objektu v úložisku pridáva HTTP hlavičky pre dlhodobé a nemenné (immutable) ukladanie do vyrovnávacej pamäte. Obrázky sa nikdy neposkytujú priamo z pôvodného úložiska (raw storage endpoint).

### Úložisko (Storage)

Počas beta testovania sú obrázky uložené v **úložisku Supabase (kompatibilnom s S3)**. Pre produkčnú verziu plánujeme migráciu k špecializovanému poskytovateľovi objektového úložiska, ako je napríklad **Google Cloud Storage**, **Amazon S3** alebo **Yandex S3**, v závislosti od ceny a výkonu.

## Prihlásenie na odber newslettera

Verejný registračný formulár na odber newslettera odosiela údaje do príslušnej Edge funkcie. Tá overí formát e-mailovej adresy, skontroluje skryté ochranné pole (honeypot), uplatní limity na frekvenciu požiadaviek (rate limiting) podľa IP adresy a e-mailu, zahashuje IP adresu pre potreby bezpečnostného logu a zapíše odberateľa do zabezpečenej tabuľky. Tabuľka odberateľov neumožňuje verejný zápis – jediným vstupným bodom je práve spomínaná Edge funkcia.
