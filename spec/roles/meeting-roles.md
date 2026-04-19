# Mötesroller

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Mötesroller väljs **per stämma** (eller per styrelsemöte) och är formellt distinkta från de stående styrelserollerna. En person kan inneha både — föreningens sittande ordförande kan också väljas till mötesordförande — men systemet måste kunna skilja dem, eftersom lag och stadgar skiljer dem i jäv, signering och ansvar.

**Relaterade dokument:**
- [board-roles.md](board-roles.md) — styrelseroller
- [oversight-roles.md](oversight-roles.md) — revisor och revisorssuppleant
- [nominating-committee.md](nominating-committee.md) — valberedning
- [other-roles.md](other-roles.md) — firmatecknare, kommittémedlem, klassrepresentant, ExternalAdministrator
- [case-types.md](../case-types.md) — motion-livscykel refererar mötesroller vid beredning och beslut
- [core-concepts.md](../core-concepts.md) — röstlängd, röstlängdsetablering, kallelsemodell

## Principer

1. **Ephemerala tilldelningar.** Mötesrollen gäller bara för det specifika mötet. Ingen kvarstående behörighet efteråt.
2. **Separation från stående roller är lagkrav.** LEF 6:19 och LFS 49§ föreskriver att stämman *själv väljer* sin mötesordförande — stadgarna kan inte förbestämma att föreningens ordförande alltid är mötesordförande. Systemet ska inte heller göra det.
3. **Fyra roller, inte fler.** Lagen och normalstadgorna namnger mötesordförande, mötessekreterare, protokolljusterare och rösträknare. Andra "roller" vid mötet (t.ex. tidhållare, teknisk support) är inte formella.
4. **Alla mötesroller kan sammanfalla.** En ordförande kan väljas till mötesordförande; en sekreterare till mötessekreterare. Två justerare kan vara vilka närvarande medlemmar som helst. Men *valet* är fortfarande stämmans, loggat i protokollet.
5. **Mötesrollen följer mötet.** När stämman är avslutad försvinner behörigheten automatiskt. Kvarstående arbete (justering av protokoll) har sin egen övergång — se justerare nedan.

## Mötesordförande

- **Stödnivå:** förstklassig.
- **Lag- och stadge-grund:** LEF 6:19, LFS 49§. Stämman ska välja mötesordförande som första punkt.
- **Ansvar:** Leda stämman. Föredra dagordning, ge och återta ordet, föreslå propositionsordning, genomföra omröstningar, förkunna resultat. Har i praktiken "mötesklubban".
- **RBAC-kärna (under mötet):** Etablera röstlängden tillsammans med stämman (se [core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande)); markera omröstningar som öppnade/avslutade; förkunna beslut; markera bordläggning på stämmans begäran.
- **RBAC-kärna (efter mötet):** Justera protokoll jämte vald justerare (LEF 6:26, LFS 49§). Rollen förblir aktiv tills protokollet är signerat.
- **Edition-avvikelser:** Inga strukturella.
- **Hot att skydda:** Jäv — om föreningens sittande ordförande är part i en stämmofråga kan stämman välja någon annan till mötesordförande just för den frågan. Systemet måste tillåta byte mid-stämma, loggat i protokollet.
- **`[ÖPPEN]`:** Ska systemet stödja *byte av mötesordförande mid-stämma* för enskilda punkter, eller kräva att stämman pausas och ny mötesordförande väljs? Förslag: stödja punkt-specifikt byte, eftersom det är juridiskt legitimt och vanligt vid jäv.

## Mötessekreterare

- **Stödnivå:** förstklassig.
- **Lag- och stadge-grund:** LEF 6:25, LFS 49§.
- **Ansvar:** Föra protokollet vid stämman. Får ordet för att läsa upp förslag, beslutstexter och omröstningsresultat vid behov.
- **RBAC-kärna (under mötet):** Skriva i mötesprotokollet; markera deltagare som närvarande/frånvarande löpande; registrera omröstningsresultat när mötesordföranden förkunnar.
- **RBAC-kärna (efter mötet):** Slutföra protokollet, skicka till justerare. Rollen förblir aktiv tills protokollet är signerat.
- **Edition-avvikelser:** Inga strukturella.
- **Hot att skydda:** Historierevision — protokollet är stämmans primära bevisvärde. Ett slarvigt protokoll gör det svårt att försvara beslut efteråt.

## Protokolljusterare

