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

### Vad systemet inte bygger som egna features

Systemet är verksamhetsstödjande — det utgår från att användare är vettiga och censurerar inte innehåll i legitima fritextfält. Det bygger däremot inte features *dedikerade för* eller *exklusivt tjänande* problematiska mönster. Se [policy 8 i mission.md](mission.md#grund-policies) för helhetsramverket; i korthet:

- Inga dedikerade åsiktsregistrerings-fält (*"medlemsbedömning"*, *"problemmedlem-tagg"*).
- Inga kommentartrådar eller forum på ärenden.
- Inga automatiska kategoriseringar av händelser som brott eller flaggor på medlemmar baserat på rättsliga referenser pre-domstol.

Om en styrelseledamot skriver olämpligt innehåll i ett motion-utkast eller beredningsdokument är det föreningens angelägenhet — systemet polisar inte.

## Tre persistens-nivåer

Förening-relaterad data bor i tre lager med olika ändamål och rättsliga egenskaper:

| Nivå | Scope | Persistens | I hash-kedja | Bevis? |
|---|---|---|---|---|
| **Draft (flyktigt)** | Sessionsägaren | Inte persisterat | – | – |
| **Arbetsyta** | Varierar (se *Arbetsytans scope* nedan) | Ja, ägare/rollgrupp styr retention | Nej | Nej |
| **Granskningslogg (signerad)** | Per visibility-klass (PUBLIC/MEMBER/BOARD/AUDITOR/PERSON) | Ja, append-only | Ja | Ja |

**Draft** är flyktigt. Inget spår efter stängt formulär.

**Arbetsytan** är beredningsutrymmet — utkast, anteckningar, delat beredningsarbete. Den är separerad från granskningsloggens hash-kedja (redigerbar, gallringsbar) men lagras i föreningens databas.

**Granskningsloggen** är föreningens register — formella händelser, hash-kedjad, bevis-värdig.

### Arbetsytans scope

Arbetsyta är inte alltid individuell. Beroende på arbetets natur har scope olika varianter:

| Scope | Exempel | Vem ser |
|---|---|---|
| **Individuellt** | Revisorns egna reflektioner; valberedarens intervju-intryck; medlemsansvarigs tidiga bedömning innan beslut | Endast ägaren |
| **Delegerings-/ärendebundet** | Ordförande delegerar upphandling till vice ordförande | Delegerare + delegerad |
| **Rollgrupp — styrelsen** | Sekreteraren skissar dagordning, kassören bifogar utlägg-utkast, ordföranden kommenterar | Alla styrelseledamöter |
| **Rollgrupp — revisorer** | Revisor + revisorssuppleant delar granskningsarbete | Revisor + suppleant |

Jäv bryter scope för det specifika ärendet: en jävig styrelseledamot ser inte arbetsyta-innehåll för det ärendet.

**Handover:** Om en person blir oförmögen (sjuk, avgår) kan individuell arbetsyta överföras via `STYRELSEBESLUT`. Delegeringsbunden arbetsyta återgår till delegeraren när uppdraget avslutas. Rollgrupp-arbetsyta följer rollen — nyvald sekreterare har access till föregående sekreterares pågående ärenden.

Arbetsytans RBAC är åtkomst-scope, inte kryptografisk. Systemet censurerar inte innehållet — det är föreningens ansvar att använda arbetsytan sakligt.

Det finns **ingen fjärde nivå** som försöker spåra *"vem ändrade vad kl. 14:32"* utanför dessa tre mekanismer. Beredningsaktivitet hanteras inom arbetsyta enligt scope; formella händelser går i granskningsloggen.

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
