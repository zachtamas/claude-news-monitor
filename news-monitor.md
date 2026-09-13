# Hírfigyelő agent – instrukciók

Ez a fájl egy Claude Code agent számára készült instrukció. A cél: naponta
átnézni a megadott magyar hírportálokat, és összefoglalni az aznap
megjelent, a megadott témákba tartozó fontosabb híreket, linkekkel ellátva.

## Figyelt weboldalak

- https://telex.hu
- https://444.hu
- https://hvg.hu
- https://portfolio.hu

> Megjegyzés: a prog.hu-t szándékosan nem tartalmazza a lista (JavaScript-
> védelem miatt egyszerű HTML-lekéréssel nem érhető el; ha mégis kellene,
> az RSS feedjét kellene használni, ha van neki).

> Megjegyzés: bizonyos futtatási környezetekben (pl. sandboxolt Claude Code
> munkameneteknél) a portfolio.hu (és a www.portfolio.hu, amire átirányít)
> domain szervezeti szintű hálózati egress-policy miatt egyáltalán nem
> érhető el (nem egy adott URL, hanem maga a domain van letiltva a
> proxy szintjén). Ilyenkor ezt a forrást ki kell hagyni az adott
> futásból, és a kimenet végén jelezni kell, hogy nem volt elérhető —
> ne próbáld megkerülni a tiltást. Ha a futtatási környezet (pl. egy
> felhőben futó Cloud Routine) másképp van konfigurálva és a domain
> mégis elérhető, természetesen normálisan fel kell dolgozni.

## Érdeklődési témák

1. **Belföldi politika**
2. **Tudomány és technológia**
3. **Sport**

Minden más témát (bulvár, gasztro, kultúra stb.) hagyj ki az
összefoglalóból, kivéve ha egy hír egyértelműen átfedésben van a fenti
három témával (pl. egy tech-cég körüli hazai politikai/szabályozási
hír, vagy egy sportesemény körüli politikai vonatkozás).

## Feladat menete

1. **Lekérdezés**: töltsd le mindegyik forrás főoldalát (vagy ha elérhető,
   RSS feedjét — ez stabilabb, mint a HTML-parsing).
2. **Szűrés**: az egyes oldalak rovatai alapján (pl. telex.hu:
   `/rovat/belfold`, `/rovat/techtud`, `/rovat/sport`; hvg.hu:
   `/itthon`, `/tudomany`, `/sport`; 444.hu: „POLITIKA", „TECH",
   „SPORT"; portfolio.hu: cikkek témája alapján) válaszd ki azokat a
   cikkeket, amik a három témába tartoznak, és amik **aznap** (vagy az
   előző futás óta) jelentek meg.
3. **Deduplikáció / kereszthivatkozás**: ha ugyanarról a hírről több
   forrás is ír, ne külön tételként listázd, hanem egyetlen tételként,
   és **mindegyik forrás linkjét** tüntesd fel mellette. A hírek
   azonosítása témák/szereplők/dátum alapján történjen (nem szükséges
   szó szerint egyező cím).
4. **Állapotkövetés**: tartsd nyilván egy helyi fájlban (pl.
   `seen_urls.json`) a már látott cikk-URL-eket, hogy a következő
   futáskor ne dolgozd fel újra ugyanazokat a híreket.
5. **Összefoglalás**: minden hírhez írj egy 1-2 mondatos, tényszerű
   összefoglalót (ne a cikk címét ismételd meg szó szerint).
6. **Kimenet**: strukturált lista, témák szerint csoportosítva
   (Belföldi politika / Tudomány és technológia / Sport), minden
   tételnél a forrás(ok) linkjével.

## Kimeneti formátum (példa)

```
## Belföldi politika

- **<rövid, tényszerű cím>** – <1-2 mondatos összefoglaló>
  [Telex](URL) · [HVG](URL)

## Tudomány és technológia

- **<rövid, tényszerű cím>** – <1-2 mondatos összefoglaló>
  [444](URL)

## Sport

- **<rövid, tényszerű cím>** – <1-2 mondatos összefoglaló>
  [Telex](URL)
```

## Futtatási gyakoriság

Javasolt: naponta egyszer, reggel (pl. cronjob vagy Claude Code
ütemezett feladat / Routine formájában). Minden futás csak az előző
futás óta megjelent cikkeket dolgozza fel (lásd: állapotkövetés).

Ütemezési lehetőségek:
- **Cloud Routine**: claude.ai/code/routines vagy Desktop app →
  Routines → New routine → Cloud. Ehhez szükséges egy git repó, ahol
  ez a fájl és az állapotfájl (`seen_urls.json`) tárolva van. Felhőben
  fut, a gép lehet kikapcsolva, és jóváhagyás nélkül lefut.
- **Helyi (Desktop local routine vagy cron)**: a saját gépeden fut,
  csak akkor, ha a gép be van kapcsolva. Cron esetén pl.:
  `0 7 * * * cd /projekt/utvonala && claude -p "kövesd a news-monitor.md instrukcióit" --dangerously-skip-permissions`

## Email küldés

A futás végén a napi összefoglalót el kell küldeni emailben a
felhasználónak.

**Címzett:** zachtamas@gmail.com

Ehhez:

1. **Gmail (vagy más email) connector** legyen csatlakoztatva a
   Claude Code-hoz (MCP-n keresztül) — ez adja meg, melyik fiókból
   megy ki a levél. Ha ez elérhető, a feladat végén a connectorral
   küldd el az összefoglalót a fenti "Címzett" mezőben megadott
   email-címre.
   - Tárgy: `Napi hírfigyelő – <dátum>`
   - Törzs: a fenti "Kimeneti formátum" szerinti, témák szerint
     csoportosított összefoglaló, linkekkel.
   - Ha egy forrás nem volt elérhető, azt is említsd meg a levél
     végén egy rövid sorban.
2. Ha nincs connector csatlakoztatva, és a felhasználó explicit
   engedélyezte, egy egyszerű script is használható (pl. Python
   `smtplib` + Gmail alkalmazásjelszó), amit ilyenkor a projektben
   kell tárolni és a feladat lépéseként meghívni.
3. **Ne küldj emailt**, ha nincs új hír az előző futás óta – ilyenkor
   elég egy log-bejegyzés, felesleges levéllel zaklatni a
   felhasználót.

## Egyéb megjegyzések

- A cikkek tartalmát csak összefoglalni szabad, nem szó szerint
  másolni (szerzői jogi okokból).
- Ha egy forrás nem elérhető (pl. hálózati hiba, JS-védelem), azt jelezd
  a kimenet végén egy rövid megjegyzésben, de ne állítsd le emiatt a
  többi forrás feldolgozását.
