# Hírfigyelő

Ez a repó egy Claude Code agent számára készült automatizált hírfigyelő
feladatot tárol. A program naponta átnézi a megadott magyar hírportálokat,
kiszűri a beállított témákba tartozó, aznap megjelent fontosabb híreket, és
egy strukturált, forráslinkekkel ellátott összefoglalót küld el emailben.

## Mit csinál pontosan

1. **Lekérdezi** a figyelt hírportálokat (RSS feed vagy HTML alapján).
2. **Szűri** a cikkeket a megadott rovatok/témák alapján, csak azokat tartja
   meg, amik a beállított érdeklődési témákba tartoznak.
3. **Dedupikál**: ha ugyanarról a hírről több forrás is ír, egyetlen
   tételként listázza, minden érintett forrás linkjével.
4. **Állapotot követ** a `seen_urls.json` fájlban, hogy egy adott cikket ne
   dolgozzon fel kétszer, és mindig csak az előző futás óta megjelent
   újdonságokat vegye be az összefoglalóba.
5. **Összefoglal**: minden hírhez rövid, tényszerű, 1-2 mondatos leírást ír
   (nem a cikk címét ismétli meg, és nem másolja szó szerint a tartalmat).
6. **Emailt küld** a kész, témák szerint csoportosított összefoglalóval a
   beállított címzettnek — de csak akkor, ha volt új hír az előző futás óta.

## Figyelt témák

- Belföldi politika
- Tudomány és technológia
- Sport

## Figyelt hírportálok

- telex.hu
- 444.hu
- hvg.hu

(A prog.hu és a portfolio.hu szándékosan nincs a listán — az előbbi
JavaScript-védelem, az utóbbi bizonyos futtatási környezetekben fennálló
hálózati korlátozás miatt nem érhető el egyszerű lekéréssel; részletek a
`news-monitor.md`-ben.)

## Fájlok

- **`news-monitor.md`** – a tényleges, részletes agent-instrukció: figyelt
  oldalak, témák, feldolgozási lépések, kimeneti formátum, email küldési
  szabályok és a git-munkafolyamat leírása.
- **`seen_urls.json`** – állapotfájl, a már feldolgozott cikkek URL-jeit és
  az utolsó futás adatait tárolja.

## Futtatás

A feladatot egy Claude Code agent futtatja a `news-monitor.md`
instrukciói alapján, naponta egyszer (pl. Claude Code Cloud Routine vagy
helyi cronjob formájában). Részletek: lásd `news-monitor.md` →
„Futtatási gyakoriság” szakasz.
