# Förlanseringsgranskning – Fantasy Rinken → fantasyrinken.se

Granskad version: **2.1.3** (branch `pre-launch`, utgångsläge `main` a4d01c0). Datum: 2026-09-29.
Radnummer avser källfilen `kod/Fantasy-Rinken.html` (sajtens `index.html` byggs från den med `bygg.py`), om inget annat anges.

**Så granskades det**
- Kodgenomgång (grep och läsning) av `index.html`, `sw.js`, `manifest.webmanifest`, `statnet.yml`, `statnet_sync.py`, `bygg.py`, `bygg.json`, `grafik_in.py`.
- Webbläsartest i Chromium (Claude-webbläsaren) mot en lokal server med `site/`: hela flödet med riktig data, konsolfel, fel- och tomlägen via en egen testserver som svarar med 500 / tomt / ogiltig JSON / 30 s fördröjning / nedkopplad, blockerad `localStorage`/`sessionStorage`, bredderna 360/390/768/1280 px, mätning av klickytor och textstorlek.
- Node: tidszonsbeteende. curl: CORS-headers mot fantasy.shl.se, github.io och GoatCounter; externa länkar.
- Python: sökning efter hemligheter i alla 134 publicerade filer i `versioner/*/webb`, `webb-full` och `site/`, samt git-historiken i det nya repot.

**Kunde inte verifieras här:** riktiga Safari/Firefox/Edge (bara kodgranskning), riktiga telefoner, Cloudflares faktiska headers och deploy, git-historiken i dina befintliga GitHub-repon (jag har ingen åtkomst), Lighthouse (bedömt utifrån filstorlekar och laddningsordning).

**Obs:** min CORS-kontroll registrerade en testvisning ("test-cors") i GoatCounter. Den kan tas bort där.

---

## Blockerar lansering

| # | Område | Fil / rad | Fynd | Åtgärd |
|---|---|---|---|---|
| B1 | A | `kod/bygg.json:2`; genererat i `index.html` (canonical, og:url, og:image, twitter:image, JSON-LD `url`/`image`, `FSL_PUBLIC_URL`), `sitemap.xml`, `llms.txt` | Alla absoluta adresser pekar på `https://fantasyshlgameweek.github.io/Verken-Ligan/`. Canonical till fel domän gör att Google indexerar den gamla adressen. | `"url": "https://fantasyrinken.se/"` i `bygg.json`. Allt genereras om. |
| B2 | A | `.github/workflows/statnet.yml` (committar `data/statnet.json` var 10:e min); `kod/Fantasy-Rinken.html:3089` (`SN_URL`) | Med Cloudflare Pages kopplat till repot blir varje datacommit en ny deploy: cirka 4 300/månad, mot gratisnivåns 500. Byggen skulle ta slut efter några dagar. | Sajten läser Statnet från `raw.githubusercontent.com/<konto>/<repo>/main/data/statnet.json` (`FR_SN_URL`, ny `bygg.json "statnet_url"`). I Cloudflare exkluderas `data/**` i *Build watch paths*. |
| B3 | E | saknas | Ingen integritetspolicy och inga kontaktuppgifter. Sajten skickar lagnamn och liga till GoatCounter (tredje part) och sparar uppgifter i webbläsaren; det måste beskrivas. | Ny `integritet.html` (vad som sparas, var, hur man raderar, GoatCounter, datakällor, kontakt). Länk "Integritet" i sidfoten. Kontakt `kontakt@fantasyrinken.se` – **fungerar först när du satt upp Cloudflare Email Routing** (står i checklistan). |
| B4 | B | `:2006, 2046, 2047, 2048, 2290, 2408, 3019, 3223, 3267, 3302` | Datum och tider formateras utan tidszon. Testat i node: en match 19:00 svensk tid visas som 13:00 i New York och 00:00 i Bangkok. | En gemensam hjälpare med `timeZone: 'Europe/Stockholm'` för alla visningar. |

## Bör fixas

