# CLAUDE.md

Lokalt investeringsverktyg byggt på användarens Nordnet-innehav. Beslutsstöd: fakta och analys,
aldrig order och aldrig köp- eller säljråd. Hela specifikationen finns i `docs/SPEC.md`.

## Status
Fas 0: arkitektur och plan väntar på användarens OK. Ingen kod är skriven än.
Byggs fas för fas enligt `docs/SPEC.md`. Efter varje fas körs testerna, resultatet visas och en commit görs.

## Köra och testa
Fylls i under fas 1. Planerat:
- `uv sync`: installera beroenden
- `uv run pytest`: kör alla tester
- `uv run invest import data/import/<fil>.csv`: läs in Nordnet-exporten
- `uv run invest update`: hämta all data och visa vilka källor som misslyckades
- `uv run invest serve`: starta webbappen lokalt

## Regler
- **Inga råd och inga order.** Ingen inloggning mot Nordnet och inga sparade lösenord.
- **Användarens data stannar lokalt.** `data/`, databasen, `.env` och `reports/` får aldrig hamna i git.
  Kontrollera `git status` före varje commit.
- **Svenska i gränssnittet, engelska i koden.** SEK som basvaluta och svensk talformatering
  (1 234 567,89 kr, −3,2 %). Kod, filnamn och kommentarer skrivs på engelska.
- **ISIN är nyckeln.** Tickers och andra id:n mappas per datakälla.
- **Konton hålls isär** (ISK, KF och depå beskattas olika).
- **Nyckeltal räknas i en och samma valuta.** Värdering räknas i bolagets rapportvaluta,
  så börsvärdet räknas om till den innan P/E, EV/EBIT med flera. Belopp visas i SEK.
- **Varje siffra har källa och datum.** Skriv "uppgift saknas" i stället för att gissa.
  Håll isär fakta och bedömning. Varna för gammal data.
- **Varje datakälla ligger bakom ett utbytbart adapterlager.** En källa som fallerar får inte stoppa
  de andra. Cacha, begränsa anrop och följ källans villkor.
- **Aktuella fakta kontrolleras med webbsökning**, inte ur minnet: skatteregler, API:er och villkor.
  Notera källa och datum. Fråga hellre än gissa.
- **Varje beräkning har tester med kända exempel**: GAV, XIRR, tidsviktad avkastning, skatt
  och valutaomräkning.
