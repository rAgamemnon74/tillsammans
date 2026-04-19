# Styrelseförslag

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Styrelseförslaget är styrelsens egen formella framställning till stämman. Det bär samma stadgemässiga tyngd som en motion men har styrelsen som initiator. Skillnaden mot motion är att styrelsens position *är* förslaget — inget separat yttrande-spår behövs.

## Primär initiator

`BOARD_*` via styrelsebeslut. Förslaget skapas under ett styrelsemöte och kräver beslut i styrelsen för att läggas fram — en enskild ledamot kan inte själv föra fram ett "styrelseförslag".

## Obligatoriska fält

- **Yrkande** — en mening, formen *"Styrelsen yrkar att stämman beslutar ..."*.
- **Motivering** — fri text med stadge- och budgetkonsekvenser.
- **Stadgaförankring** — minst en paragraf.
- **Underliggande styrelsebeslut** — referens till `STYRELSEBESLUT` som auktoriserade förslagets framläggande.
- **Bilagor** — underlag, offerter, kalkyler.

## Livscykel

```
INITIERAT (styrelsens utkast)
  → ANTAGET (styrelsen beslutar att lägga fram förslaget)
    → PUBLICERAT (del av kallelse till stämma)
      → PÅ_STÄMMA
        → AVGJORT
          · BIFALL
          · AVSLAG
          · ÅTERREMISS

Sidospår:
  → ÅTERKALLAT (styrelsen drar tillbaka före PUBLICERAT — kräver nytt styrelsebeslut)
  → BORDLAGT_TILL_STAMMA (se [core-concepts.md#bordläggning](../core-concepts.md#bordläggning))
```

Inget yttrande-spår — styrelsens ställningstagande sker redan vid `ANTAGET`.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INITIERAT | `BOARD_*` (utkastsfas) |
| INITIERAT → ANTAGET | `BOARD_*` genom styrelsebeslut |
| ANTAGET → ÅTERKALLAT | `BOARD_*` genom nytt styrelsebeslut |
| ANTAGET → PUBLICERAT | automatik vid kallelseutskick |
| PÅ_STÄMMA → AVGJORT | stämmans beslut, verkställs av `BOARD_SECRETARY` |

## Tidsregler

- **Antagande senast:** så att `PUBLICERAT` hinner ske före kallelsens min-gräns enligt stadgan.
- **Återkallelse-fönster:** tillåtet t.o.m. `PUBLICERAT`. Efter att kallelse gått ut hör förslaget till stämman.

## Relationer

- Styrelseförslag ↔ underliggande `STYRELSEBESLUT` (1..1, obligatorisk).
- Styrelseförslag ↔ `AgendaItem` — skapas automatiskt vid `PUBLICERAT`.
- Styrelseförslag ↔ `Decision` — stämmans avgörande.
- Styrelseförslag ↔ `Stadgaparagraf` — 1..n.
- Styrelseförslag ↔ Motion — kan grupperas med motion som rör samma fråga.

## Publicering

| Status | Publik | Medlem | Styrelse |
|---|---|---|---|
| INITIERAT | — | — | läs + skriv |
| ANTAGET | — | — | läs |
| PUBLICERAT | **läs** | **läs** | läs |
| PÅ_STÄMMA | läs | läs | läs |
| AVGJORT | **läs** | läs | läs |

Publik vy visar förslagstext + resultat. Beredningshistoriken (utkasts-versioner) är intern.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| ANTAGET | Styrelsen (kvittens), revisorn (informativ) |
| PUBLICERAT | Alla medlemmar (via kallelseutskick) |
| AVGJORT | Styrelsen, ev. berörd part |

## Textbibliotek

- **Förslags-skelett** — *"Styrelsen yrkar att stämman beslutar [konkret yrkande]. Som grund anförs [stadgaparagraf och saklig motivering]. Konsekvenser: [budget, tidsram, andra effekter]."*
- **Återkallelse-meddelande** — *"Styrelsen har vid sitt möte den DD/MM beslutat att återkalla styrelseförslag NN. Skäl: [saklig motivering]. Förslaget kommer inte att tas upp på stämman den DD/MM."*
- **Stämmoresultat (intern)** — *"Stämman avgjorde styrelseförslag NN den DD/MM: [resultat]. Protokollsutdrag: [länk]. Verkställighet: [vad styrelsen behöver göra härnäst]."*

## GDPR-hänsyn

Styrelseförslag är genomgående publika från `PUBLICERAT`. Inga personuppgifter i själva förslaget normalt — om förslaget rör enskild medlem (t.ex. nämnda personnamn) gäller samma försiktighet som i motion. Bevaras långsiktigt som governance-historik; ingen särskild gallring.

## Hot och försvar

Se [threats.md](../threats.md) — särskilt:

- **Egenintressen och jäv:** styrelseledamöter som föreslår åtgärder som gynnar dem själva → obligatorisk jäv-deklaration vid det underliggande styrelsebeslutet ([core-concepts.md#jäv](../core-concepts.md#jäv-conflict-of-interest)).
- **Maktkoncentration:** styrelsen kan använda förslagsrätten för att kringgå motionsfristen → publik historik visar mönster över tid; revisor ser löpande.

## Öppna frågor

- `[ÖPPEN]` Får ett styrelseförslag dras tillbaka *efter* `PUBLICERAT` om styrelsen ändrar uppfattning innan stämman? Förslag: nej, men styrelsen kan på stämman yrka avslag på sitt eget förslag — transparent och spårbart.
- `[ÖPPEN]` Krävs enhälligt styrelsebeslut för att lägga fram styrelseförslag, eller räcker majoritet? Förslag: majoritet (samma som vanliga styrelsebeslut), reserverande ledamöters position synlig i underlaget.