| # | Område | Fil / rad | Fynd | Åtgärd |
|---|---|---|---|---|
| S1 | A | `:3094` | Den lokala filen (`app/…html`, `file://`) hämtar Statnet via `FSL_PUBLIC_URL + data/statnet.json` och ignorerar `FR_SN_URL`. Efter flytten ligger den färska datan på GitHub raw. | Använd `FR_SN_URL` först, även från `file://`. |
| S2 | B | `:1779–1785` (`forgetMe`) | **Glöm mig** tar bara bort `fsl-*`/`fsl1:*`. `fr-season:*` (alla lags kort), `fr-rv-n`, `fr-schema*` och `sessionStorage` `fr-stat:*` blir kvar – motsäger integritetstexten "Glöm mig raderar allt". | Ta även bort `fr-*` i `localStorage` och `sessionStorage` (i try/catch). |
| S3 | B | `:2028` → `:2220` | Ett svar som inte är JSON (t.ex. en felsida med status 200) tolkas som tom lista och ger **"Ingen omgång är låst ännu"** – missvisande. Testat. | Behandla ogiltig JSON som fel: "Fantasy SHL svarade med något oväntat. Försök igen om en stund." |
| S4 | B | `:2178, 2183, 1856, 2248` | Felmeddelanden: vid nedkoppling/timeout nämns `FantasyRinken.exe` även på webben, timeout (20 s) skiljs inte från "ingen anslutning", och 500 visas som "Oväntat svar (500)". Testat: alla lägen visar text, inga JS-fel. | Webben: "Fantasy SHL svarar inte just nu (tog för lång tid / serverfel 500). Försök igen om en stund." Exe-tipset bara i exe/fil-läget. |
| S5 | C | `:52` `.hbtn`, `:55` `.gwpill select`, `:58` `.lgsel`, `:60` `.sync`, `:136` `.lfind`, `:114` `.faq summary`, `:115` `.about-go` | Klickytor under 44 px. Uppmätt vid 360 px: ⟳ 33×25, ⚙ 32×25, omgångslistan 90×24, ligalistan 139×23, "Uppdaterad" 85×15, "Vet du inte ditt lag-ID?" 281×19–38, FAQ-frågor 23 px höga, "Kom igång" 23 px hög. | Större träffyta utan att utseendet ändras: `min-height:44px` där det inte syns (FAQ, länkknappar), och en osynlig utvidgning (`::after`, `inset:-10px`) på knapparna i sidhuvudet. |
| S6 | D | saknas | Ingen `_headers` (CSP m.m.). | Ny `_headers`, se avsnitt D nedan. |
| S7 | F | saknas | Ingen `404.html` och ingen `robots.txt` (går nu att lägga i domänens rot). | Enkel `404.html` i samma stil (statisk), `robots.txt` med länk till sitemap. |
| S8 | F | `kod/grafik_in.py:51`, `:15–16` | `bakgrund-900.jpg` (70 kB, cirka 95 kB som base64) byggs in men används bara som reserv om canvas saknas – rinken ritas med kod. `index.html` är 708 kB (402 kB gzip), varav 427 kB inbyggda bilder. | Ta bort den inbyggda reservbilden (ersätts av samma mörka toning). Cirka −95 kB. |

## Kosmetiskt (åtgärdas inte)

| # | Fil / rad | Fynd |
|---|---|---|
| K1 | tabeller i Lista/Ligan/Byten/Rivalernas spelare | Spelarnamn i tabellrader är klickbara text-knappar (cirka 20 px höga, radhöjd cirka 33 px). En höjning till 44 px skulle ändra tabellernas täthet (design). |
| K2 | `.tabs button{opacity:.7}` | Ej valda flikar ligger på cirka 3,9:1 i kontrast (muted-grå × 0,7). Vald flik och all brödtext klarar AA (4,5–13,9:1). |
| K3 | diverse `.7rem` | Minsta text 11,25 px (meta-rader, märken). Läsbart men litet. |
| K4 | `:2178, 2183` | Vid fel öppnas inställningsrutan automatiskt ovanpå meddelandet. |
| K5 | – | Sidan är en fil på 708 kB. Allt laddas i ett anrop, och service workern cachar den; godtagbart, men delning i flera filer skulle ge snabbare första visning på långsamma nät. |

## Kontrollerat utan fynd