- **Stödnivå:** förstklassig.
- **Lag- och stadge-grund:** LEF 6:26, LFS 49§. Stämman väljer typiskt två justerare — ofta kallade *"justeringsmän"* eller *"justerare"*; *"rösträknare"* kan väljas som kombinerad roll (se nedan).
- **Ansvar:** Granska protokollets överensstämmelse med det som faktiskt inträffade på stämman. Underteckna protokollet jämte mötesordförande när de är nöjda.
- **RBAC-kärna (under mötet):** Läsa protokollutkastet löpande; lägga kommentarer/invändningar på enskilda punkter utan att blockera mötets gång.
- **RBAC-kärna (efter mötet):** Läsa det slutgiltiga protokollet; begära ändring; signera när nöjd. Rollen är aktiv från mötet till signering — typiskt 2–4 veckor.
- **Edition-avvikelser:** Samfällighet har ofta två justerare av lagligt skäl (LFS 49§). Föräldraförening: stadge-variation från en till två.
- **Hot att skydda:** Justerarens roll *är* skyddsmekanismen — ett protokoll utan justering har sämre bevisvärde. Systemet ska göra det enkelt för justeraren att granska utan att behöva sitta bredvid sekreteraren.
- **`[ÖPPEN]`:** Vad händer om justerare vägrar signera? Förslag: protokollet går till stämman som första punkt vid nästa möte för godkännande; motiveringen för vägran loggas.

## Rösträknare

- **Stödnivå:** förstklassig.
- **Lag- och stadge-grund:** LEF 6:19, stadgar. Oftast valda i början av stämman.
- **Ansvar:** Räkna röster vid sluten omröstning, vid räknad votering, eller när mötesordföranden begär. Ofta samma personer som justerarna — men rollen är formellt separat.
- **RBAC-kärna (under mötet):** Vid sluten omröstning: se individuella röstsedlar i digital form, genomföra räkningen, meddela resultatet till mötesordföranden. Se total röstvikt per alternativ när andelsmetod används.
- **RBAC-kärna (efter mötet):** Ingen. Räkningen är klar när resultatet är förkunnat och protokollfört.
- **Edition-avvikelser:** Andelsmetod i samfällighet kräver att rösträknaren kan hantera viktade röster inkl. `voteCapPerMember` (1/5-tak enligt SFL 49§). Systemet levererar siffran; rösträknaren bekräftar.
- **Hot att skydda:** Likabehandling — rösträknaren är kontrollen mot räknefel som kan gynna ena sidan. I små möten kan rollen slås ihop med justerare (normalstadgor tillåter), men när ett beslut är omtvistat ska de vara separata.

## Sammanfattning — RBAC

| Roll | Under mötet | Efter mötet | Kvarstår tills |
|---|---|---|---|
| Mötesordförande | Leda, etablera röstlängd, förkunna beslut | Signera protokoll | Protokoll signerat |
| Mötessekreterare | Skriva protokoll, registrera resultat | Slutföra protokoll, skicka till justerare | Protokoll signerat |
| Protokolljusterare | Läsa utkast, kommentera | Granska, signera | Protokoll signerat |
| Rösträknare | Räkna röster, meddela resultat | — | Stämman avslutad |

## Relation till stående roller

En stående roll kan väljas till en mötesroll, men det är alltid *stämman* som väljer. Systemet får inte förifylla *"mötesordförande = föreningens ordförande"* utan val — det skulle obstruera stämmans suveränitet.

Däremot kan systemet **föreslå** i UI att *"Anna är föreningens ordförande — kryssa för om stämman vill välja henne till mötesordförande"*. Skillnaden är pedagogiskt: förslag med medvetet val, inte default som ignoreras.

## Öppna frågor

- **Styrelsemötens mötesroller.** Styrelsemöten har också en mötesordförande (nästan alltid föreningens ordförande) och en sekreterare. Ska dessa modelleras som mötesroller precis som stämman, eller automatiskt följa de stående rollerna? Förslag: automatiskt följa med möjlighet till explicit val när det finns skäl (ordförande jävig på en punkt).
- **Digital signering av justerare.** BankID-signering diskuteras i [open-questions.md#öppen-autentisering](../open-questions.md). När det beslutas får justering automatiskt en signeringsväg. Tills dess: loggad godkännande räcker.
- **Stämmans pausning.** Kan en stämma pausas och återupptas senare, och vad händer då med mötesrollerna? Praktiskt fall: en stämma som ajourneras till nästa kväll. Förslag: mötesrollerna kvarstår, en ny öppnings-ceremoni behövs inte. Men `[ÖPPEN]` — lagstödet är osäkert.
