# Upphandlingsärende

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Strukturerat flöde för föreningens upphandling av varor och tjänster — vägrenovering, takomläggning, ekonomi-förvaltare, försäkring, tekniska tjänster. Mekaniken hanterar **två ovanliga drag** mot andra ärendetyper:

1. **Jäv-deklaration redan vid initiering**, inte bara vid beslut. Risken börjar när ledamoten föreslår leverantör — då är riktningen redan satt.
2. **Stadge-bestämda tröskelbelopp** för krav på stämmobeslut. Stora upphandlingar är inte styrelsens ensak.

Bilage-tunga ärenden — offerter, kravspec, leverantörsreferenser. Försvarar mot de mest direkt egenintresse-känsliga utgifterna en förening gör.

**Edition:** bägge — relevant för alla föreningstyper med substansiell ekonomi.

## Primär initiator

`BOARD_*` (typiskt ordförande, kassör eller utskottsledamot). Medlem kan föreslå behov via [allmänt ärende](allmant_arende.md) som styrelsen sedan kan omvandla till upphandlingsärende.

## Obligatoriska fält

- **Behovsbeskrivning** — vad ska upphandlas och varför.
- **Stadgaförankring** — vilken paragraf eller stadgebestämmelse motiverar utgiften (t.ex. underhåll enligt §X).
- **Uppskattat värde** — preliminärt belopp; styr vilken beslutsväg som krävs.
- **Beslutsväg** — `KASSÖR_MANDAT` / `STYRELSEBESLUT` / `STÄMMOBESLUT` (avgörs av tröskelvärde).
- **Initiatorns jäv-deklaration** — `INGET_JÄV` / `JÄV` (måste väljas vid initiering, inte senare). Vid `JÄV`: relation till leverantör beskrivs.
- **Kravspecifikation** — strukturerade krav på leverans.
- **Bilagor:** kravspec, ev. teknisk dokumentation, foton, ritningar.

## Livscykel