- **A – sökvägar:** alla lokala resurser är relativa (`icon-*.png`, `manifest.webmanifest`, `sw.js`, `data/…`). Inga `/FantasyRinken/`-sökvägar. Manifestets `start_url`/`scope` är `./`. Service workerns skal använder relativa sökvägar.
- **A – CNAME:** ingen `CNAME`-fil finns i projektet eller i `site/`.
- **A – CORS:** fantasy.shl.se, github.io/raw och GoatCounter svarar `Access-Control-Allow-Origin: *` (testat med `Origin: https://fantasyrinken.se`). Inget behöver ändras hos dem.
- **B – flödet:** laddning, alla flikar (Mitt lag, Ligan, Byten, Kommande, Säsong), utfällt lag, spelarkort (öppna/stäng), byte av omgång fram och tillbaka, Rink/Lista – inga JavaScript-fel. Statnet-datan laddas.
- **B – lagring:** alla 30 läsningar och skrivningar mot `localStorage`/`sessionStorage` ligger i try/catch. Testat med lagringen blockerad (som i vissa privata lägen): startsidan visas, inga fel.
- **B – kantfall:** selftestet (135 fall) täcker lika poäng (samma placering), spelare utan matcher (0 p), saknade värden, kort, byten, JR/VT och prisklass. Alla gröna.
- **B – konsol:** inga fel från appen. De 404-fel och den canvas-varning som syntes kom från mitt testupplägg (serverade `kod/` utan `data/`, min egen pixelmätning), inte från sajten.
- **B – länkar:** externa länkar fungerar (`user-profile` ger 401 utan inloggning, vilket är väntat). Alla lokala filer finns.
- **C – bredder:** ingen sidled-scroll på 360/390/768/1280 px, varken på startsidan eller i någon flik (tabeller scrollar inom sin egen ruta).
- **C – Safari:** `-webkit-backdrop-filter`, `-webkit-background-clip:text`, `-webkit-image-set` finns; `100vh` före `lvh`; reserv för `dialog.showModal`. Inga funktioner utan stöd i aktuell Safari hittades.
- **D – hemligheter:** inga nycklar, lösenord, tokens eller riktiga user codes i 134 publicerade filer eller i git-historiken (bara påhittad testdata som `c@d.se`, `DEMOA1…`).
- **D – XSS:** all data från API:t (lag-, spelar-, kort-, liganamn, felmeddelanden) går genom `esc()` före `innerHTML`. JSON-LD escapar `</`.
- **E – annonser:** avstängda, inga annonsskript. Annonsplatserna är `display:none` → ingen CLS. `ads.txt` behövs inte nu.
- **E – samtycke:** inga cookies sätts. GoatCounter anropas som en bild utan cookie; `localStorage` används bara för att tjänsten ska fungera (undantaget för "strikt nödvändig"). Ingen samtyckesbanner krävs i dag. Den krävs innan annonser slås på (står i LÄSMIG).
- **E – attribuering:** sidfoten anger "Statistik: Statnet AB via shl.se"; förklaringen och FAQ anger att verktyget är fristående från SHL/Fantasy SHL. Jag har inte kunnat läsa några villkor som kräver mer.
- **F:** `lang="sv"`, title, description, OG/Twitter med bild, JSON-LD, favicon (inbyggd), `sitemap.xml`, inga dubbla id, inga `<img>` utan alt (bilder är CSS med `role="img"` + `aria-label`), kontrast AA för brödtext, fokusmarkering (webbläsarens egen, inte borttagen på knappar).

---

## D – `_headers` (Cloudflare Pages)

