# Tillsammans — specifikation

> Status: **Utkast.** Levande specifikation. Öppna frågor är markerade `[ÖPPEN]` i respektive sektionsfil — de ska avgöras tillsammans med användaren innan implementation.

Specifikationen är uppdelad i sektioner under `spec/`. Det här dokumentet är ett innehållsregister — läs in den sektion som är relevant för uppgiften, inte hela specen.

## Sektioner

- **[Uppdrag, förtroendemodell och fastslagna beslut](spec/mission.md)** — *ladda när:* frågor om varför plattformen finns, vem den vänder sig till på principnivå, eller när något fastslaget beslut behöver refereras (licens, scope, editions).
- **[Hotbild och skyddsmekanismer](spec/threats.md)** — *ladda när:* beslut som rör vad plattformen försvarar mot, vilka mekanismer som är vapen i arsenalen, eller gränsen mot "AI-övervakning".
- **[Målgrupp](spec/audience.md)** — *ladda när:* frågor om föreningstyper (samfällighet/föräldraförening/LEF), primärexponerade roller, eller edition-specifik roadmap.
- **[Arkitektur, MVP och kärnmoduler](spec/architecture.md)** — *ladda när:* scope-diskussioner (in/ut), typ-anpassning vs. typ-läckage, MVP-strategi, eller prioritering mellan kärnmoduler.
- **[Kärnkoncept](spec/core-concepts.md)** — *ladda när:* arbete med "Ni lovade"-flödet, röstning (huvudmetod/andelsmetod/splittning), jäv, reservation/solidariskt ansvar, bordläggning, kallelsemodell, anslagstavla.
- **[Ärendetyper](spec/case-types.md)** — *ladda när:* arbete med motioner, medlemsansökan, uteslutning, utläggsgodkännande, stadgeändring, revisorsfråga, kandidatnominering — eller med det gemensamma livscykel- och textmallsbiblioteket för ärenden.
- **[Styrelseroller](spec/roles/board-roles.md)** — *ladda när:* arbete med ordförande, vice ordförande, sekreterare, kassör, ledamot eller suppleant — deras RBAC, ansvar eller vakans-hantering.
- **[Revisionsroller](spec/roles/oversight-roles.md)** — *ladda när:* arbete med revisor eller revisorssuppleant — deras permanenta läsrätt, granskningsuppdrag och konstitutionella oberoende från styrelsen.
- **[Valberedning](spec/roles/nominating-committee.md)** — *ladda när:* arbete med valberedningens ledamöter eller sammankallande — nominerings-flödet, arbetsytan, självjäv-hantering och oberoendet från styrelsen.
- **[Mötesroller](spec/roles/meeting-roles.md)** — *ladda när:* arbete med mötesordförande, mötessekreterare, protokolljusterare eller rösträknare — deras flyktiga tilldelning per stämma, signering, och separation från stående roller.
- **[Övriga roller](spec/roles/other-roles.md)** — *ladda när:* arbete med firmatecknare, utskotts-/kommittéledamot, klassrepresentant, fond-kontaktperson, `ExternalAdministrator` eller bas-rollen medlem/sakägare/vårdnadshavare.
- **[Domänmodell, medlemsmodell och lapptäcket](spec/domain-model.md)** — *ladda när:* entitets-/relationsfrågor, medlemsmodell per typ, livscykel (`ACTIVE`/`LAPSED`/`EXCLUDED`, reconfirmation), eller dokumentkonflikter vid onboarding.
- **[Teoriverifikation](spec/verification.md)** — *ladda när:* planering av stadgejämförelse, typscenarier, juridisk mappning, konflikt-simulering, eller diskussion om verifikations-tröskel före kod.
- **[Analys-regler vid funktionsändringar](spec/analysis-rules.md)** — *ladda när:* ett förslag till ny funktion, spec-ändring eller designbeslut diskuteras — checklistan ska köras igenom innan förslaget accepteras.
- **[Öppna frågor, framtid och agent-regler](spec/open-questions.md)** — *ladda när:* `[ÖPPEN]`-frågor ska diskuteras, post-MVP-aspirationer bedöms, eller gränsen mellan produktfunktioner och utvecklaragent-regler behöver klargöras.
- **[Kontextkunskap (svensk rätt)](spec/legal-context.md)** — *ladda när:* lagrum eller svenska juridiska begrepp behöver refereras.

## Editioner (typ-specifik spec)

Edition-filer dokumenterar det som är specifikt för en `AssociationType`. Kärnans governance-flöden bor i sektionerna ovan; edition-filerna innehåller defaults, stadge-alternativ och domänmönster för varje typ.

- **[Tillsammans för föräldraföreningar](spec/editions/foraldraforening.md)** — *ladda när:* arbete med `AssociationType = föräldraförening`, röstmodeller per sub-typ, klasskassor, underfonder, utskott, huvudmanna-relation, arbetsplikt för kooperativ-förskolor.
- **[Tillsammans för samfälligheter](spec/editions/samfallighet.md)** — *ladda när:* arbete med `AssociationType = samfällighet`, sub-typer (väg / vatten / multi-GA / skärgård / fjäll-väg), normalstadge-wizard, andelstal och debiteringslängd, multi-GA-kostnadsfördelning, extern administratör, underhållsfond.

*LEF får egen edition-fil när verifikation påbörjas.*

## Verifikations-rapporter

Empirisk bakgrund till spec-beslut. Offentliga enligt [mission.md](spec/mission.md)s princip om granskningsbar governance.

- **[Föräldraföreningar 2026-04](spec/verification-reports/foraldraforeningar-2026-04.md)** — 12 svenska föräldraföreningar analyserade mot specen (apr 2026). Drev spec-ändringar i [core-concepts.md](spec/core-concepts.md) (röstmodell, kallelsetid), [domain-model.md](spec/domain-model.md) (LAPSED-triggers), [architecture.md](spec/architecture.md) (utskott/underfonder/huvudmanna-relation) och skapandet av föräldraförening-editionen.
- **[Samfälligheter 2026-04](spec/verification-reports/samfalligheter-2026-04.md)** — 10 svenska samfällighetsföreningar analyserade mot specen (apr 2026). Vände samfällighets-rösten i [core-concepts.md](spec/core-concepts.md) (huvudmetod som default + 1/5-tak + fullmakts-regel), tillade `ExternalAdministrator`-rollen i [architecture.md](spec/architecture.md), och skapade samfällighets-editionen. Kvitterade [verification.md](spec/verification.md) Spår 1-tröskeln för samfälligheter.
