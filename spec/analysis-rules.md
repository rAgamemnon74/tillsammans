# Analys-regler vid funktionsändringar

> Del av [Tillsammans-specifikation](../tillsammans.md).

Tillsammans försvarar mot specifika governance-risker ([threats.md](threats.md)). För att dessa inte ska glida ur sikte när plattformen utvecklas finns denna systematiska checklista — varje ny funktion, varje spec-ändring och varje designbeslut ska granskas mot reglerna nedan.

**Syftet är inte att förutsätta konflikt.** De flesta föreningar aktiverar noll av reglerna i sin vardag. Reglerna finns för edge-fallen där en funktion, oavsiktligt, öppnar en yta som kan utnyttjas.

**Användning:** När ett förslag diskuteras, räkna igenom regelgrupperna. Dokumentera vilka som aktiveras och hur förslaget hanterar dem. Om ingen regel aktiveras — säg det explicit. Det är själva övningen som är värdet, inte att alltid hitta något.

## 1. Asymmetri mellan enskild medlem och kollektiv

- **Resursasymmetri.** Kan en medlem med mer tid, juridisk/ekonomisk kompetens eller professionellt nätverk använda funktionen för att skapa övertag mot en ideell styrelse? Exempel: formaliakrav som är triviala för en jurist men svåra för en lekmannaledamot att formulera korrekt.
- **Påtryckningsyta.** Öppnar funktionen nya kanaler för informell påverkan vid sidan av den strukturerade beslutsprocessen — direktmeddelanden, "brådskande"-flaggor, sidokanaler till enskilda ledamöter?
- **Formalia som vapen.** Kan kravuppfyllnad användas för att blockera legitima ärenden istället för att förbättra dem? Finns konstruktiva vägar framåt (kompletteringsförslag) i stället för enbart avvisning?

## 2. Historierevision och beslutsförvanskning

- **Redigerbarhet i logg.** Kan funktionen ändra eller dölja äldre logg-poster utan att ändringen själv är spårbar? Rättelser ska alltid vara nya poster som refererar originalet.
- **Praxis över tid.** Dokumenteras återkommande mönster så att en efterträdande styrelse kan jämföra dagens beslut med tidigare — eller går kontinuiteten förlorad vid styrelsebyte?
- **Stadge-ankare.** Kopplar funktionen sitt beslut mot en specifik stadgaparagraf så att logiken kan granskas mot den juridiska grunden, eller svävar beslutet fritt?

## 3. Passiv erosion efter personalväxling

- **Biprodukt eller rutin?** Är funktionens governance-värde en automatisk biprodukt av att styrelsen arbetar i systemet, eller kräver den att en engagerad person manuellt underhåller den? Det senare dör med kohorten.
- **Sökbart arv.** Kommer nästa styrelse kunna hitta, tolka och återanvända det denna styrelse skapat — utan att ha varit med?
- **Default på.** Är den säkra/transparenta konfigurationen default, eller måste styrelsen aktivt välja den? Defaults ärvs; aktiva val glöms bort.

## 4. Taktiskt inflöde och obstruktion

- **Volym-angrepp.** Kan funktionen drunknas i mängd — motionsspam, ärendespam, flodförfrågningar — så att styrelsen förlamas?
- **Gruppering och avvisning.** Finns legitima vägar att gruppera snarlika ärenden eller avvisa på saklig formalia, eller tvingas styrelsen behandla varje enskilt fall oavsett kvalitet?
- **Deadline-asymmetri.** Har styrelsen och medlemmen samma tidsfönster att agera, eller favoriserar funktionen den som kan agera snabbast?

## 5. Jäv och egenintressen

- **Upptäcktsbarhet.** Skulle en direkt eller indirekt intressekoppling bli synlig i funktionens flöde, eller kan den döljas genom strukturvalet? Krävs aktiv deklaration?
- **Likabehandling.** Behandlar funktionen alla medlemmar lika? Finns implicit roll-baserad övervikt — t.ex. styrelseledamot som söker utlägg med egen godkännandemöjlighet, eller valberedning som nominerar sig själv utan spärr?
- **Revisorns linsvy.** Har revisorn permanent läsrätt på de datapunkter funktionen genererar, eller är granskningen beroende av att något aktivt eskaleras?

## 6. Ex-medlemmar och passiva register

- **Rättighets-läckage.** Kan någon som inte längre är medlem fortsätta påverka via datat funktionen skapar — genom cachade roller, gamla fullmakter, kvarstående notifikationsabonnemang?
- **Reconfirmation.** Kräver funktionen att rollen/rättigheten aktivt bekräftas över tid, eller lever den på tidigare beviljanden till den explicit återkallas?

