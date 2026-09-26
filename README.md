# 🕯️ Luolaskut

Staattinen selainsovellus PDF-laskujen generointiin nimi+sähköposti-listasta.
Nimi on kaivettu sanoista luola ja laskut – käyttöliittymä on teemoitettu sen mukaisesti,
mutta itse laskut pysyvät asiallisen näköisinä.
Ei palvelinta, ei asennusta, ei tilien luontia – kaikki tapahtuu selaimessa, eikä vastaanottajalista
lähde koneelta mihinkään.

## Ominaisuudet

- **Vastaanottajien tuonti**: liitä leikepöydältä, tuo **CSV- tai Excel-tiedosto (.xlsx)** tai raahaa
  tiedosto kenttään. Erottimena `;`, `,` tai tab. Valmiin pohjan saa napeista *Excel-pohja* ja
  *CSV-pohja* (samat tiedostot ovat myös repossa).
- **Sarakkeiden tunnistus**: otsikkorivistä tunnistetaan nimi- ja sähköpostisarake, myös
  erilliset `Etunimi`/`Sukunimi`-sarakkeet ja englanninkieliset otsikot (`Name`, `Email`).
  Muut sarakkeet – jäsennumerot, summat – ohitetaan, ja sovellus kertoo mitkä se tunnisti.
  Ilman otsikkoriviä sarakkeet päätellään sisällöstä.
- **Laskun sisältö**: otsikko, monirivinen saatesanat-teksti, laskurivit (`Kuvaus;summa`),
  kuva/logo (oikea ylänurkka, vasen tai leveä banneri) ja alatunniste.
- **Maksutiedot**: saaja, IBAN (tarkistetaan mod-97-algoritmilla), BIC, laskun päivä,
  eräpäivä, juokseva laskunumero ja vapaa lisätietokenttä.
- **Viitenumerot** kahdessa tilassa:
  - *Oma viite jokaiselle* – suomalainen viitenumero, runko = `etuliite + juokseva numero`,
    tarkistenumero 7-3-1-menetelmällä.
  - *Yhteinen viite* – sama viite kaikille. Jos annat pelkän rungon, tarkiste lisätään automaattisesti.
- **Pankkiviivakoodi**: laskuun piirretään Code 128C -viivakoodi ja CSV-vientiin lisätään
  54-merkkinen virtuaaliviivakoodi (versio 4) Finanssiala ry:n pankkiviivakoodi-oppaan mukaisesti.
  Sama numerosarja tulostuu laskulle luettavana tekstinä, jolloin sen voi kopioida verkkopankkiin
  ilman skannausta. Vaatii suomalaisen IBANin, viitenumeron ja summan alle 1 000 000 € – muuten
  sovellus kertoo syyn eikä piirrä koodia.
- **Viivakoodin koko** on valittavissa:

  | | leveys × korkeus | kapein palkki | lukeutuu |
  |---|---|---|---|
  | Suuri (oletus) | 156 × 20 mm | 0,47 mm | 100 dpi asti |
  | Vakio | 104 × 12,7 mm | 0,31 mm | 150 dpi asti |

  Vakiokoko noudattaa oppaan mittoja (70–105 mm, 10–12,7 mm). Suuri ylittää leveysrajan, mikä voi
  haitata pankin maksuautomaatissa mutta ei puhelinsovelluksissa – ja se lukeutuu selvästi
  helpommin sekä tulosteesta että näytöltä. Hiljainen alue on molemmilla 10 moduulia Code 128
  -vaatimuksen mukaisesti.
- **Tulosteet**: esikatselu selaimessa, yksi PDF jossa jokainen lasku omana sivuna,
  tai ZIP jossa oma PDF per vastaanottaja (`1001_Matti_Meikalainen.pdf`) + `laskut.csv`.
