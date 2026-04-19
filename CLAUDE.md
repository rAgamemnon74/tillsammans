@tillsammans.md

# Tillsammans

Open source-plattform för föreningsdemokrati i Sverige — samfälligheter, ekonomiska föreningar och föräldraföreningar. Kärnfokus: governance och transparens som skyddar lekmannastyrelser mot informella påtryckningar, asymmetrier och historierevision.

## Analys-regler vid funktionsändringar

Varje gång en ny funktion föreslås, en befintlig ändras, eller ett spec-beslut tas, ska förslaget granskas mot [spec/analysis-rules.md](spec/analysis-rules.md). Checklistan är projektets systematiska försvar mot att governance-öppningar smyger in via välmenande funktioner. Dokumentera i diskussionen hur funktionen hanterar de regler som aktiveras — om ingen regel aktiveras, säg det explicit.

## Status

**Specifikationsfas.** Ingen kod är skriven ännu. Fullständig spec ligger i [tillsammans.md](./tillsammans.md). Öppna specifikationsfrågor är markerade där — diskutera med användaren innan implementation påbörjas.

## Relation till Hemmet

Systerprojekt till Hemmet (`../hemmet`). Fristående kodbas, men designmedveten om Hemmet:
- Svensk UI, svenska URL-sökvägar, all kod på engelska (samma konvention)
- Följer sannolikt liknande teknikval (Next.js + Prisma + tRPC + Tailwind), men beslut tas när specifikationen är stabil
- Dela aldrig Hemmet-specifik domänlogik (BRF ≠ samfällighet ≠ föräldraförening) — konceptuellt syskon, inte gemensam kärna

Läs gärna `../hemmet/CLAUDE.md` för att förstå hur systerprojektet är strukturerat — men kopiera inte in dess regler utan att först fråga om de gäller här.

## Licens

MIT (samma som Hemmet).