| Header | Värde | Varför |
|---|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https://fantasyrinken.goatcounter.com; connect-src 'self' https://fantasy.shl.se https://raw.githubusercontent.com; manifest-src 'self'; worker-src 'self'; font-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests` | Bara de källor sajten faktiskt använder: Fantasy SHL (data), GitHub raw (Statnet), GoatCounter (räknare), inbyggda bilder (`data:`). `'unsafe-inline'` behövs eftersom sajten är en fil med inbäddade skript och stilar. `frame-ancestors 'none'` stoppar inbäddning (clickjacking). |
| `X-Content-Type-Options` | `nosniff` | Webbläsaren gissar inte filtyper. |
| `Referrer-Policy` | `no-referrer` | Inga adresser skickas vidare till andra sajter (samma som sidans meta). |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(), payment=(), usb=(), interest-cohort=()` | Stänger funktioner sajten inte använder. |
| `Strict-Transport-Security` | `max-age=31536000` | Alltid https (utan `includeSubDomains`/`preload` tills allt är bekräftat). |
| `X-Frame-Options` | `DENY` | Äldre motsvarighet till `frame-ancestors`. |
| `Cache-Control` (`/`, `/index.html`, `/sw.js`) | `no-cache` | Nya versioner syns direkt. Service workern hämtar alltid färskt först. |

---

## Utfall efter åtgärder (branch `pre-launch`, version 2.1.4)

| # | Status | Vad som testades |
|---|---|---|
| B1 | ✅ Åtgärdad | Byggda filer: canonical, og:url, sitemap, robots och llms pekar på https://fantasyrinken.se/. Enda kvarvarande `github.io` är ett påhittat exempel i selftestet. |
| B2 | ✅ Åtgärdad | Webbläsaren läste Statnet från GitHub raw (200) med sajtens `data/statnet.json` som reserv. Byggavstängningen för `data/**` görs i Cloudflare (checklistan) och kan inte testas här. |
| B3 | ✅ Åtgärdad, **väntar på dig** | `integritet.html` svarar 200 och sidfotslänken finns. `kontakt@fantasyrinken.se` fungerar först när Email Routing är uppsatt. |
| B4 | ✅ Åtgärdad | Alla 82 datum/tid-formateringar under ritning av alla flikar och spelarkort använde `Europe/Stockholm` (uppfångat i webbläsaren). Node visar skillnaden (19:00 vs 13:00 i New York). |
| S1 | ✅ Åtgärdad | `FR_SN_URL` finns i app-, full- och webb-full-filerna. Den lokala filen har inte öppnats via `file://` i testet. |
| S2 | ✅ Åtgärdad | Glöm mig: 189 `fsl`- + 41 `fr`-nycklar + 2 i sessionStorage → 0, startsidan visas. |
| S3 | ✅ Åtgärdad | Ogiltig JSON: "Fantasy SHL svarade med något oväntat …". |
| S4 | ✅ Åtgärdad | 500: "tillfälligt problem (serverfel 500)"; nere: "Kunde inte nå Fantasy SHL …" (utan exe-tips på webben); 20 s: "svarar inte just nu (det tog för lång tid)". Inga JS-fel. |
| S5 | ⚠️ Delvis | Uppmätt träffyta (elementFromPoint): "Vet du inte ditt lag-ID?" 44, FAQ-frågor 44, "Kom igång" 46, KÖR 48, ⚙ 44×44, ⟳ 38×44 (begränsas av grannknappen). **Kvar:** omgångslistan (24 px) och ligalistan (22 px) i sidhuvudet – att förstora dem ändrar sidhuvudets utseende (första försöket krympte omgångskapseln, återställt). Flyttas till kosmetiskt/designbeslut. |
| S6 | ✅ Åtgärdad | `_headers` skapad. CSP testad som meta-tagg: 0 överträdelser, inga fel, allt laddar. Riktiga headers kontrolleras efter deploy (`curl -I`). |
| S7 | ✅ Åtgärdad | `404.html` och `robots.txt` finns och svarar. Att Cloudflare visar 404-sidan för okända adresser testas efter deploy. |
| S8 | ✅ Åtgärdad | `index.html` 708 kB → 634 kB. |
| Ny | ✅ Åtgärdad | **Byten – sparade byten** (rapporterat av dig): avdraget tas nu från Fantasy SHL:s `paid_transfer_points`. Verifierat med läsande anrop för alla 33 lag i omgång 2 (fältet finns; 2 byten → −10, 3 → −20). Omgång 2 hade inga sparade byten, så effekten syns första gången i omgång 3. Selftest för sparat byte (0 p) och officiellt avdrag. Själva Byten-fliken med riktig data har **inte** kunnat visas efter ändringen (den sparade koden i testfliken raderades av Glöm mig-testet, och jag skriver inte in din kod) – testa i telefonen (checklistan). |

**Övrigt från testandet:** min CORS-kontroll skapade en testvisning ("test-cors") i GoatCounter. Tidigare flödestester mot byggd `index.html` kan ha räknat ditt eget lag några gånger (inga sådana anrop syntes i webbläsarens logg, men det går inte att utesluta helt). Testservern har nu statistiken avstängd.
