# Projekt: mitt investeringsverktyg

> Ursprunglig specifikation, sparad ordagrant 2026-11-03. Ändringar och beslut som fattas
> under arbetet noteras längst ned under "Beslutslogg" i stället för att skriva om texten.

Du är en erfaren utvecklare och en noggrann aktieanalytiker. Bygg ett lokalt investeringsverktyg som utgår från mina faktiska innehav och samlar allt väsentligt om dem på ett ställe: hur portföljen går, vilka risker jag tar, vad som händer i bolagen och vad jag behöver hålla koll på. Verktyget är beslutsstöd. Det presenterar fakta och analys, lägger aldrig order och ger inga köp- eller säljråd.

## Förutsättningar
- Mina innehav ligger på Nordnet. Exporten från "Transaktioner och notor" (CSV, hela historiken) ligger i projektmappen. Flytta den till data/import/. Den är facit.
- Just nu äger jag bland annat Hacksaw, Bonesupport, ServiceNow, Vitec och Kambi (kontrollera mot exporten). Vissa bolag rapporterar i en annan valuta än den aktien handlas i, så räkna nyckeltal konsekvent.
- Gränssnitt på svenska, SEK som basvaluta och svensk talformatering. Kod och filnamn på engelska.

## Arbetssätt
1. Läs först exportfilen. Ställ sedan högst 10 frågor om det du inte ser där: kontotyp (ISK, KF eller depå), mål och tidshorisont, månadssparande, riskvilja, egna regler (t.ex. max andel per bolag) och hur jag vill använda verktyget.
2. Föreslå arkitektur, datakällor och en plan i faser. Vänta på mitt OK innan du skriver kod.
3. Spara den här specifikationen som docs/SPEC.md och skapa en kort CLAUDE.md (hur man kör och testar, vilka regler som gäller) så att nya sessioner har sammanhanget.
4. Bygg en fas i taget. Efter varje fas: kör testerna, visa vad som fungerar och gör en commit.
5. Kontrollera aktuella fakta (skatteregler, API:er, användarvillkor) med webbsökning i stället för att lita på minnet, och notera källa och datum. Fråga hellre än gissa.

## 1. Grunden: innehav och transaktioner
- Läs Nordnets CSV robust (teckenkodning, avgränsare, decimalkomma, svenska kolumnnamn) och bygg en transaktionslogg: köp, sälj, utdelningar, insättningar, uttag, avgifter, valutaväxling och splittar.
- Räkna fram innehav, antal och GAV ur loggen. Använd ISIN som nyckel, mappat till ticker per datakälla.
- Håll isär konton (skatten skiljer sig) och låt mig lägga till transaktioner manuellt.
- Kontroll: värde och GAV ska stämma med det Nordnet visar. Jag kan skicka en skärmdump.

## 2. Allt väsentligt per innehav
- Kurs: aktuell, historik och 52-veckorsspann, omräknad till SEK med historiska valutakurser.
- Fundamenta 5–10 år: omsättning, rörelseresultat, vinst per aktie, marginaler, fritt kassaflöde, nettoskuld och tillväxt.
- Värdering: P/E, EV/EBIT, EV/S, P/FCF och direktavkastning, jämfört med bolagets egen historik och närmaste konkurrenter.
- Estimat: konsensus för i år och nästa år. Finns ingen gratis källa, leta upp senast publicerade konsensus i nyhetsflödet och ange källa och datum. Spara varje ögonblicksbild så att estimatrevisioner kan följas över tid.
- Kalender: rapportdatum, årsstämma, utdelning och kapitalmarknadsdagar.
- Nyheter och pressmeddelanden (IR-sidor, MFN, Cision, samt SEC EDGAR för amerikanska bolag).
- Insynshandel (FI:s register, SEC Form 4) och blankning (FI:s blankningsregister).

## 3. Portföljanalys
- Översikt: värde, utveckling i dag, i år och sedan köp, realiserat och orealiserat resultat, mottagna utdelningar.
- Avkastning: tidsviktad och pengaviktad (XIRR), jämfört med index (t.ex. OMXSGI och en global indexfond i SEK). Svara tydligt på: hade jag gjort bättre med en indexfond?
- Fördelning per bolag, sektor, land och valuta (både noteringsvaluta och var bolagen tjänar sina pengar), plus egna temataggar som iGaming, medtech och mjukvara.
- Risk: koncentration (största innehav, effektivt antal innehav), volatilitet, max drawdown, beta, korrelation mellan innehaven och stresstester (börsfall, starkare krona).
- Vad händer om: simulera ett köp eller en försäljning och se hur vikter, risk och exponering ändras.
- Kostnader: courtage och växlingsavgifter hittills och per år.
- Skatt: schablonskatt för ISK/KF enligt gällande regler, utländsk källskatt och, för depå, realiserade vinster och förluster (underlag för K4).
- Omviktning: avvikelse mot målvikter jag sätter och hur månadssparandet behöver fördelas för att närma sig dem.
- Scenarier: Monte Carlo-simulering av portföljvärdet framåt utifrån mitt sparande.

