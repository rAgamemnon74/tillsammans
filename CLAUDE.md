@tillsammans.md

# Tillsammans

Open source-plattform för föreningsdemokrati i Sverige — samfälligheter och övriga ekonomiska föreningar (LEF). Kärnfokus: governance och transparens som skyddar lekmannastyrelser mot informella påtryckningar, asymmetrier och historierevision.

## Analys-regler vid funktionsändringar

Varje gång en ny funktion föreslås, en befintlig ändras, eller ett spec-beslut tas, ska förslaget granskas mot [spec/analysis-rules.md](spec/analysis-rules.md). Checklistan är projektets systematiska försvar mot att governance-öppningar smyger in via välmenande funktioner. Dokumentera i diskussionen hur funktionen hanterar de regler som aktiveras — om ingen regel aktiveras, säg det explicit.

## Status

**Specifikationsfas.** Ingen kod är skriven ännu. Fullständig spec ligger i [tillsammans.md](./tillsammans.md). Öppna specifikationsfrågor är markerade där — diskutera med användaren innan implementation påbörjas.

## Relation till Hemmet och Framtiden

Systerprojekt till Hemmet (`../hemmet`, BRF) och Framtiden (`../framtiden`, föräldraföreningar). Fristående kodbaser, men designmedveten om dem:
- Svensk UI, svenska URL-sökvägar, all kod på engelska (samma konvention i alla tre)
- Följer sannolikt liknande teknikval (Next.js + Prisma + tRPC + Tailwind), men beslut tas när specifikationen är stabil
- Dela aldrig domänlogik (BRF ≠ samfällighet ≠ föräldraförening ≠ LEF) — konceptuellt syskon, inte gemensam kärna

Framtiden knoppades av från Tillsammans april 2026 när föräldraförenings-editionen visade sig ha tillräckligt egna mönster (medlemskap följer barnets skolgång, klasskassor, kooperativ-förskolornas arbetsplikt) för att grumla kärnan som försökte täcka både samfälligheter och FF.

Läs gärna `../hemmet/CLAUDE.md` och `../framtiden/CLAUDE.md` för att förstå hur syskonprojekten är strukturerade — men kopiera inte in deras regler utan att först fråga om de gäller här.

## Licens

MIT (samma som Hemmet).