- **CSV-vienti**: `nimi;sahkoposti;laskunumero;viite;summa;erapaiva` – kätevä sähköpostien
  massalähetykseen (mail merge), jolla PDF:t toimitetaan vastaanottajille.
- **Selaimeen tallennus on hallittavissa**: oma osionsa sivun lopussa. Tallennuksen voi kytkeä
  pois (jolloin tiedot poistetaan heti eikä uusia kirjoiteta), tallennetut tiedot voi tyhjentää
  ja lomakkeen voi nollata oletuksiin. Molemmat toiminnot vaativat vahvistuksen.

## Sähköpostilähetys (valinnainen)

Sovelluksessa on oletuksena piilotettu osio, joka lähettää jokaiselle vastaanottajalle oman
laskun PDF-liitteenä [Brevon](https://www.brevo.com) rajapinnan kautta suoraan selaimesta.
Osio avataan napista *Näytä sähköpostilähetys*.

Käyttöönotto:

1. Luo Brevo-tili (ilmainen taso riittää: 300 viestiä/vrk) ja vahvista lähettäjän osoite.
2. Luo API-avain kohdassa Settings → SMTP & API → API Keys.
3. Liitä avain sovelluksen kenttään, täytä lähettäjän nimi ja osoite sekä viestipohja.
4. Lähetä ensin testi itsellesi, ja vasta sitten kaikille.

Viestipohjassa voi käyttää paikkamerkkejä `{{nimi}}`, `{{laskunumero}}`, `{{viite}}`,
`{{summa}}` ja `{{erapaiva}}`.

**Avaimesta:** Brevon API-avain antaa täydet oikeudet tiliin, joten sitä **ei tallenneta**
lainkaan – se elää vain avoimen välilehden muistissa ja katoaa sivun sulkiessa. Luo avain
lyhyellä vanhenemisajalla ja poista se laskutuserän jälkeen. Lähettäjän nimi, osoite ja
viestipohja tallentuvat localStorageen, avain ei.

Lähetys etenee vastaanottaja kerrallaan pienellä viiveellä. Epäonnistuneet eivät keskeytä
ajoa, vaan ne listataan lopuksi virheineen, jolloin ne voi yrittää uudelleen.

## Käyttö paikallisesti

```bash
nvm use           # Node-versio .nvmrc-tiedostosta (24)
npm install
npm run dev       # http://localhost:5173
```

## Julkaisu GitHub Pagesiin

Julkaisu tapahtuu GitHub Actionsilla: jokainen push `main`-haaraan ajaa testit, kääntää
sovelluksen ja julkaisee `dist/`-hakemiston Pagesiin (`.github/workflows/deploy.yml`).

Ota käyttöön kerran: **Settings → Pages → Source: GitHub Actions**. Oma verkkotunnus
(`luolaskut.kettuniemi.fi`) tulee `public/CNAME`-tiedostosta, joka kopioituu buildin mukana.

Käännetty tuloste ei ole versionhallinnassa – buildin tekee CI.

## Tekniikka

| | |
|---|---|
| React 19 + TypeScript | käyttöliittymä |
| Vite | kehityspalvelin ja build |
| Vitest | testit |
| [jsPDF](https://github.com/parallax/jsPDF) | PDF-piirto (MIT) |
| [JSZip](https://stuk.github.io/jszip/) | ZIP ja .xlsx (MIT tai GPLv3) |

Kirjastot niputetaan buildissa omaan bundleen npm:stä – sovellus ei lataa mitään CDN:stä
eikä muualta verkosta. Riippuvuuksien eheys tulee `package-lock.json`:n tarkisteista, ja
`npm ci` asentaa tasan lukitut versiot. jsPDF ja JSZip ladataan dynaamisesti vasta kun
laskuja luodaan tai Excel-tiedostoa käsitellään, joten sivun ensilataus pysyy kevyenä.

```bash
npm run dev       # kehityspalvelin
npm run build     # tyyppitarkistus + tuotantobuild dist-hakemistoon
npm run preview   # tuotantobuildin esikatselu
npm test          # Vitest
npm run check     # build + testit
```

## Rakenne

| Hakemisto | Sisältö |
|---|---|
| `src/lib/` | kehysriippumaton logiikka: viitenumerot, IBAN, vastaanottajalistan jäsennys, Excel, PDF-piirto |
| `src/components/` | React-komponentit, yksi per lomakeosio |
| `src/lib/email.ts` | viestipohjat ja Brevon rajapinta |
| `src/hooks/` | `usePersistentState` – lomakkeen tila localStoragessa |
| `tests/` | Vitest-testit ja testiaineistot |

`src/lib` ei tunne Reactia eikä DOM:ia, joten sama koodi ajetaan selaimessa ja testeissä.

## Tiedostot

| Tiedosto | Sisältö |
|---|---|
| `index.html` | Viten entry-tiedosto |
| `src/main.tsx`, `src/App.tsx` | sovelluksen juuri |
| `src/styles.css` | ulkoasu (vaalea ja tumma tila) |
| `public/CNAME` | oma verkkotunnus GitHub Pagesille |
| `.github/workflows/deploy.yml` | testaa, kääntää ja julkaisee |
| `esimerkki-vastaanottajat.csv` | esimerkkiaineisto tuontia varten |
| `vastaanottajat-pohja.xlsx` / `.csv` | pohjat vastaanottajalistalle (samat kuin napeista) |
| `esimerkki-lasku.pdf` | esimerkkituloste |
| `LICENSE` | MIT-lisenssi |

## Huomioita

- Viitenumeron pituus on 4–20 numeroa, eli etuliite + juokseva numero saa olla enintään 19 numeroa.
- **Viivakoodin lukeminen** on mitattu rasteroimalla valmis PDF ja lukemalla se zxing-cpp:llä.
  Suuri koko lukeutuu 100 dpi:hin asti, vakiokoko 150 dpi:hin. Näytöltä skannattaessa PDF
  kannattaa silti zoomata (suurella noin 130 %, vakiolla noin 200 %), sillä A4 näkyy täydessä
  koossa noin 96 dpi:n tarkkuudella. Suomalaiset pankkisovellukset lukevat nimenomaan
  viivakoodin – QR-koodia ne eivät tue, joten sellaista ei ole.
- Pankkiviivakoodin Code 128C -koodaus on omaa koodia (`src/lib/code128.ts`), ja testit vertaavat
  sen tuotosta jsbarcode-kirjastoon 50 satunnaisella koodilla. Lisäksi yksi testi lukee valmiin
  PDF:n sisältövirrasta piirretyt palkit takaisin moduulijonoksi ja vertaa sitä koodaukseen.
- Excel-tuki kattaa `.xlsx`-muodon (luetaan ja kirjoitetaan suoraan JSZipillä, ilman taulukkokirjastoa).
  Vanhaa binääristä `.xls`-muotoa ei tueta – tallenna se Excelissä muodossa `.xlsx` tai CSV. Laskukaavat
  luetaan niiden tallennetusta arvosta, joten tallenna tiedosto Excelissä ennen tuontia.
- Laskurivit ovat samat kaikille vastaanottajille. Jos summat vaihtelevat (esim. opiskelijajäsenet),
  aja laskutus useammassa erässä eri laskuriveillä.
- PDF käyttää Helvetica-fonttia, joka tukee skandeja (ä, ö, å).
- Mahdollinen jatkokehitys: virtuaaliviivakoodi / QR-koodi maksuosioon, laskujen lähetys
  suoraan sähköpostilla (vaatisi palvelimen tai esim. Mailgun-integraation).

## Lisenssi

[MIT](LICENSE) © 2026 laurinie

Riippuvuudet omilla lisensseillään: React (MIT), jsPDF (MIT) ja JSZip (MIT / GPLv3, kaksoislisenssi).
