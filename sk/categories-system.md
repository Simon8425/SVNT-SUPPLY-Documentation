# Systém kategórií

## Úloha kategórií

SVNT SUPPLY využíva kategórie ako základnú štruktúru na prezeranie webu. Úvodná stránka (landing page) obsahuje kurátorovaný výber kategórií (tech, home, books, lifestyle, jewelry, clothing, cosmetics, gifts, pets, travel, beauty, skincare, fragrance, fashion, accessories, shoes, bags a zastrešujúcu kategóriu „all“). Kategórie sa negenerujú automaticky z produktových dát – ide o premyslenú, ručne spravovanú taxonómiu.

## Dátový model

Každá kategória obsahuje tieto polia:

| Pole | Účel |
|---|---|
| `id` | Krátky textový kľúč |
| `slug` | Unikátny identifikátor vhodný pre URL adresy |
| `label` | Názov kategórie zobrazovaný v UI |
| `image_url` | Obrázok na pozadí dlaždice (voliteľný, inak sa použije obrázok produktu) |
| `desktop_sort_order` | Poradie zobrazenia v mriežke na desktope |
| `mobile_sort_order` | Poradie zobrazenia v mobilnom zozname |

Kategórie existujú nezávisle od produktov. Každý produkt odkazuje na svoju primárnu kategóriu. Prepojovacia tabuľka (junction table) navyše podporuje priradenie do **viacerých kategórií súčasne (multi-category)**, takže jeden produkt sa môže zobrazovať vo viacerých kategóriách s vlastným poradím zoradenia.

## Zoradenie

Pre desktop a mobil sa používa **odlišné poradie zoradenia**. Je to zámerné – zatiaľ čo pre desktopovú mriežku je vhodné iné rozloženie, na mobile lepšie funguje klasický vertikálny zoznam. Administračné rozhranie (Admin UI) umožňuje nezávislé usporiadanie pomocou drag-and-drop pre oba typy zobrazení (viewporty) so živým náhľadom.

## Admin UI

Editor kategórií ponúka:

- **Náhľad pre desktop**: Responzívna mriežka kategórií, ktorá presne zodpovedá verejnej úvodnej stránke.
- **Náhľad pre mobil**: Rovnaké dlaždice zobrazené v makete (mockupe) iPhonu s naznačeným rozhraním e-shopu.
- **Drag-and-drop usporiadanie**: Ovládanie kurzorom (pointer events) na desktope a natívny HTML5 drag-and-drop na mobile.
- **Inline úprava názvov**: Stačí kliknúť na ikonu ceruzky na ľubovoľnej dlaždici, prepísať názov a kliknutím mimo poľa (blur) ho uložiť.
- **Nahrávanie obrázkov pre jednotlivé dlaždice**: Kliknutím na dlaždicu nahrajete alebo vymeníte jej obrázok na pozadí. Obrázky sa optimalizujú na strane servera cez zabezpečenú (podpísanú) Edge funkciu.
- **Hromadné ukladanie (Batch save)**: Všetky vykonané zmeny (názvy, poradie, obrázky) sa uložia naraz. Tlačidlom „Reset“ môžete kedykoľvek vrátiť späť posledný uložený stav.

## Verejné zobrazenie

Na úvodnej stránke sa kategórie zobrazujú ako mriežka dlaždíc (na desktope) alebo ako posúvateľný zoznam (na mobile). Každá dlaždica obsahuje názov kategórie a jej obrázok pozadia. Kliknutím na dlaždicu prejdete na stránku kolekcie, kde sa zobrazia produkty z danej kategórie vo filtrovanej mriežke (gride).

Zberná kategória „All“ (Všetko) existuje natrvalo – jej poradie môžete meniť, no nie je možné ju vymazať. Zobrazuje všetky publikované produkty bez ohľadu na ich zaradenie.

## Multi-category produkty

Produkt môže byť zaradený do viacerých kategórií naraz prostredníctvom prepojovacej tabuľky. Práve jedna kategória je označená ako „primárna“ a určuje kontext omrvinkovej navigácie (breadcrumbs). Ostatné kategórie sú sekundárne – produkt sa na týchto stránkach zobrazí, ale pre jeho hlavnú identifikáciu sa používa primárna kategória.

V editore produktov v administrácii sa kategórie vyberajú cez rozbaľovacie pole s možnosťou viacnásobného výberu (multi-select) a tlačidlom „Nastaviť ako primárnu“. Kategórie je možné jednoducho vyhľadávať a prepínať.

## Edge funkcia

Edge funkcia category-media spracováva nahrávanie obrázkov pre dlaždice kategórií. Funkcia prijme obrázok prostredníctvom podpísanej URL adresy, optimalizuje ho a vráti verejný odkaz na obrázok.

## Budúce smerovanie

- Pridanie popisov alebo krátkych sloganov (taglines) pre kategórie na stránkach kolekcií.
- Editoriálne úvody špecifické pre každú kategóriu (napríklad odsek „Prečo milujeme tech“).
- Dynamická správa a vytváranie nových kategórií priamo z administrácie (v súčasnosti sú kategórie definované ako pevný kurátorovaný set).
