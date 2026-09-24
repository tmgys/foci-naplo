# AGENTS.md — tudnivalók a projektről

Ez a fájl a coding agenteknek szól. Olvasd el, mielőtt bármit módosítasz.

> A repo **nyilvános**. Ebbe a fájlba és a kódba ne kerüljön személyes adat,
> név, e-mail-cím, kulcs vagy jelszó.

## Mi ez

Focimeccs-napló: egy gyerek meccseinek nyilvántartása. Felvitel, szerkesztés,
törlés, szezonválasztó, korosztály, szezonösszesítő, két diagram, mentés fájlba és visszatöltés.

Élőben: https://tmgys.github.io/foci-naplo/ (GitHub Pages, a `main` ágról).
A tulajdonos telefonon használja, a kezdőképernyőre kitéve.

## Mi van a mappában

```
index.html    a teljes alkalmazás — HTML + CSS + JS egy fájlban (~900 sor)
.gitignore    kizárja a mentés- és exportfájlokat (foci-naplo-*.json / *.csv)
AGENTS.md     ez a fájl
```

Nincs build, nincs `package.json`, nincs függőség. A fájlt a böngésző
közvetlenül megnyitja. Egyetlen külső erőforrás a Google Fonts
(Archivo, IBM Plex Sans, IBM Plex Mono) — ha nem tölt be, van tartalék betűkészlet.

## Az index.html felépítése

- `<style>` — design tokenek a `:root`-ban, sötét mód a
  `@media (prefers-color-scheme: dark)` blokkban, utána a komponensek
- `<body>` — emlékeztető sáv, fejléc (benne a szezonválasztó és a korosztály), összesítő, űrlap panel, diagramok,
  meccslista, Adatok sáv, törlés megerősítő ablak, korosztály-kérdés ablak
- `<script>` — tíz számozott szakasz:

| Szakasz | Miről szól |
|---|---|
| 1. Adattárolás | `betolt`, `ment`, segédfüggvények |
| 2. Diagramok | kézzel rajzolt SVG, külső könyvtár nélkül |
| 3. Meccslista és összesítő | `kirajzol` — innen megy minden újrarajzolás |
| 4. Műveletek | űrlap: `urlapNyit`, `szerkeszt`, `megse`, submit, `torol` |
| 5. Mentés, visszatöltés, CSV | `mentesFajlba`, `csvExport`, fájl beolvasás |
| 6. Mentési emlékeztető | `emlekeztetoFrissit` és a hozzá tartozó állapot |
| 7. Törlés megerősítése | `torlesKerdes`, `torlesMegse`, `torlesMegerosit` |
| 8. Szezon | `szezonDatumbol`, `aktualisSzezon`, `szezonMeccsei`, `szezonValasztoFrissit`, `szezonValt` |
| 9. Korosztály | `korosztaly`, `korosztalyFrissit`, `korKerdesNyit`, `korKerdesBezar`, beállítás betöltés/mentés |
| 10. Indulás | betöltés, első kirajzolás, korosztály-kérdés ha nincs születési év |

## Adatok

Minden a böngésző `localStorage`-ában, három kulcs alatt. Szerver nincs.
Az adat eszközönként külön él (telefon és gép nem látja egymásét).

`foci-naplo` — a meccsek tömbje:

```js
{ id, datum, tipus, ellenfel, helyszin, sajatGol, ellenfelGol,
  gol, assziszt, percek, ertekeles, jegyzet }
```

- `id`: `Date.now()` alapú szám
- `datum`: `"ÉÉÉÉ-HH-NN"`, szövegként rendezhető
- `tipus`: Bajnoki / Kupa / Torna / Edzőmeccs / Egyéb (zárt lista)
- `ertekeles`: `"1"`–`"5"` szövegként, vagy `""` ha nincs
- `gol`, `assziszt`, `percek`: a játékos saját adatai
- `sajatGol`, `ellenfelGol`: a csapat eredménye

`foci-naplo-meta` — `{ utolsoMentes, utolsoValtozas }`, időbélyegek.
Mentésnek a **fájlba mentés** és a **visszatöltés** számít, a CSV export nem.

`foci-naplo-beallitas` — `{ szuletesiEv }`, egész szám 1990 és a mostani év
között. **Személyes adat:** csak itt él. Nem kerül a kódba, a mentésfájlba,
a CSV-be, és a visszatöltés nem nyúl hozzá. Eszközönként egyszer kell megadni.

**Szezon:** nincs eltárolva, a `datum`-ból számoljuk. Egy szezon július 1-jétől
június 30-ig tart (a júliusi felkészülés már az új szezon). Jelölése a kezdő év:
`2026` = „2026/27". Választható: amelyikben van meccs, plusz mindig az aktuális.
Nyitáskor a mai dátum szerinti szezon látszik. Ha egy felvitt vagy szerkesztett
meccs más szezonba esik, az app átvált arra.

