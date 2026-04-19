# Kandidatnominering

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Förslag om kandidat till en förtroendepost (styrelse, revisor, valberedning). Stödjer både valberedningens egna förslag och medlemmars öppna nomineringar — inkl. självnominering. Mekaniken försäkrar **kandidatens samtycke** innan publicering, och **konfidentialitet för avvisade kandidater** (princip 5 från [nominating-committee.md](../roles/nominating-committee.md#principer)).

**Edition:** bägge.

## Primär initiator

- **Valberedning:** `NOMINATING_*` lägger fram nominering som del av samlat förslag.
- **Medlem:** `MEMBER` (status `AKTIV`) lägger självnominering eller nominerar annan medlem.

## Obligatoriska fält

- **Föreslagen post** — `BOARD_CHAIR` / `BOARD_TREASURER` / `BOARD_MEMBER` / `BOARD_DEPUTY` / `AUDITOR` / `AUDITOR_DEPUTY` / `NOMINATING_MEMBER` / `NOMINATING_CONVENER`.
- **Mandatperiod** — vilken period nomineringen avser (typiskt nästa stämmas valperiod).
- **Föreslagen kandidat** — `Medlem`-referens (eller extern person för revisor — se [oversight-roles.md#öppna-frågor](../roles/oversight-roles.md#öppna-frågor)).
- **Föreslagare** — den som lägger fram förslaget; vid självnominering = kandidaten själv.
- **Bakgrund** — strukturerade fält (erfarenhet, tidigare roller); inga subjektiva omdömen om lämplighet (se [nominating-committee.md#öppna-frågor](../roles/nominating-committee.md#öppna-frågor)).
- **Bilagor:** ev. CV / kort presentation (kandidat tillhandahåller).

## Livscykel

Specialiserad med samtyckes-fas:

```
INLÄMNAD (förslag inlämnat)
  → SAMTYCKES_FÖRFRÅGAN (kandidaten tillfrågas)
    → SAMTYCKE_GIVET
      → BEREDNING_VALBEREDNINGEN (om medlem-nominerad — valberedningen tar ställning)
        → INKLUDERAD_I_FÖRSLAG (valberedningens förslag inkluderar kandidaten)
        → AVVISAD_AV_VALBEREDNING (konfidentiell — kandidaten kan ändå ställa upp på stämman)
      → PUBLICERAD (del av valberedningens förslag inför stämma)
        → PÅ_STÄMMA
          → AVGJORD
            · VALD
            · INTE_VALD
    → SAMTYCKE_AVSLAGET (kandidaten tackar nej)
      → AVSLUTAT

Sidospår:
  → ÅTERKALLAD (föreslagaren drar tillbaka)
  → SJÄLVANMÄLD_PÅ_STÄMMA (medlem anmäler sig direkt på stämman, förbi valberedningen)
```

Vid medlem-nominering går ärendet via valberedningens beredning. Vid valberedningens egna förslag hoppas `BEREDNING_VALBEREDNINGEN` över.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INLÄMNAD | `MEMBER`, `NOMINATING_*` |
| INLÄMNAD → SAMTYCKES_FÖRFRÅGAN | automatik (notifiering till kandidat) |
| SAMTYCKES_FÖRFRÅGAN → SAMTYCKE_GIVET / AVSLAGET | föreslagen kandidat |
| SAMTYCKE_GIVET → BEREDNING_VALBEREDNINGEN | automatik vid medlem-nominering |
| BEREDNING → INKLUDERAD / AVVISAD | `NOMINATING_*` |
| INKLUDERAD → PUBLICERAD | automatik vid kallelseutskick |
| PÅ_STÄMMA → AVGJORD | stämmans val |
| skapa SJÄLVANMÄLD_PÅ_STÄMMA | `MEMBER` (på stämman, registreras av mötessekreterare) |

## Tidsregler

- **Nomineringsperiod:** stadge- eller stadgekonfigurations-bunden — öppnas och stängs av valberedningens sammankallande.
- **Samtyckes-fönster:** kandidaten har default 14 dagar att svara. Ej svar inom fristen → ärendet stängs som `SAMTYCKE_AVSLAGET` (default).
- **Valberedningens beslut:** senast så att samlat förslag hinner med kallelsens publicering.
- **Självnominering på stämma:** möjlig fram till valpunkten påbörjas (svensk praxis).

## Relationer

- Nominering ↔ `Medlem` (kandidat).
- Nominering ↔ `Medlem` (föreslagare); samma som kandidat vid självnominering.
- Nominering ↔ `RoleAssignment` vid `VALD`.
- Nominering ↔ valberedningens samlade `NominationProposal` (parent).
- Nominering ↔ ev. tidigare nominering av samma person (mönster över tid).

## Publicering

| Status | Publik | Medlem | Styrelse | Valberedning | Kandidat | Föreslagare |
|---|---|---|---|---|---|---|
| INLÄMNAD | — | — | — | läs | — | läs |
| SAMTYCKES_FÖRFRÅGAN | — | — | — | läs | **läs** + svar | läs |
| SAMTYCKE_AVSLAGET | — | — | — | läs (konfidentiell) | läs | läs |
| BEREDNING_VALBEREDNINGEN | — | — | — | läs + skriv | — | — |
| AVVISAD_AV_VALBEREDNING | — | — | — | läs (konfidentiell) | läs (princip 5) | läs |
| INKLUDERAD_I_FÖRSLAG | — | (efter publicering) | (efter publicering) | läs | läs | läs |
| PUBLICERAD | **läs** (namn + post + bakgrund) | **läs** | **läs** | läs | läs | läs |
| AVGJORD | **läs** (utfall) | läs | läs | läs | läs | läs |

**Konfidentialitet för avvisade kandidater:** `SAMTYCKE_AVSLAGET` och `AVVISAD_AV_VALBEREDNING` syns inte publikt (princip 5 från [nominating-committee.md](../roles/nominating-committee.md#principer)). Skyddar den som frivilligt ställt upp men inte gick vidare.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INLÄMNAD | Valberedningen, föreslagare (kvittens) |
| SAMTYCKES_FÖRFRÅGAN | Föreslagen kandidat (förfrågan med tydligt val) |
| SAMTYCKE_GIVET | Valberedningen, föreslagare |
| SAMTYCKE_AVSLAGET | Valberedningen, föreslagare |
| BEREDNING / AVVISAD | Föreslagare (informativ) |
| INKLUDERAD / PUBLICERAD | Kandidaten (informativ — du finns på valberedningens lista), alla medlemmar (via kallelsen) |
| AVGJORD: VALD | Vald kandidat (välkomst), styrelsen, kassören |
| AVGJORD: INTE_VALD | Kandidat (konstruktivt besked), styrelsen |

## Textbibliotek

Neutralt — varken tryck (*"vi räknar med dig"*) eller avrådan.

- **Inlämningskvitto till föreslagare** — *"Tack för förslaget. [Kandidaten] kommer att tillfrågas inom kort. Du får besked när hen svarat."*
- **Samtyckes-förfrågan till kandidat** — *"Hej [namn], du har föreslagits till posten [post] i [förening] för mandatperioden [period]. Föreslagaren är [namn] [/ du själv om självnominering]. Innan ditt namn publiceras behöver vi din bekräftelse. Du kan tacka ja, tacka nej, eller be om mer information via [länk]. Inget tvång — det är helt ok att tacka nej."*
- **Bekräftelse av samtycke** — *"Tack för ditt samtycke. Valberedningen tar ditt namn vidare i sitt arbete. Du får besked om hur det landar inför kallelsen till stämman."*
- **Avslag av samtycke** — *"Tack för ditt svar. Vi noterar att du inte vill kandidera den här gången. Inga andra åtgärder behövs från din sida."*
- **Avvisning av valberedning (konfidentiellt)** — *"Hej [namn], valberedningen har efter genomgång valt att inte föra fram ditt namn i sitt samlade förslag den här gången. Skäl: [valberedningens motivering — sakligt formulerat]. Du har rätt att ändå anmäla dig direkt på stämman om du vill. Tack för att du var villig."*
- **Inkluderings-meddelande** — *"Ditt namn finns med i valberedningens samlade förslag inför stämman DD/MM. Förslaget publiceras med kallelsen. Vid stämman kommer stämman att rösta — du behöver inte närvara, men det är välkommet."*
- **Stämmoresultat: vald** — *"Vi gratulerar — stämman har valt dig till [post] för perioden [period]. Konstituering av styrelsen sker DD/MM; mer information kommer."*
- **Stämmoresultat: inte vald** — *"Stämman valde [vald person] till posten. Tack för att du var villig — föreningen behöver alltid engagerade kandidater och vi hoppas att du överväger att ställa upp i framtiden."*

## GDPR-hänsyn

Måttlig–hög känslighet under beredning. Avvisade och avslagna kandidaters identitet skyddas (princip 5). Bakgrundsinformation (CV) är persondata — gallras 12 månader efter mandatperiodens slut för icke-valda; för valda bevaras som del av roll-historiken under hela mandatperioden + 5 år. Subjektiva omdömen ska *inte* skrivas i strukturerade fält (diskrimineringsrisk).

## Hot och försvar

Se [threats.md](../threats.md):

- **Påtryckningar mot kandidater:** samtyckes-fas + neutral ton i mallar minskar social press; kandidaten kan tacka nej utan att skälen blir publika.
- **Självjäv vid valberedning:** [nominating-committee.md#jäv-mellan-valberedning-och-styrelse](../roles/nominating-committee.md#jäv-mellan-valberedning-och-styrelse) flaggar vid `INKLUDERAD_I_FÖRSLAG` om föreslagaren är valberedningsledamot som föreslår sig själv.
- **Konfidentialitet vs. transparens:** princip 5 är default; stadgan kan öppna för full publicering om föreningen så väljer.
- **Diskriminering:** strukturerade bakgrundsfält + förbud mot subjektiva lämplighets-omdömen.

## Öppna frågor

- `[ÖPPEN]` Får valberedningen avvisa en kandidat utan motivering? Förslag: motivering är obligatorisk men endast synlig för kandidaten själv (princip 5) — inte publikt.
- `[ÖPPEN]` Vid självnominering på stämman (`SJÄLVANMÄLD_PÅ_STÄMMA`): hur säkerställs att kandidaten faktiskt samtycker? Förslag: muntligt samtycke vid stämman dokumenteras av mötessekreterare; inget separat samtyckes-fönster.
- `[ÖPPEN]` Får extern person nomineras till revisor (se [oversight-roles.md](../roles/oversight-roles.md#öppna-frågor))? Förslag: ja — kandidat utan medlemskap kan registreras som extern aktör med kontaktuppgifter för kallelse-flöden.
- `[ÖPPEN]` Vid `INTE_VALD`: ska personen automatiskt komma med på nästa års nomineringslista, eller börjar det om? Förslag: börjar om — explicit förslag varje år bevarar kandidatens egen agens.