## 7. Persondata och GDPR

- **Minsta nödvändiga data.** Kräver funktionen mer persondata än den faktiskt behöver för sitt syfte?
- **Roll-baserad filtrering.** Filtreras personuppgifter per roll på serversidan, eller litar funktionen på klientens presentation?
- **Gallring.** Har funktionens data en definierad gallringspunkt kopplad till ärendets livscykel?

## 8. Ton och kommunikation

- **Saklig konstruktiv ton.** Använder funktionens automatgenererade texter en god och samarbetsvillig ton (se [case-types.md#textbiblioteket](case-types.md))? Eller riskerar mallen att eskalera konflikt?
- **Stadgehänvisning istället för auktoritetspåbud.** Motiveras restriktiva formuleringar i texter via stadgan/lagen, inte via styrelsens vilja?
- **Konstruktivt avslag.** Vid formalia-avvisning — pekar texten ut en väg framåt, eller stänger den bara dörren?

## 9. Transparens mot medlem

- **Publik vs. intern insyn.** Vad får medlemmen se om funktionens utfall? Följer nivån principen i [mission.md](mission.md) om process-transparens före utfall-påverkan?
- **Personärende-undantag.** Om funktionen hanterar personärenden — är den reducerade insynen ett *explicit och motiverat* undantag från transparens-default, eller smyger det in av bekvämlighet?
- **Stadge-klickbarhet.** Kan medlemmen klicka sig från funktionens utfall till den paragraf beslutet vilar på?

## 10. Gränsen mot operativ funktionalitet

- **Governance-värde.** Bidrar funktionen till en *governance-sköld* (beslut, jäv, protokoll, röstlängd, kassörsstöd, audit) eller glider den in i operativ drift (bokningar, inventarier, event)?
- **Scope-spill.** Om operativ — hänvisas den explicit till externt verktyg med export-integration, eller absorberas den tyst i plattformen?

## Diagnostiska signaler — tolkning vid drift

Analys-reglerna ovan används vid *designtid*. När systemet är i drift producerar det löpande signaler om ofullständighet — template-rendering-brister, vakanta roller, uteblivna protokoll-justeringar, ärenden som inte når beslut inom stadgans tidsfrist, debiteringslängder som inte balanserar. Dessa signaler ska tolkas enligt tre rot-orsaker:

1. **Engagemang/ordningssinne.** Förtroendevalda hann inte, prioriterade inte, eller missade en rutin. Kräver inte systemförändring — kanske stöd och påminnelser, men inte teknik.
2. **System-miss — saknad funktion.** Specen eller implementationen täcker inte det fall användaren behövde registrera. Åtgärden är att utvidga specen.
3. **CX/UX-problem.** Funktionen finns men är för krånglig, otydlig eller kräver för mycket. Åtgärden är gränssnittsjustering — inte ny funktion.

Att blanda ihop dessa leder till fel åtgärd: att skylla på styrelsen när det är ett system-miss, att bygga ny funktion när gränssnittet bara behöver förtydligas, eller att ändra gränssnittet när en funktion faktiskt saknas.

**Signaler från systemet pekar på var problemet kan ligga — aldrig ut vem.** Stämpling av personer är uttryckligen utanför scope ([threats.md#det-systemet-inte-gör](threats.md)). Diagnostiken är en sorteringsmekanism vid drift och vid vidareutveckling.

När en signal förekommer upprepat är det tecken på att en av de tre rot-orsakerna är aktiv och ska adresseras där — inte symptomatiskt på händelsesidan.

## Vad analys-reglerna *inte* är

- **Inte personstämpling.** Ingen av reglerna används för att bedöma enskilda personer. De används för att bedöma om *systemet* har strukturella öppningar för missbruk.
- **Inte algoritmisk flagga.** Reglerna är läslistor för människor som designar funktioner — aldrig körkod som stämplar medlemmar.
- **Inte konfliktsökande.** Normalfallet i en välfungerande förening aktiverar få regler. Checklistan finns för undantagen, inte för vardagen.
- **Inte statisk.** Regeluppsättningen uppdateras när verifikationsrapporter avslöjar nya mönster eller efter incidenter i driftade föreningar.

## Tillämpning i praktiken

När ett spec-beslut diskuteras i en session — eller en PR granskas i framtiden — ska diskussionen kunna svara på:

1. Vilka regelgrupper aktiverades av detta förslag?
2. Hur hanterar förslaget dem?
3. Finns det en regelgrupp där hanteringen är svag eller `[ÖPPEN]`? Flagga den i spec:en eller PR:en.

Den systematiska övningen är försvaret mot smygande design-glidning. Inte varje funktion behöver lång analys — men varje funktion ska ha gått igenom listan.
