# Wishlist a zdieľanie

## Aktuálny stav

SVNT SUPPLY má drawer **Uložené položky** ("wishlist"), ktorý uchováva ID produktov v `localStorage`. Používateľ môže:

- Ukladať produkty z produktovej karty alebo detailu
- Prezerať uložené položky v slide-out draweru z pravého okraja
- Odstraňovať položky individuálne
- Zdieľať svoj uložený zoznam ako URL

### Architektúra zdieľania

Uložené zoznamy sa zdieľajú cez dve Supabase edge funkcie podporené tabuľkou `wishlist_shares`:

**Create flow:**
1. Používateľ klikne "Zdieľať uložený zoznam" v draweri
2. Klient validuje položky (produkty musia existovať v live katalógu)
3. Edge funkcia prijíma ID produktov, hashuje náhodný share token a uloží riadok
4. Vráti share token + URL klientovi
5. Klient skopíruje URL do schránky (alebo fallback s viditeľným textovým inputom + manuálne kopírovanie)

**Read flow:**
1. Niekto otvorí zdieľaný odkaz
2. Edge funkcia vyhľadá hash tokenu a vráti ID produktov
3. Klient načíta dáta produktov z katalógu a zobrazí read-only grid

### Bezpečnosť

- Share tokeny sú náhodné reťazce, hashované pred uložením
- Hash tokenu je unikátny — brute-force scanning je nepraktický
- Rate limiting: IP-hashovaný create log, vynucovaný per IP
- Záznamy sú immutable — raz vytvorený zoznam nie je možné upraviť
- Na vytvorenie ani čítanie nie je potrebná autentifikácia (anonymné zdieľanie)
- Nie sú uchovávané žiadne osobné údaje (len ID produktov a hashované tokeny)

### Limity

- Minimálny a maximálny počet produktov na zdieľanie (vynucované klientom aj serverom)
- Rate limit per IP (konfigurovateľné v edge funkcii)
- Záznamy referencujú ID produktov ako reťazce, nie foreign keys (tolerancia voči zmazaným produktom)

---

## Budúcnosť: Verejné profily

Ďalšou evolúciou wishlist systému sú **verejné používateľské profily**. Namiesto jednorazových zdieľacích odkazov budú mať používatelia trvalé profilové stránky so svojimi kurátorskými výbermi.

### Plánované funkcie

- **Vytvorenie profilu**: Používateľ sa zaregistruje a získa verejný profil na `/@username`
- **Trvalé uložené zoznamy**: Položky sa synchronizujú do backendu namiesto localStorage. Prežijú zmenu prehliadača.
- **Viacero zoznamov**: "Letný wishlist", "Tipy na darčeky", "Daily carry" — používatelia si vytvoria pomenované kolekcie
- **Profilová stránka**: Zobrazuje používateľove zoznamy, jeho vkusový profil (obľúbené kategórie, značky) a krátky bio
- **Follow systém**: Používatelia môžu sledovať profily iných kurátorov. Landing page by mohla zobrazovať "Trending medzi ľuďmi, ktorých sleduješ."
- **Editoriálne výbery**: Profil admin tímu bude first-class citizen — "The SVNT Edit" ako kurátorský zoznam aktualizovaný s každým dropom

### Čo zostáva rovnaké

- Zdieľanie jedného zoznamu cez odkaz naďalej funguje (aktuálny share systém)
- Profily sú voliteľné — saved drawer stále funguje anonymne cez localStorage
- Limit počtu položiek na jedno zdieľanie zostáva

### Technické zváženia pre profily

- Nové tabuľky: používateľské profily, uložené zoznamy, položky zoznamov, follows
- Auth integrácia: Supabase Auth s email/heslo alebo OAuth
- RLS politiky: Používatelia môžu čítať/písať len vlastné zoznamy; verejné profily sú čitateľné pre všetkých
- Existujúca share tabuľka sa stáva sekundárnou funkciou — profily budú primárnym mechanizmom zdieľania

---

## Prečo to má pre biznis význam

Zdieľané wishlisty sú vírusové objavovanie. Keď niekto zdieľa "10 vecí, ktoré si ukladám na SVNT SUPPLY," každý príjemca vidí značku, kurátorstvo aj partnerské odkazy. Verejné profily to robia trvalým — influenceri, editori a power používatelia získajú domov na platforme. Ich followeri sa stávajú návštevníkmi SVNT SUPPLY. Ich affiliate odkazy generujú príjmy.
