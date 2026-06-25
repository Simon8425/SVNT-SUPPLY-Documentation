# Zobrazenie produktov a obnovovanie

## Mriežka na landing page

Landing page zobrazuje publikované produkty v jednej mriežke. Predvolené zoradenie je **Featured** (označené ako *Editor's choice*).

Poradie Featured riadi server-side `popularity` skóre, ktoré sa pravidelne prepočítava pre každý publikovaný produkt. Skóre kombinuje päť signálov:

| Signál | Váha | Zdroj |
|---|---|---|
| **Novosť** | ~35% | Sekvenčný counter priradený pri prvom publikovaní. Vyšší = novší. |
| **Staff pick** | ~20% | Manuálny flag nastavený v admin produktovom editore. |
| **Freshness** | ~10% | Časový pokles od vydania alebo publikovania. Novšie položky majú vyššie skóre. |
| **Najnovší drop** | ~15% | Produkt patrí do najnovšieho vydaného Sunday Dropu. |
| **Period jitter** | ~20% | Deterministický hash produktového ID + aktuálne časové okno. Zabezpečuje, že sa poradie mení bez toho, aby bolo náhodné. |

Novosť + najnovší drop = približne polovica skóre, takže produkty z najnovšieho dropu dominujú vrchnej časti mriežky počas svojho týždňa. Staff picky poskytujú editoriálny override. Jitter udržuje povrch živý.

## Periodický refresh job

Plánovaná databázová úloha beží niekoľkokrát denne. Kontroluje single-row guard tabuľku a preskočí, ak posledný refresh bol príliš nedávno. Každý beh:
1. Prepočíta popularitu pre každý publikovaný produkt
2. Identifikuje najnovší vydaný drop a aplikuje boost
3. Prebuduje súvisiace produkty pre každý produkt
4. Aktualizuje časovú značku guard tabuľky

## Freshness na strane klienta

Storefront pravidelne pollinguje backend a odoberá real-time zmeny katalógu. In-memory stav katalógu rýchlo expiruje. Ak sa real-time udalosť stratí, ďalší poll alebo focus okna stiahne čerstvé dáta. Typická latencia je pod sekundu; najhorší fallback je pod minútu.

## Sekvenčné zoradenie

Databázový trigger reaguje na insert produktu a na prvé publikovanie. Priradí ďalšiu hodnotu sekvenčného countera, ktorá zostáva trvalo. To zaručuje, že signál novosti sa nikdy nestratí — aj keď sa produkt archivuje a znovu publikuje, jeho pôvodné poradie zostáva.

## Produktové obrázky

### Odstránenie pozadia

Všetky produktové cutout obrázky začínajú ako raw fotografie. Pozadie sa odstraňuje pomocou **PhotoRoom**, ktoré je v súčasnosti pre tento workflow zadarmo. Cutout sa nahrá do admina ako PNG s transparentnosťou. V budúcnosti sa plánuje API pripojenie k PhotoRoom (alebo podobnej službe) na automatizáciu tohto kroku.

### Upload a optimalizácia

Po odstránení pozadia prechádzajú obrázky trojkrokovým admin workflow:
1. **Vyžiadanie upload URL** — podpísané URL z media edge funkcie
2. **Upload** — priamy PUT do object storage
3. **Finalizácia** — edge funkcia spracuje obrázok: zistí rozmery, alfa kanál, bounding box a optimalizuje súbor na doručenie

Verejná dodávka prebieha cez cachovací proxy, ktorý pridáva dlhodobé immutable cache hlavičky pre každý storage objekt. Obrázky sa nikdy neservujú priamo z raw storage endpointu.

### Storage

Počas bety sú obrázky uložené v **Supabase S3-compatible storage bucket**. Pre produkciu je plán migrovať na dedikovaného object storage providera, ako napríklad **Google Cloud Storage**, **Amazon S3** alebo **Yandex S3**, v závislosti od ceny a výkonu.

## Newsletter signup

Verejný signup formulár posiela dáta do newsletter edge funkcie. Funkcia overí formát emailu, skontroluje honeypot pole, vynúti rate limiting podľa IP a emailu, hashuje IP pre audit log a vloží odberateľa do zamknutej tabuľky. Tabuľka odberateľov nemá verejný write prístup — jediný vstupný bod je edge funkcia.
