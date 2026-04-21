# Grunddata — loggen som grund, entiteter som projektion

> Del av [Tillsammans-specifikation](../tillsammans.md).

Två perspektiv på systemets data som båda är nödvändiga:

- **Loggen är grunden för transparens och förtroende.** Alla händelser skrivs som logghändelser. Sanningen om vad som har hänt i föreningen är summan av dess logghändelser, inte tabellrader.
- **Entiteter, roller, aktiviteter, funktioner, flöden/processer och ärenden är grunden för användbarhet.** De är projektioner av loggen som människor interagerar med. Utan dem är systemet obrukbart i praktiken.

De är inte konkurrerande lager utan komplementära. Loggen bär sanningen; entiteterna bär funktionen. Vid konflikt mellan de två är loggen auktoritativ — entiteterna följer loggen, inte tvärtom.

Detta är en konkretisering av [Grund-policies](mission.md#grund-policies) nr 1 (synlighet framför tvingning) och 11 (bevarande med bevisvärde).

## Den absoluta minimum-uppsättningen

För att loggen ska kunna börja snurra krävs *fyra datafält + en användare*:

1. **Org-nummer** — identifierar föreningen unikt.
2. **Namn** — så människor känner igen den.
3. **`AssociationType`** — samfällighet eller LEF. Utan detta kan inga defaults, terminologi eller editions aktiveras (se [policy 13: typ-anpassning utan typ-läckage](mission.md#grund-policies)).
4. **Första förtroendevald** — en autentiserad person med rätt att skriva i loggen. Initiatorn.

Det är det. Allt annat kommer in som logghändelser över tid.

## Genesis-händelserna

Minimum-uppsättningen skrivs som två händelser:

- `FÖRENING_REGISTRERAD` — org-nummer, namn, typ, räkenskapsår (default kalenderår för samfällighet — styrs av edition-konfigurationen).
- `ROLL_TILLDELAD` — initiatorn tilldelas initial roll (typiskt ordförande, sekreterare eller kassör enligt uppladdat konstituerande protokoll).

Efter genesis är föreningen **drift-redo**. Vad den sedan kan göra beror på vilka andra händelser som hunnit skrivas.

## Loggens omfång — formella händelser, inte åsikter eller debatter

Granskningsloggen innehåller **formella händelser** som föreningen antingen är skyldig att dokumentera (LEF/LFS-tvingande protokollföring) eller som part självvalt bidrar med (reservation med egen motivering, medlem publicerar egen motion). Inga "konfigurationsvärden", "inställningar" eller "import-tabeller" vid sidan av — föreningens formella tillstånd är summan av dess händelser:

- Medlem läggs till → `MEDLEM_REGISTRERAD`
- Stadga laddas upp → `STADGEVERSION_UPPLADDAD`
- Andelstal registreras → `ANDELSTAL_REGISTRERAT`
- Styrelsebeslut fattas → `STYRELSEBESLUT` + `RÖST_AVGIVEN`
- Motion lämnas → `ÄRENDE_INLÄMNAT`

Fullständig händelsetyps-katalog i [granskningslogg.md](granskningslogg.md).

**Konsekvens 1:** Ingen bulkimport är "setup-data" — en medlemsimport är *en serie `MEDLEM_REGISTRERAD`-händelser*, alla synliga i loggen.

**Konsekvens 2:** Inget skrivs över. Varje ändring är en ny händelse. Aktuellt tillstånd är projektion av hela loggen bakåt. Rättelse sker via ny korrigerings-händelse, inte via mutation.

### Vad som *inte* hör i loggen

Tre gränser — per [policy 8 i mission.md](mission.md#grund-policies):

- **Åsikter om personer.** Svensk rätt (Regeringsformen 2:3 + GDPR Art. 9) förbjuder åsiktsregistrering utan rättslig grund och samtycke. Personliga bedömningar av medlemmar/sökande/entreprenörer hör inte i loggen.
- **Debatt och kommentartrådar.** Diskussion mognar till motion eller förs på stämman — inte som kommentarkedjor i systemet. Parallella debatt-arenor urholkar stämman som förhandlingsforum.
- **Rättsliga ställningstaganden.** Juridiska referenser (polisanmälnings-, åtals-, domnummer) lagras som data, men systemet kategoriserar inte händelser som brott och varnar inte om misstänkta överträdelser innan domstol har avgjort.

## Tre persistens-nivåer

Förening-relaterad data bor i tre lager med olika ändamål och rättsliga egenskaper:

| Nivå | Exempel | Persistens | I hash-kedja | Bevis? |
|---|---|---|---|---|
| **Draft (flyktigt)** | Utkast i formulär, pågående skrivfält | Session-bundet, inte persisterat | – | – |
| **Privat arbetsyta** | Revisorns egna noteringar, medlemsansvarigs beredningsanteckningar, valberedarens arbetsnoter | Ägarens egen data, inte föreningens logg. Ägaren styr retention. | Nej | Nej |
| **Granskningslogg (signerad)** | Formella händelser med tvingande rättslig grund eller självvalt samtycke | Ja | Ja | Ja |

**Draft** är flyktigt. Inget spår efter stängt formulär.

**Privat arbetsyta** är ägarens domän — revisorns anteckningar är *revisorns* data, inte föreningens. Systemet tillhandahåller arbetsytan men persisterar inte innehållet som del av föreningens register. Retention, radering och innehåll är ägarens val. Lösning för Spår 2-rapportens lucka om revisor-anteckningar (lucka 30).

**Granskningsloggen** är föreningens register — formella händelser, hash-kedjad, bevis-värdig.

Det finns **ingen fjärde nivå** som fångar "vem gjorde vad under beredning". Att konstruera en sådan "arbetslogg" skulle bryta mot åsiktsregistreringsförbudet.

## Entiteter som projektion

Entiteterna — `Association`, `Medlem`, `Participation`, `Stadga`, `Meeting`, `Case` och alla andra — är *aktuella tillstånd* beräknade från loggen. De är användarnas arbetsyta: listor, tabeller, formulär, vyer. Oumbärliga för att systemet ska vara brukbart.

**Förhållande:**

- **Skriva** sker till loggen. En användaråtgärd genererar en eller flera logghändelser som sedan projiceras till entiteter.
- **Läsa** sker från entiteterna i vardaglig användning, från loggen vid granskning.
- **Revisorn läser båda** — entiteterna för aktuellt tillstånd, loggen för att granska *hur* tillståndet blev sådant.

Detaljerad entitetsmodell i [domain-model.md](domain-model.md).

## Vad det betyder för onboarding

Ingen fas-grindning. Föreningen blir drift-redo efter två genesis-händelser. Funktioner som kräver viss data blir användbara när den datan existerar i loggen:

| Funktion | Förkrav (händelser i loggen) |
|---|---|
| Styrelsemöte | Genesis + minst en styrelseledamot |
| Stämma | Röstande medlemmar registrerade + stadga uppladdad |
| Debiteringslängd | Andelstal + budget registrerade |
| Auto-stöd på stadge-paragrafer | Paragrafer taggade |
| Revisionsberättelse | Revisor + hela föregående räkenskapsåret i loggen |
| Signerade protokoll | Mötesroller attesterade för aktuellt möte |

Det finns inget "allt-måste-vara-klart"-tillstånd. Föreningen använder det den har; lägger till mer över tid; loggen växer. Detta är den konkreta formen av [policy 7 (gradvis nyttoackumulering)](mission.md#grund-policies).

## Förhållande till tidigare spec-filer

Flera spec-filer beskriver entiteter som om de vore primär data — `Medlem`, `Stadga`, `Möte`, `Case`. Detta är fortsatt korrekt som **arbetsytans beskrivning**; men grundval-principen är att dessa är projektioner av logghändelser. Vid konflikt mellan entitetsbeskrivning och logghändelse-beskrivning är logghändelsen auktoritativ.

## Kopplingar

- [granskningslogg.md](granskningslogg.md) — teknisk logg-infrastruktur (hash-kedja, epoker, kanonisering, OTS).
- [domain-model.md](domain-model.md) — entiteter som projektioner.
- [mission.md#grund-policies](mission.md#grund-policies) — designprinciper som denna fil är konsekvens av.
- [föreningen.md](föreningen.md) — årshjul och vad "aktiv förening" betyder i logg-termer.
