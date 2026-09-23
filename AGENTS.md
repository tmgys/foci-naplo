# AGENTS.md — tudnivalók a projektről

Ez a fájl a coding agenteknek szól. Olvasd el, mielőtt bármit módosítasz.

> A repo **nyilvános**. Ebbe a fájlba és a kódba ne kerüljön személyes adat,
> név, e-mail-cím, kulcs vagy jelszó.

## Mi ez

Focimeccs-napló: egy gyerek meccseinek nyilvántartása. Felvitel, szerkesztés,
törlés, szezonösszesítő, két diagram, mentés fájlba és visszatöltés.

Élőben: https://tmgys.github.io/foci-naplo/ (GitHub Pages, a `main` ágról).
A tulajdonos telefonon használja, a kezdőképernyőre kitéve.

## Mi van a mappában

```
index.html    a teljes alkalmazás — HTML + CSS + JS egy fájlban (~700 sor)
.gitignore    kizárja a mentés- és exportfájlokat (foci-naplo-*.json / *.csv)
AGENTS.md     ez a fájl
```

Nincs build, nincs `package.json`, nincs függőség. A fájlt a böngésző
közvetlenül megnyitja. Egyetlen külső erőforrás a Google Fonts
(Archivo, IBM Plex Sans, IBM Plex Mono) — ha nem tölt be, van tartalék betűkészlet.

## Az index.html felépítése

- `<style>` — design tokenek a `:root`-ban, sötét mód a
  `@media (prefers-color-scheme: dark)` blokkban, utána a komponensek
- `<body>` — emlékeztető sáv, fejléc, összesítő, űrlap panel, diagramok,
  meccslista, Adatok sáv, törlés megerősítő ablak
- `<script>` — nyolc számozott szakasz:

| Szakasz | Miről szól |
|---|---|
| 1. Adattárolás | `betolt`, `ment`, segédfüggvények |
| 2. Diagramok | kézzel rajzolt SVG, külső könyvtár nélkül |
| 3. Meccslista és összesítő | `kirajzol` — innen megy minden újrarajzolás |
| 4. Műveletek | űrlap: `urlapNyit`, `szerkeszt`, `megse`, submit, `torol` |
| 5. Mentés, visszatöltés, CSV | `mentesFajlba`, `csvExport`, fájl beolvasás |
| 6. Mentési emlékeztető | `emlekeztetoFrissit` és a hozzá tartozó állapot |
| 7. Törlés megerősítése | `torlesKerdes`, `torlesMegse`, `torlesMegerosit` |
| 8. Indulás | betöltés és első kirajzolás |

## Adatok

Minden a böngésző `localStorage`-ában, két kulcs alatt. Szerver nincs.
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
  `emlekeztetoElhalasztva`. Ne állítgass felületi elemeket szétszórva.

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

## Munkamenet

1. **Előbb terv, kódolás nélkül.** Írd le a feltételezéseidet is, ott ahol
   a kérés nem volt egyértelmű. Várj jóváhagyásra.
2. **Külön feature ágon dolgozz**, ne a `main`-en. Egy ág, egy téma.
3. Kódolás után ellenőrizd: JS szintaxis, valósághű mennyiségű próbaadat,
   **világos + sötét + telefon** nézet, és a hibás utak is (Mégse,
   félbehagyás, rossz fájl).
4. **Ne commitolj.** A commitot, merge-öt és pusht a tulajdonos végzi a
   Terminálból és a VS Code-ból.
5. A munka végén sorold fel, milyen döntéseket hoztál, amik nem voltak
   a tervben.

## Hogyan kommunikálj

A projekt tulajdonosa IT projektmenedzser, nem fejlesztő. Magyarul,
egyszerű nyelven írj, fejlesztői zsargon nélkül. Új fogalmat egy-két
mondatban magyarázz el. Ne adj tíz lépést egyszerre.