```
INITIERAT (initiator + jäv-deklaration registrerade)
  → KRAVSPEC_FÖRBEREDS (styrelsen utvecklar kravspec, ev. via utskott)
    → OFFERTFÖRFRÅGAN_UTSKICKAD (förfrågningar går till leverantörer; minst stadge-bunden tid eller praxis 2-3 leverantörer)
      → OFFERTER_INKOMNA (alla offerter registrerade som bilagor)
        → BEDÖMNING (styrelsen jämför och rangordnar)
          → STYRELSEBESLUT (om inom mandat) → AVGJORT
          → ESKALERAT_TILL_STÄMMA (om över tröskelvärde)
            → PUBLICERAT_FÖR_STÄMMA → PÅ_STÄMMA → STÄMMOBESLUT → AVGJORT
          → AVBRUTET (ingen lämplig offert; ärendet stängs)
        → KOMPLETTERANDE_OFFERTER (utvidgad förfrågan)
          → BEDÖMNING (ny iteration)
    → AVTAL_TECKNAT (vald leverantör; underliggande för framtida utlägg/fakturor)
      → AVSLUTAT

Sidospår:
  → ÅTERKALLAT (initiator drar tillbaka eller behov bortfaller)
  → JÄV_OMPRÖVAT (under processen — t.ex. om en ny styrelseledamot upptäcker indirekt jäv)
    → BEDÖMNING (med uteslutna jäviga aktörer)
```

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INITIERAT | `BOARD_*` (med obligatorisk jäv-deklaration) |
| INITIERAT → KRAVSPEC_FÖRBEREDS | `BOARD_*` |
| → OFFERTFÖRFRÅGAN_UTSKICKAD | `BOARD_SECRETARY` eller utskottsledamot |
| → OFFERTER_INKOMNA | automatik vid registrering |
| → BEDÖMNING → STYRELSEBESLUT | `BOARD_*` (jäviga avstår enligt [core-concepts.md#jäv](../core-concepts.md#jäv-conflict-of-interest)) |
| → ESKALERAT_TILL_STÄMMA | `BOARD_*` när uppskattat eller bedömt värde överstiger stadge-bundet tröskelvärde |
| → STÄMMOBESLUT | stämmans beslut |
| → AVTAL_TECKNAT | firmatecknare ([other-roles.md#firmatecknare](../roles/other-roles.md#firmatecknare)) |
| → AVBRUTET | `BOARD_*` med saklig motivering |

## Tidsregler

- **Offert-tid till leverantörer:** stadge-bart eller styrelse-praxis; default minst 14 dagar för att ge skälig svarstid.
- **Tröskelvärde för stämmobeslut:** stadge-bart, typiskt 1–10 % av föreningens årsomsättning eller fast belopp (t.ex. 100 000 – 500 000 kr).
- **Beslutstid:** styrelsen bör besluta inom 30 dagar efter `OFFERTER_INKOMNA` (default), annars informeras leverantörer om förlängning.
- **Avtals-fönster:** stadge-bart från `STYRELSEBESLUT/STÄMMOBESLUT` till `AVTAL_TECKNAT`, default 60 dagar.

## Relationer

- Upphandlingsärende ↔ `STYRELSEBESLUT` eller `STÄMMOBESLUT` (1..1).
- Upphandlingsärende ↔ `Stadgaparagraf` (minst en).
- Upphandlingsärende ↔ ev. `Fund` om belastning sker mot specifik underfond.
- Upphandlingsärende ↔ `Avtal`-entitet vid `AVTAL_TECKNAT`.
- Upphandlingsärende ↔ framtida `UTLÄGG_*` / `FAKTURA_UTSTÄLLD` som refererar tillbaka till denna upphandling som auktorisationsgrund (se [utlaggsgodkannande.md](utlaggsgodkannande.md)).
- Upphandlingsärende ↔ ev. `JÄV_DEKLARERAT`-händelser (vid initiering + vid beslut).

## Publicering

| Status | Publik | Medlem | Styrelse |
|---|---|---|---|
| INITIERAT | — | — | läs (jäv-deklaration synlig) |
| KRAVSPEC_FÖRBEREDS | — | — | läs |
| OFFERTFÖRFRÅGAN_UTSKICKAD | — | — | läs |
| OFFERTER_INKOMNA | — | — | läs |
| BEDÖMNING | — | — | läs |
| STYRELSEBESLUT | (statistik vid AVTAL_TECKNAT) | (informativ) | läs |
| ESKALERAT_TILL_STÄMMA / PUBLICERAT | **läs** | **läs** | läs |
| STÄMMOBESLUT | **läs** | läs | läs |
| AVTAL_TECKNAT | (publik aggregerad rapport) | **läs** (vald leverantör + belopp) | läs |
| AVSLUTAT | (publik aggregerad rapport) | läs | läs |

Offerter under bedömning är inte publika — leverantörers prisuppgifter är affärshemlighet och måste skyddas tills beslut är fattat. Efter `AVSLUTAT` aggregeras data till publik kvartalsrapport (samma princip som utläggsgodkännande): vald leverantör, belopp, kategori, beslutslänk.

Revisor har permanent läsrätt enligt [oversight-roles.md](../roles/oversight-roles.md#revisor).

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INITIERAT | Styrelsen (jäv-deklaration synlig), revisorn (informativ) |
| KRAVSPEC_FÖRBEREDS | Styrelsen, ev. utskott |
| OFFERTFÖRFRÅGAN_UTSKICKAD | Förtecknade leverantörer |
| OFFERTER_INKOMNA | Styrelsen |
| STYRELSEBESLUT / STÄMMOBESLUT | Vald leverantör (kontraktsförslag), ej-valda leverantörer (saklig avslagsmeddelande) |
| AVTAL_TECKNAT | Vald leverantör, kassören, revisorn |
| AVBRUTET | Alla offert-givare (saklig grund) |

## Textbibliotek

- **Initierings-jäv-deklarations-prompt (intern UI)** — *"Du initierar ett upphandlingsärende. Har du, eller någon närstående, ekonomisk eller annan koppling till någon möjlig leverantör? Välj ett alternativ: [Inget jäv] [Jäv — beskrivs nedan]. Detta påverkar din rätt att delta i beredning och beslut."*
- **Offertförfrågan till leverantör** — *"Hej, [förening] söker offerter på [tjänst/vara]. Kravspecifikation finns i bilaga. Skicka offert till [kanal] senast DD/MM. Vid frågor kontakta [namn]."*
- **Bekräftelse till leverantör vid mottagen offert** — *"Tack för er offert. Vi behandlar inkomna offerter och meddelar utfall senast DD/MM."*
- **Beslut till vald leverantör** — *"Vi har valt [leverantör] för [uppdrag]. Avtalsförslag bifogas. Vänligen återkom med signerat avtal senast DD/MM."*
- **Beslut till ej-vald leverantör** — *"Tack för er offert. Föreningen har efter bedömning valt en annan leverantör. Skäl: [saklig grund — pris, leveranstid, referenser eller annan jämförelsefaktor]. Vi uppskattar er medverkan."*
- **Avbrutet** — *"Föreningen har efter bedömning beslutat att inte gå vidare med upphandlingen i nuläget. Skäl: [saklig grund]. Tack för er medverkan."*
- **Stämmoeskalering** — *"Då upphandlingens uppskattade värde överstiger stadgebestämt tröskelbelopp (§X) eskaleras beslutet till föreningsstämman. Beslut fattas vid stämman DD/MM."*

## GDPR-hänsyn

Låg känslighet vad gäller medlemsuppgifter (upphandlingen rör leverantörer, inte medlemmar). Leverantörsuppgifter (firma, kontakt, prisuppgifter) hanteras enligt avtals- och affärssekretess — offerter är hemliga tills beslut, sedan aggregeras vald leverantörs uppgift publikt. Bevaras 7 år enligt bokföringslagen för avtal och fakturering.

Vid jäv där initiator har personlig affärsrelation: deklarationens innehåll (vilken relation, vilken leverantör) är synlig för styrelse + revisor; aggregeras till "jäv-historik" per ledamot över tid.

## Hot och försvar

Se [threats.md](../threats.md):

- **Egenintressen och jäv (kärnhot för denna ärendetyp):** obligatorisk jäv-deklaration vid initiering — inte bara vid beslut. *"Asfaltera vägen fram till ordförandens hus först"*-mönstret motverkas redan på förslagsstadiet.
- **Bakrumsförhandlingar:** offertförfrågan till minst flera leverantörer + kravspec dokumenterad innan offerter inkommer. Ad-hoc upphandling utan dessa steg är svår att försvara mot revisor och stämma.
- **Leverantörs-favoritism:** systemet visar `JÄV_DEKLARERAT`-historik per ledamot över tid — mönster av återkommande relationer blir synliga.
- **Mandatläckage:** tröskelbelopp + automatisk stämmoeskalering hindrar styrelsen från att passera sin egen beslutsbehörighet.
- **Smyg-utgifter via fragmenterade fakturor:** efterföljande utlägg/fakturor refererar denna upphandling som auktorisationsgrund — saknas referens, kassören eskalerar enligt [utlaggsgodkannande.md](utlaggsgodkannande.md).

## Öppna frågor

- `[ÖPPEN]` Tröskelbelopp för stämmobeslut: stadge-bart, men finns det rimliga defaults? Förslag: 5 % av årsomsättning, alternativt fast belopp som stadgan väljer (50 000 – 500 000 kr beroende på föreningsstorlek).
- `[ÖPPEN]` Krav på minst antal offerter: stadge-bart eller systemdefault? Förslag: default rekommendation 3 offerter; stadgan kan föreskriva minst 2.
- `[ÖPPEN]` Vid `JÄV` deklarerad vid initiering: får initiator delta i `KRAVSPEC_FÖRBEREDS`? Förslag: nej — om jäv föreligger, ny initiator från styrelsen tar över redan vid kravspec; jäv-deklarationen följer ärendet som spårbar bilaga.
- `[ÖPPEN]` Direktupphandling utan offertförfarande (akut behov, mycket små belopp): hur ska det modelleras? Förslag: separat sidospår `DIREKTUPPHANDLING` under stadge-bunden tröskel; samma jäv-deklaration vid initiering, men inget offertförfarande. Stadgan reglerar tröskeln.
- `[ÖPPEN]` Avtals-uppföljning: ärendet stängs vid `AVTAL_TECKNAT` — men kontraktet löper. Hur visas pågående avtal i UI? Förslag: ny entitet `Avtal` skapas vid `AVTAL_TECKNAT` med egen livscykel; upphandlingsärendet är historik.
- `[ÖPPEN]` Vid `JÄV_OMPRÖVAT` mid-process: rivs hela bedömningen upp eller fortsätter med kvarvarande styrelseledamöter? Förslag: bedömning fortsätter med kvarvarande, jäv-deklarationen registreras retroaktivt; om så stor andel av styrelsen blir jävig att kvorum inte uppnås → eskalering till stämma.
