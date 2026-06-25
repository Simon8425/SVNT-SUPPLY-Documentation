# Zoznam prianí (Wishlist) a zdieľanie

## Aktuálny stav

SVNT SUPPLY ponúka bočný panel (drawer) **Uložené položky** („wishlist“), ktorý ukladá ID produktov do `localStorage`. Používateľ môže:

- Ukladať produkty priamo z karty produktu alebo z jeho detailnej stránky.
- Prezerať uložené položky vo vysúvacom bočnom paneli (slide-out draweri) z pravej strany.
- Odstraňovať položky zo zoznamu jednotlivo.
- Zdieľať svoj uložený zoznam prostredníctvom vygenerovanej URL adresy.

### Architektúra zdieľania

Zdieľanie uložených zoznamov zabezpečujú dve Edge funkcie v Supabase, ktoré využívajú databázovú tabuľku `wishlist_shares`:

**Proces vytvorenia (Create flow):**
1. Používateľ klikne na tlačidlo „Zdieľať uložený zoznam“ v bočnom paneli.
2. Klientska aplikácia validuje položky (produkty musia v tej chvíli existovať v aktívnom katalógu).
3. Edge funkcia prijme ID produktov, zahashuje náhodne vygenerovaný token zdieľania (share token) a zapíše nový riadok do databázy.
4. Funkcia vráti token zdieľania a vygenerovanú URL adresu späť klientskej aplikácii.
5. Klientska aplikácia skopíruje odkaz do schránky (v prípade záložného riešenia (fallbacku) zobrazí textové pole pre manuálne skopírovanie).

**Proces načítania (Read flow):**
1. Príjemca otvorí zdieľaný odkaz.
2. Edge funkcia vyhľadá príslušný hash tokenu a vráti zoznam ID produktov.
3. Klientska aplikácia načíta údaje o produktoch z katalógu a zobrazí ich v režime len na čítanie (read-only mriežke).

### Bezpečnosť

- Tokeny zdieľania (share tokens) sú náhodne vygenerované reťazce, ktoré sa pred uložením zahashujú.
- Hash tokenu je jedinečný – pokus o uhádnutie hrubou silou (brute-force scanning) je nereálny.
- Ochrana proti zahlteniu (Rate limiting): Uplatňuje sa na základe hashovanej IP adresy odosielateľa požiadavky.
- Dáta sú nemenné (immutable) – raz vygenerovaný zoznam už nie je možné dodatočne upraviť.
- Vytvorenie a prezeranie zoznamov nevyžaduje prihlásenie (anonymné zdieľanie).
- Neukladajú sa žiadne osobné údaje (iba zoznam ID produktov a hashované tokeny).

### Limity

- Limitovaný minimálny a maximálny počet produktov v jednom zdieľaní (kontrolované na strane klienta aj servera).
- Limity frekvencie požiadaviek (rate limit) na jednu IP adresu sú konfigurovateľné priamo v Edge funkcii.
- Záznamy odkazujú na ID produktov ako na textové reťazce, nie cez cudzie kľúče (foreign keys) – systém je tak odolný voči zmazaniu produktov z katalógu.

---

## Budúcnosť: Verejné profily

Ďalším krokom v rozvoji systému wishlistov sú **verejné používateľské profily**. Namiesto jednorazových odkazov na zdieľanie získajú používatelia trvalé profilové stránky so svojimi vlastnými kurátorskými výbermi.

### Plánované funkcie

- **Vytvorenie profilu**: Používateľ sa zaregistruje a získa verejný profil na adrese `/@username`.
- **Permanentné ukladanie**: Položky sa budú synchronizovať priamo s backendom namiesto ukladania v `localStorage`. Používateľ o ne nepríde ani pri zmene zariadenia či prehliadača.
- **Viacero zoznamov súčasne**: Používatelia si môžu vytvárať vlastné pomenované kolekcie (napr. „Letný wishlist“, „Tipy na darčeky“, „Daily carry“).
- **Prehľadná profilová stránka**: Zobrazí používateľove zoznamy, jeho vkusový profil (obľúbené kategórie, značky) a krátke predstavenie (bio).
- **Systém sledovania (Follow)**: Možnosť sledovať profily iných kurátorov. Úvodná stránka by tak mohla zobrazovať sekciu „Populárne medzi ľuďmi, ktorých sledujete“.
- **Editoriálne výbery**: Profil administrátorského tímu bude mať prioritné postavenie – napríklad ako „The SVNT Edit“, čo bude kurátorovaný zoznam aktualizovaný pri každom novom drope.

### Čo zostane rovnaké

- Zdieľanie konkrétneho zoznamu prostredníctvom odkazu zostane zachované (súčasný systém zdieľania).
- Profily budú voliteľné – bočný panel s uloženými produktmi bude naďalej fungovať anonymne cez `localStorage`.
- Limit počtu položiek pre jedno zdieľanie zostáva v platnosti.

### Technické zváženia pre profily

- Nové databázové tabuľky: používateľské profily, uložené zoznamy, položky v zoznamoch, sledovania (follows).
- Integrácia prihlasovania (Auth): Využitie Supabase Auth s možnosťou prihlásenia cez e-mail/heslo alebo OAuth (Google, Apple atď.).
- Zabezpečenie na úrovni riadkov (RLS politiky): Používatelia môžu zapisovať a upravovať iba svoje vlastné zoznamy, no verejné profily budú dostupné na čítanie pre všetkých.
- Existujúca tabuľka zdieľania (shares) prejde do úzadia a profily sa stanú hlavným nástrojom na zdieľanie obsahu.

---

## Prečo to má pre biznis význam

Zdieľané wishlisty fungujú ako virálny nástroj na objavovanie e-shopu. Keď niekto zazdieľa výber „10 vecí, ktoré si ukladám na SVNT SUPPLY“, každý príjemca spozná našu značku, kurátorstvo a uvidí aj naše partnerské odkazy. Verejné profily dajú tomuto zdieľaniu trvalý charakter – influenceri, redaktori a aktívni používatelia získajú na platforme svoje stále zázemie. Ich sledovatelia sa stanú novými návštevníkmi webu a prekliky cez ich affiliate odkazy budú priamo generovať príjmy.
