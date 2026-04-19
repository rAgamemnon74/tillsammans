# Valberedning

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Valberedningen är stämmans eget beredningsorgan för val av nya förtroendevalda — **fristående från styrelsen**. Dess uppgift är att inventera sittande roller, söka kandidater, och föreslå en samlad valslista till nästa stämma.

En valberedningsledamot bör inte samtidigt vara styrelseledamot. Självnominering till styrelsen skapar en uppenbar jävsituation; flera observerade stadgar förbjuder det uttryckligen eller hanterar det via särskild jäv-paragraf. Systemet flaggar sammanfall och uppmanar stämman att lösa situationen.

**Relaterade dokument:**
- [board-roles.md](board-roles.md) — styrelserollerna som valberedningen nominerar till
- [oversight-roles.md](oversight-roles.md) — det andra fristående stämma-valda organet (revision)
- [case-types.md](../case-types.md) — kandidatnominering som ärendetyp
- [core-concepts.md](../core-concepts.md) — jäv

## Principer

1. **Oberoende från styrelsen.** Valberedningen är inte styrelsens förlängda arm utan stämmans eget verktyg. Systemet behandlar den som separat organisation med egen arbetsyta, inte som styrelse-delmängd.
2. **Valberedningen föreslår — stämman väljer.** Valberedningens förslag är en bindande rekommendation men stämman kan alltid avvika och välja andra kandidater. Systemet underlättar bägge spåren (valberedningens samlade förslag + öppen nominering på stämman).
3. **Jäv mot styrelsen är spärrat; självjäv flaggas.** Person som är styrelseledamot kan tekniskt tilldelas valberedning men systemet varnar skarpt; person som föreslår sig själv till styrelsen i valberedningens slutprotokoll flaggas för stämmans egen bedömning.
4. **Kandidatens samtycke är krav.** Ingen nomineras utan att ha samtyckt. Systemet skickar bekräftelseförfrågan; nominering publiceras inte förrän bekräftelse finns.
5. **Konfidentialitet för avvisade kandidater.** En kandidat som föreslagits men som valberedningen inte för vidare syns inte publikt som standard — skyddar den som frivilligt ställt upp från offentlig avrådan.

## Valberedningsledamot

- **Stödnivå:** förstklassig.
- **Lag- och stadge-grund:** Föreningsstadgar. Obligatorisk i alla observerade föreningar (12 föräldraföreningar + 10 samfälligheter).
- **Ansvar:** Tillsammans med övriga valberedningsledamöter — inventera sittande förtroendevalda, intervjua kandidater, föreslå en samlad valslista till stämman.
- **RBAC-kärna:**
  - Läsrätt på medlemsregistret (namn + kontakt; GDPR-filtrerat enligt [analysis-rules.md](../analysis-rules.md) grupp 7).
  - Läsrätt på tidigare styrelsebeslut och närvaro — för att kunna inventera sittande.
  - Skrivrätt i valberedningens arbetsyta (intern).
  - Skrivrätt på kandidatnominering-ärenden ([case-types.md](../case-types.md)).
  - Publiceringsrätt på valberedningens samlade förslag inför stämma.
- **Edition-avvikelser:** I mindre kooperativ-förskolor ofta en person istället för en grupp — stadgan accepterar detta. Onboarding-wizarden frågar om *antal* valberedningsledamöter, inte om *stadgan tillåter* en ensam person.
- **Hot att skydda:** Självjäv — en valberedningsledamot som föreslår sig själv till styrelsen. Systemet flaggar det i förslaget så att stämman kan se fallet och hantera det.

## Valberedningens sammankallande

- **Stödnivå:** förstklassig. `[ÖPPEN]` möjlig degradering till attribut `convener: true` på en valberedningsledamot.
- **Lag- och stadge-grund:** Stadgar namnger rollen specifikt i vissa föreningar; tradition i andra.
- **Ansvar:** Kalla valberedningen till arbete, representera gruppen utåt mot styrelsen och medlemmarna, säkerställa att det samlade förslaget är klart i tid inför kallelsen till stämman.
- **RBAC-kärna:** Som övriga valberedningsledamöter plus rätt att öppna nomineringsperiod, stänga den vid angiven tidpunkt, och publicera gruppens konsensus-förslag.
- **Edition-avvikelser:** Inga strukturella.
- **`[ÖPPEN]`:** Egen rollnyckel vs. attribut. Om stadgan namnger rollen explicit → egen rollnyckel. Om det är implicit *"gruppen väljer en sammankallande internt"* → attribut räcker. Förslag: attribut `convener: true` för att undvika rollexplosion; avgörs i datamodelleringen.

## Valberedningens arbetsyta

Separat från både styrelsens arbetsyta och revisorns. Innehåller:

- **Sittande-inventering** — alla förtroendevalda poster, mandattider, vakansflaggor (bägge organ + styrelsen).
- **Kandidatlista** — kandidater per position, samtyckestatus, intervjuanteckningar (interna, ej publika).
- **Historik** — tidigare års valslistor och stämmans faktiska val; mönster över tid.
- **Nomineringsperiod** — öppnas av sammankallande, stängs automatiskt vid angiven tidpunkt.
- **Publik förslagsvy** — färdigställs inför kallelse, blir del av kallelsematerialet.

## Nomineringsperiod

Tidsfönster då medlemmar själva kan föreslå kandidater till valberedningen. Inte alla föreningar har öppen nomineringsperiod — vissa låter valberedningen arbeta helt internt. Stadge-inställning.

När öppen:

- Medlem föreslår kandidat via ärendetypen kandidatnominering ([case-types.md](../case-types.md)).
- Kandidaten bekräftar eller tackar nej.
- Valberedningen tar förslaget i beaktande vid sitt samlade förslag.
- Förslag som valberedningen väljer att inte föra fram blir inte publika automatiskt (princip 5) men kan återanmälas av medlemmen direkt på stämman.

## Jäv mellan valberedning och styrelse

Två situationer att hantera:

1. **En person är både styrelseledamot och valberedningsledamot.** Tekniskt tillåtet om stadgan inte uttryckligen förbjuder, men systemet varnar vid tilldelning och föreslår att personen avsäger sig den ena rollen.
2. **En valberedningsledamot föreslår sig själv till styrelsen.** Systemet flaggar förslaget med *"valberedningsledamot nominerad till styrelsen — stämman bör beakta självjävet"* i publik förslagsvy. Stämman avgör; flaggan är informativ, inte blockerande.

## Öppna frågor

- **Självnominering utan valberedningens medverkan.** En medlem kan själv anmäla sig direkt på stämman som kandidat, förbi valberedningens förslag — svensk praxis: det kan inte förbjudas. Systemet ska stödja bägge vägarna.
- **"Lämplighetsgranskning".** Vissa föreningar förväntar sig att valberedningen granskar kandidaters lämplighet (bakgrund, tidigare föreningserfarenhet). Detta är subjektivt och potentiellt problematiskt GDPR-mässigt. Förslag: systemet erbjuder strukturerade fält (erfarenhet, tidigare roller) men inte värderande omdömen.
- **Kandidat-konfidentialitet som stadge-val.** Princip 5 (avvisade kandidater syns inte publikt) är default; stadgan kan öppna för full publicering. Onboarding-wizarden frågar om preferens.
