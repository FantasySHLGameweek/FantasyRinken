# Lanseringschecklista – fantasyrinken.se (Cloudflare Pages)

Det här repot (`site/`) är exakt det som publiceras. Källkoden ligger utanför repot (`Fantasy-Rinken/kod/`), och nya versioner byggs med
`python bygg.py <version> --statnet --site` (med `FR_KODER` satt), som speglar `webb/` hit.

## 0. Innan du börjar
- [ ] Granska `findings.md` och ändringarna på branchen `pre-launch` (`git log main..pre-launch`, `git diff main pre-launch`).
- [ ] **Statnet-repot:** `bygg.json "statnet_url"` pekar på `https://raw.githubusercontent.com/fantasyshlgameweek/fantasyrinken/main/data/statnet.json`.
  - Adressen svarar redan i dag (repot `fantasyrinken` finns och är publikt, med en `data/statnet.json`).
  - **Det här repot ska ligga där.** Heter repot något annat: ändra `statnet_url` i `kod/bygg.json` och bygg om.
  - Tills adressen är rätt används sajtens egen `data/statnet.json` som reserv (den uppdateras bara vid nya deployer).

## 1. GitHub (kodlagret)
1. Använd repot **`fantasyshlgameweek/fantasyrinken`**. Det måste vara **publikt**, annars kan sajten inte läsa Statnet-datan via raw.
2. I `site/`:
   ```
   git remote add origin https://github.com/fantasyshlgameweek/fantasyrinken.git
   git push -u origin main
   git push -u origin pre-launch
   ```
   Finns det redan filer i repot (t.ex. en tidigare uppladdning): töm repot först, eller hämta med `git pull origin main --allow-unrelated-histories` och lös eventuella konflikter.
3. På GitHub: skapa en Pull Request **pre-launch → main**, granska och merga. Det är `main` som publiceras.
4. **Settings → Actions → General → Workflow permissions: "Read and write permissions".** Statnet-jobbet (`.github/workflows/statnet.yml`) behöver det för att skriva `data/statnet.json` var tionde minut.
5. **Settings → Pages:** ska vara **avstängt** för det här repot (Cloudflare publicerar).

## 2. Cloudflare Pages
1. Cloudflare → **Workers & Pages → Create → Pages → Connect to Git** → välj `fantasyrinken`.
2. Inställningar:
   - Production branch: **`main`**
   - Framework preset: **None**
   - Build command: **(tomt)**
   - Build output directory: **`/`**
3. **Settings → Builds → Build watch paths:**
   - Include: `*`
   - Exclude: **`data/**`**

   Statnet-jobbets commits (var tionde minut) ska inte ge nya deployer. Gratisnivån har 500 byggen i månaden.
4. Första deployen: öppna `https://<projekt>.pages.dev` och kör testerna i avsnitt 4 där innan domänen kopplas.
5. **Custom domains:**
   - lägg till **`fantasyrinken.se`** och **`www.fantasyrinken.se`**;
   - ligger DNS i Cloudflare skapas posterna automatiskt;
   - omdirigera `www` till `fantasyrinken.se` (Rules → Redirect Rules).
6. **Email Routing:** skapa `kontakt@fantasyrinken.se` → vidarebefordra till din vanliga adress. **Blockerar:** adressen står på integritetssidan.
7. SSL/TLS: läget **Full** och "Always Use HTTPS" på.

## 3. Övrigt
- [ ] **GoatCounter:** kontrollera att räknaren inte är begränsad till vissa domäner (Settings → Sites). Radera testvisningen "test-cors" om du vill.
- [ ] **Google Search Console och Bing Webmaster Tools:** lägg till `https://fantasyrinken.se/` och skicka in `https://fantasyrinken.se/sitemap.xml`.
- [ ] **Gamla sidan** `fantasyshlgameweek.github.io/Verken-Ligan/`: låt den ligga kvar tills fantasyrinken.se är bekräftad. Byt sedan dess `index.html` mot en sida med `<meta http-equiv="refresh" content="0;url=https://fantasyrinken.se/">`, `<link rel="canonical" href="https://fantasyrinken.se/">` och en vanlig länk. GitHub Pages kan inte skicka riktiga 301-omdirigeringar.
- [ ] **Webb-full** (din fullversion i eget repo): ladda upp den nya `versioner/2.1.4/webb-full/`.

## 4. Testa i telefonen efter deploy (iPhone/Safari och Android/Chrome)
- [ ] `https://fantasyrinken.se/` laddar: rinken i bakgrunden, namnet i mitten, förklaringsrutan under lag-ID-fältet.
- [ ] Klistra in hela profilsidan i lag-ID-fältet: bara koden syns. Tryck KÖR.
- [ ] Ligaväljaren visas (om du har flera ligor). Välj en och ladda om: samma liga öppnas.
- [ ] Mitt lag, Ligan, Byten, Kommande och Säsong öppnas. Byt omgång fram och tillbaka.
- [ ] **Byten (omgång 3 och senare):** ett lag med sparat byte som gjort två byten ska få avdrag 0 och märkningen "sparat byte". Avdragen ska stämma med fantasy.shl.se.
- [ ] Tryck på en spelare: kvittot visas, rutan stängs med ✕, Stäng, tryck utanför. Sidan bakom rullar inte.
- [ ] Kvittot visar statistik ("uppdaterad hh:mm"), alltså att Statnet-datan läses från GitHub.
- [ ] Tider (deadline, matcher) visas i svensk tid.
- [ ] Sidfotens "Integritet och kontakt" öppnar integritetssidan, och e-postlänken fungerar.
- [ ] `https://fantasyrinken.se/finns-inte` visar 404-sidan.
- [ ] Lägg till på hemskärmen: appen startar, ikonen är rätt.
- [ ] Privat/inkognito-läge: sidan fungerar (utan att minnas något).
- [ ] Headers: `curl -I https://fantasyrinken.se/` visar `content-security-policy`, `x-content-type-options`, `referrer-policy` och `strict-transport-security`. Konsolen i datorns webbläsare ska inte visa några CSP-fel.
- [ ] Nästa matchdag: live-poäng och "Uppdaterad hh:mm" uppdateras.

## 5. Rulla tillbaka om något går fel
1. **Snabbast:** Cloudflare → projektet → **Deployments** → välj senaste fungerande → **⋯ → Rollback to this deployment**. Det tar några sekunder och kräver ingen ny commit.
2. **I koden:** `git revert <commit>` → `git push`. Cloudflare bygger om automatiskt.
3. **Nödläge:** den gamla github.io-sidan ligger kvar tills allt är bekräftat. Peka tillfälligt folk dit, eller ta bort custom domain i Cloudflare.
4. Statnet-jobbet påverkas inte av en rollback. Det skriver bara `data/statnet.json` i GitHub.
