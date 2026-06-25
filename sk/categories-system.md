# Systém kategórií

## Úloha kategórií

SVNT SUPPLY používa kategórie ako primárnu štruktúru prehliadania. Landing page obsahuje kurátorskú množinu kategórií (tech, home, books, lifestyle, jewelry, clothing, cosmetics, gifts, pets, travel, beauty, skincare, fragrance, fashion, accessories, shoes, bags a catch-all "all"). Nie sú generované z produktových dát — sú to zámerná, ručne udržiavaná taxonómia.

## Dátový model

Každá kategória má:

| Pole | Účel |
|---|---|
| `id` | Krátky textový kľúč |
| `slug` | URL-safe unikátny identifikátor |
| `label` | Zobrazovaný názov v UI |
| `image_url` | Pozadie dlaždice (voliteľné, fallback na produktový obrázok) |
| `desktop_sort_order` | Pozícia v desktopovej mriežke |
| `mobile_sort_order` | Pozícia v mobile zozname |

Kategórie existujú nezávisle od produktov. Produkt referencuje primárnu kategóriu a pomocná tabuľka podporuje **multi-category** priradenie (jeden produkt sa môže zobraziť vo viacerých kategóriách s vlastným poradím).

## Zoradenie

Desktop a mobil používajú **samostatné poradie**. Je to zámerné — desktopová mriežka profituje z iného rozloženia ako mobilný vertikálny zoznam. Admin UI umožňuje nezávislé drag-and-drop zoradenie pre každý viewport s live náhľadom.

## Admin UI

Editor kategórií poskytuje:

- **Desktop náhľad**: Responzívna mriežka kategórií zodpovedajúca verejnej landing page
- **Mobilný náhľad**: Rovnaké dlaždice vo vnútri iPhone mockupu s falošným storefront chrome
- **Drag-and-drop zoradenie**: Pointer eventy na desktope, natívny HTML5 drag na mobile
- **Inline úprava názvu**: Klik na ceruzku, písanie, blur uloží
- **Upload obrázka pre každú dlaždicu**: Klik na dlaždicu nahradí jej pozadie. Obrázky sa optimalizujú server-side cez podpísanú edge funkciu
- **Batch save**: Všetky zmeny (názvy, poradie, obrázky) sa ukladajú naraz. Tlačidlo "Reset" vráti posledný uložený stav

## Verejné zobrazenie

Na landing page sa kategórie zobrazujú ako mriežka dlaždíc (desktop) alebo skrolovateľný zoznam (mobil). Každá dlaždica ukazuje názov kategórie a obrázok. Kliknutie prejde na kolekciu, ktorá zobrazí produkty v danej kategórii vo filtrovanom gridu.

Kategória "All" existuje permanentne — dá sa zoradiť, ale nie zmazať. Zobrazuje všetky publikované produkty bez ohľadu na kategóriu.

## Multi-category produkty

Produkt môže patriť do viacerých kategórií cez pomocnú tabuľku. Jedna kategória je označená ako "primárna", ktorá určuje kontext breadcrumbu. Ostatné sú sekundárne — produkt sa zobrazí na týchto stránkach, ale primárna kategória sa používa pri označovaní.

V admin produktovom editore ide o multi-select dropdown s tlačidlom "Nastaviť ako primárnu". Kategórie sa dajú vyhľadávať a vyberať prepínaním.

## Edge funkcia

Category-media edge funkcia spracováva uploady obrázkov dlaždíc. Funkcia prijíma obrázok cez podpísané URL, optimalizuje ho a vracia verejnú URL.

## Budúce smery

- Popisy alebo taglines kategórií na stránkach kolekcií
- Kategórie-špecifické editoriálne úvody (odstavec "Prečo milujeme tech" pre každú kategóriu)
- Dynamické vytváranie kategórií z admina (aktuálne sú kategórie fixný kurátorský set)