**Korosztály:** nincs eltárolva. A kiválasztott szezon záró éve − születési év
(2017-es gyerek, 2025/26 → U9, 2026/27 → U10), tehát a szezonnal együtt vált.
U21 fölött „Felnőtt", U6 alatt is szám (U5…), a születés előtti szezonnál semmi.
Ha nincs megadva, induláskor rákérdez (kihagyható „Később"-vel); a fejlécben
„módosít" linkkel javítható.

Régi adatokban hiányozhatnak mezők — mindig legyen tartalék érték
(`m.tipus || "–"`).

## Szabályok, amikhez tartsd magad

**Stílus**
- Minden azonosító, felirat és komment **magyarul**.
- A felhasználó felé mutatott szöveg mondja meg, mi fog történni
  („Törlés", ne „OK").

**Szerkezet**
- Egy fájl marad. Ne bontsd szét, ne vezess be build lépést.
- **Ne tegyél be külső könyvtárat** (diagramhoz sem). A diagramok kézzel
  rajzolt SVG-k, és annak is kell maradniuk.
- **Egy hely dönt egy dologról**, a felület ebből következik.
  Ilyen állapotok: `szerkesztettId`, `urlapNyitva`, `torlendoId`,
  `emlekeztetoElhalasztva`, `valasztottSzezon`, `szuletesiEv`, `korKerdesNyitva`.
  Ne állítgass felületi elemeket szétszórva.
- **Új ablak a meglévő minta szerint** (`.modalHatter` + `.modal`, mint a törlés
  és a korosztály ablaka): Esc és háttérre kattintás = a biztonságos kilépés,
  a fókusz a biztonságos gombon vagy a beviteli mezőn indul.
- **Szűrő soha nem hat az adatmozgatásra.** A szezon (és bármilyen jövőbeli
  szűrő) csak azt szűri, ami látszik: lista, összesítő, diagramok. A fájlba
  mentés, a CSV export, a visszatöltés és a mentési emlékeztető mindig az
  összes meccsel (`meccsek`) dolgozik. Különben egy mentésből csendben
  kimaradnának a többi szezon meccsei.

**Kinézet**
- Színt csak tokenből (`var(--ink)`, `var(--accent)`, …). **Beégetett
  hexa érték tilos** a komponensekben — sötét módban olvashatatlan lesz.
- Új szín = új token a `:root`-ban ÉS a sötét blokkban.
- Telefonra is jónak kell lennie (390 px), és nem lehet vízszintes görgetés.
- Visszafogott formázás, ne legyen „AI-szagú”: ne KPI-bannerek, ne fölösleges
  díszítés.
- Diagramnál a szín soha ne egyedül hordozza az információt, és legyen
  színtévesztőknek is olvasható. Egy diagram = egy skála, soha ne kettő.
- Számoknál `font-variant-numeric: tabular-nums`.

**Biztonság, adat**
- Romboló művelet (törlés, felülírás) előtt: **ellenőrzés → megerősítés →
  művelet**. Hibás bemenetnél a meglévő adathoz ne nyúlj, és mondd meg a
  felhasználónak, hogy érintetlen maradt.
- A megerősítő ablak írja ki, **mire** vonatkozik — ne csak annyit, hogy „Biztos?”.
- `localStorage` írás/olvasás mindig `try/catch`-ben.
- Felhasználói szöveg megjelenítése előtt `escape_()`.
- **Személyes adat** (pl. a születési év) csak a böngészőben él. Nem kerülhet
  a kódba (a repo nyilvános), és a mentésfájlba vagy a CSV-be sem, hacsak a
  tulajdonos kifejezetten nem kéri. Ha egy új funkció új személyes adatot hoz,
  **kérdezz rá**, hol tárolódjon.

## Munkamenet

1. **Ha a kérés nem egyértelmű, előbb kérdezz — egyesével**, mindegyikhez
   2–3 lehetőséggel és a javaslatoddal. Ez bevált: rejtett döntéseket hoz
   elő (pl. hova esik a júliusi meccs).
2. **Utána terv, kódolás nélkül.** Írd le a feltételezéseidet is, ott ahol
   a kérés nem volt egyértelmű. Várj jóváhagyásra.
3. **Külön feature ágon dolgozz**, ne a `main`-en. Egy ág, egy téma.
   Kódolás előtt ellenőrizd, hogy tényleg azon az ágon vagy
   (`git branch --show-current`); ha nem, szólj, és várj.
4. Kódolás után ellenőrizd: JS szintaxis, valósághű mennyiségű próbaadat,
   **világos + sötét + telefon** nézet, és a hibás utak is (Mégse,
   félbehagyás, rossz fájl, hibás bemenet), valamint a határnapok.
   **Futtasd újra a korábbi funkciók ellenőrzését is.** Ha egy meglévő
   tesztet módosítanod kell, mondd meg, melyiket és miért.
5. **Ne commitolj.** A commitot, merge-öt és pusht a tulajdonos végzi a
   Terminálból és a VS Code-ból. A sorrend: kipróbálás → diff → commit →
   merge → push. Emlékeztesd rá, ha kimaradna a diff.
6. A munka végén sorold fel, milyen döntéseket hoztál, amik nem voltak
   a tervben.

**Tesztelési tudnivalók**
- Helyben (`file://`) megnyitva a napló üres: a böngésző címenként külön
  tárol. Próbaadatot a `localStorage`-ba kell tölteni, vagy egy mentésből
  visszatölteni.
- Ha nincs `foci-naplo-beallitas`, induláskor feljön a korosztály-kérdés, és
  eltakarja a gombokat. Automata tesztnél töltsd be előre
  (`{"szuletesiEv":2017}`), kivéve ha épp ezt az ablakot teszteled.
- A szezon és a korosztály a mai dátumtól függ: teszthez rögzítsd az időt.

## Hogyan kommunikálj

A projekt tulajdonosa IT projektmenedzser, nem fejlesztő. Magyarul,
egyszerű nyelven írj, fejlesztői zsargon nélkül. Új fogalmat egy-két
mondatban magyarázz el. Ne adj tíz lépést egyszerre.