## 4. Claude-drivna analyser
Skapa skills i .claude/skills som jag kör som kommandon. Låt skript hämta data och räkna, och använd Claude för research och tolkning. Gör research per bolag parallellt i subagenter.
- /brief: vad som hänt i mina innehav sedan förra gången – kursrörelser, nyheter, insynsaffärer, blankning, kommande händelser och larm. Sparas som daterad rapport i reports/.
- /analys <bolag>: djupanalys med affärsmodell, senaste rapporten (utfall mot förväntan, guidning, ledningens kommentarer), estimat och revisioner, värdering, finansiell styrka, risker och möjligheter samt ett bull- och ett bear-case.
- /rapport <bolag>: före rapporten vad jag ska titta efter, efter rapporten utfall mot förväntan och vad som ändrats.
- /granskning: helhetsgranskning av portföljen – risk, koncentration, kostnader, avvikelser från mina mål och frågor jag bör ställa mig.
- /jamfor <bolag> <bolag>: två bolag sida vid sida.
Gäller alla analyser: källa och datum för varje siffra, tydlig skillnad mellan fakta och bedömning, "uppgift saknas" i stället för gissningar och inga köp- eller säljrekommendationer.

## 5. Bevakning och larm
- Investeringsjournal per innehav: min tes, vad som måste vara sant och vad som skulle få mig att sälja. Flagga efter varje rapport om något talar emot tesen.
- Larm med regler jag kan ändra: stor kursrörelse, rapport inom 7 dagar, insynsaffärer, kraftigt ökad blankning, innehav över min maxandel och sänkta estimat.
- Bevakningslista för bolag jag inte äger, med samma data och larm.

## 6. Gränssnitt
- Lokal webbapp med sidorna Översikt, Innehav, Bolag, Risk, Kalender, Utdelningar och skatt, Mål och scenarier, Bevakning och Rapporter.
- Överst en sammanfattning på 30 sekunder: vad som hänt och vad som kräver min uppmärksamhet.
- Visa alltid källa och när data senast uppdaterades, och varna för gammal data. Förklara hur varje nyckeltal räknas.
- Jag läser mycket i mobilen: gör /brief-rapporten mobilvänlig, och föreslå hur jag når den därifrån och hur den kan köras automatiskt varje vecka.
- Om mitt budgetverktyg finns i ~/budget: använd gärna samma designstil och föreslå om månadssparandet kan hämtas därifrån.

## 7. Teknik och principer
- Python (pandas) för data och beräkningar och SQLite som lokal databas och cache. Välj det enklaste gränssnittet som går att underhålla.
- Ett utbytbart lager per datakälla, så att en källa kan bytas om den slutar fungera. Utvärdera gratis källor först (t.ex. Yahoo Finance via yfinance, SEC EDGAR, Riksbankens API och FI:s register) och föreslå betaltjänster (t.ex. Börsdata) bara om de ger tydligt bättre data.
- Ett kommando som uppdaterar all data och visar vad som misslyckades. En källa som fallerar får inte stoppa resten.
- Respektera källornas villkor, cacha och begränsa anrop. Ingen inloggning mot Nordnet och inga sparade lösenord.
- Mina data stannar lokalt: data/, databasen och .env ligger i .gitignore.
- Tester för alla beräkningar (GAV, XIRR, tidsviktad avkastning, skatt, valutaomräkning) med kända exempel.
- README som visar hur jag startar, uppdaterar och lägger till transaktioner.

## Faser
1. Import, kurser, valutor och en översikt (värde, avkastning, fördelning) som stämmer mot Nordnet.
2. Data per bolag, kalender, nyheter, insynshandel och blankning.
3. Risk, indexjämförelse, kostnader och skatt.
4. Skills och rapportarkiv.
5. Journal, larm, bevakning, omviktning, scenarier och automatisk veckokörning.

---

## Beslutslogg

| Datum | Beslut |
|---|---|
| 2026-11-03 | `reports/` läggs också i .gitignore, eftersom rapporterna innehåller innehav och belopp. |
