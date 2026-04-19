# Arkitektur, MVP och kärnmoduler

> Del av [Tillsammans-specifikation](../tillsammans.md).

## Arkitektur — rent governance-fokus

**Kärninsikten:**

> Förtroendevald-rollerna är extremt lika mellan föreningstyperna. Det operationella stödet skiljer sig enormt — både *mellan* typer och *inom* samma typ.

Styrelsens vardag — möten, beslut, jäv, protokoll, utbetalningar mot beslut, revisorsgranskning, röstlängd, bordläggning — är nästan identisk oavsett om föreningen förvaltar en skogsväg, en vattenanläggning eller en ekonomisk förening. Det operativa — vad föreningen faktiskt *gör* mellan mötena — divergerar så kraftigt att varje försök att bygga gemensamma operativa moduler skulle drunkna i undantag.

**Slutsats:** Tillsammans bygger *bara* governance-kärnan. Operativa behov löses med befintliga verktyg.

### Inom scope (governance-kärnan)

Domänagnostisk kärna, samma kod oavsett `AssociationType`:

- Roller och RBAC
- Möten, dagordning, protokoll, beslutslogg
- Jäv, bordläggning, "Ni lovade"-flödet
- Stadge-modul + beslut↔paragraf-koppling
- Kassörsstöd (utbetalning mot beslut + stadga, dokumenterar — utför inte)
- Medlemsregister + medlemskapslivscykel (`ACTIVE`/`LAPSED`/`EXCLUDED`, reconfirmation)
- Röstlängd-snapshot-mekanik
- Revisorsstöd, valberedning, audit-trail
- Kallelsemodell
- Dokumentarkiv för beslut-relevanta bilagor (servitut, förrättningsakter, gåvobrev, kvitton)
- Anslagstavla (intern kommunikation *kring* governance)
- Utskott och kommittéer med delegerat beslutsmandat (val, mandatperiod, takbelopp, rapporteringsflöde)
- Underfonder inom föreningens ekonomi (fördelningsregel, beslutsgång, tak/golv, konfidentialitetsnivå)
- Huvudmanna-relation: förening som formell intressent i extern juridisk person (utser ledamot till stiftelse/ab som driver verksamheten)
- Räkenskapsår-konfiguration (kalender 1/1–31/12, brutet läsårsföljande 1/7–30/6, eller stadgebestämt annat fönster)
- Extern administratör (`ExternalAdministrator`) — tidsbegränsad RBAC-roll för extern part som sköter delar av ekonomin (HSB, ekonomibyrå). Stadge-/avtalsbestämt scope (bokföring / fakturering / årsredovisning / hela). Läs/skriv-rättigheter mot ekonomiska vyer, läsrätt övrigt, **inget beslutsmandat** — alla beslut ligger kvar på styrelsen. Empiriskt vanligt i samfälligheter (Kyrkmossen→HSB, Fröåvägen→CarLe Ekonomi).

### Utanför scope (externa verktyg)

Explicit avgränsat bort — finns redan och skiljer sig för mycket per förening:

- Bokningssystem (bastu, släpvagn, lokal, tennisbana)
- Mätarinsamling, vattenavläsning
- Leverantörshantering, upphandling, projekthantering (väg-underhåll, asfaltering, renovering)
- Inventarielistor och utlåning
- Event- och aktivitetskoordinering
- Swish-insamling, betalningsinfrastruktur
- Bokföring (kassörsstödet exporterar underlag dit)
- Social kommunikation bortom beslut (föreningens Facebook-grupp, grupp-SMS, event-inbjudningar)

### Typ-anpassning utan typ-läckage i kärnan

`AssociationType` styr konfiguration, inte kod-förgreningar i kärnlogik:

- **Onboarding-wizarden** forkar per typ (manuell digital tvilling för samfällighet, formell ansökan-flöde för LEF).
- **Terminologi-lager** (i18n-liknande) mappar generiska termer till typ-specifika: `medlem` → *tomtägare* / *sakägare* / *medlem*.
- **Domänmodellen** varierar per typ (PropertyUnit+GA+Participation för samfällighet; renare `Member`-modell för LEF) — men varje typ har en komplett, integrerad modell, inte en hybrid.
- **Röstmetod-defaults** och **stadgemallar** levereras per typ.

**Designregel:** kärnans governance-flöden (möte, beslut, jäv, bordläggning, kassör, röstlängd) får inte innehålla `if (associationType === 'X')`. Typ-specifikt beteende pressas ned i data (konfiguration) eller upp i presentation (terminologi) — aldrig i själva governance-logiken. Om kärnan börjar grena på typ har vi ett läckage att åtgärda.

## MVP — "Governance-motorn"

Första släpp av Tillsammans är i praktiken en anpassad version av Hemmets förtroendevald-stöd. Strategi: återanvänd det som fungerar, byt terminologi, behåll governance-logiken.

**Termöversättning från Hemmet:**

| Hemmet | Tillsammans |
|---|---|
| Lägenhet | Andel / objekt / fastighet |
| Bostadsrättslagen | Föreningsstadgar / Anläggningslagen / LEF |
| Boende | Sakägare / intressent |

**Behålls direkt:** protokoll, jävsdeklaration, beslutslogg, audit-trail, mötesflöde, revisorsroll.

**Anpassas:** röstlängd (se [Kärnkoncept: Röstning](core-concepts.md#röstning)), medlemsregister (fastighetsanknutet), kallelsemodell (LEF-specifik).

## Kärnmoduler

Prioriteringsordning inom MVP — diskuteras med användaren:

1. **Styrelsestöd** — möten, dagordning, protokoll, beslutslogg. Fundamentet för allt annat.
2. **Stadge-modul** — föreningens stadgar som strukturerad data. Alla beslut ska kunna förankras mot en paragraf. Avgör vad styrelsen får besluta själva vs. vad som kräver stämman.
3. **Kassörsstöd / ekonomimodul** — utbetalningar knutna till styrelsebeslut och stadgeparagraf. Kassören ska kunna svara på: *"finns det ett giltigt beslut som täcker denna utgift, och ligger den inom stadgarnas ram?"* Revisorsspår byggs in from-the-start.
4. **Medlemsregister / sakägarregister** — modell beror på föreningstyp, se [Domänmodell: Medlemsmodell per föreningstyp](domain-model.md#medlemsmodell-per-föreningstyp).
5. **Röstlängd** — dynamisk, stödjer huvudmetod och andelsmetod.
6. **Årsmöteshantering** — kallelse, motioner, yttranden, röstning, protokoll.
7. **Revisorsstöd** — löpande läsrättigheter till beslut, jäv, utbetalningar, audit-trail.
8. **Valberedningsstöd** — egen roll, egen arbetsyta, publicering före stämma.
9. **Anslagstavla** — modererad intern kommunikation, rollbaserad.

**Arkitektoniskt centralt:** stadge-modul och beslutslogg är primära datastrukturer som kassör, revisor och ordförande alla läser från. Kassörens utbetalningsvy är en projektion av *"vilka beslut har godkänts och vad har redan betalats ut mot dem"*. Revisorns vy är samma data med granskningslins.
