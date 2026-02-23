# Fullstack zadanie (Java + Next.js) – Car Rental Availability & Pricing

Cieľom zadania je ukázať, ako rozmýšľaš nad **dátami, edge-caseami, robustnosťou API a použiteľnosťou UI**.  
Kód môžeš písať s pomocou AI nástrojov, ale výsledok musí byť **tvoj** – hlavne rozhodnutia, vysvetlenia a správne spracovanie nečistých dát.

---

## Kontext

Máme dáta o autách a rezerváciách v požičovni. Dáta sú zámerne **nečisté** (chýbajúce polia, neznáme statusy, zlé dátumy…), aby sa ukázalo, či vieš spraviť odolné riešenie.

Dataset nájdeš v priečinku `data/`.

---

## 1) Backend (Java Spring Boot)

Implementuj endpoint:

`GET /api/availability?from=YYYY-MM-DD&to=YYYY-MM-DD`

### Vstupy
- `from` – dátum začiatku (inkluzívne)
- `to` – dátum konca (exkluzívne)

### Validácia
- Ak `from` alebo `to` chýba → HTTP 400
- Ak `from >= to` → HTTP 400
- Počet dní = `to - from` (celé dni). Ak vyjde `0` → HTTP 400

### Výstup
Endpoint vracia JSON pole položiek (jedna položka na auto):

- `carId` (string)
- `model` (string)
- `isAvailable` (boolean)
- `conflicts` (integer)
- `estimatedPrice` (number | null)

Príklad:

```json
[
  {
    "carId": "C-100",
    "model": "Skoda Octavia 1.6 TDI",
    "isAvailable": false,
    "conflicts": 2,
    "estimatedPrice": null
  }
]
```

---

## Biznis pravidlá

### A) Overlap (prekrývanie intervalov)
Rezervácia sa považuje za prekrývajúcu sa s dotazom `[from, to)` ak platí:

`reservation.from < to` **a zároveň** `reservation.to > from`

Pozn.: interval je polootvorený `[from, to)`, takže rezervácia končiaca presne v `from` sa nepočíta ako konflikt.

### B) Dostupnosť (`isAvailable`)
Auto je **nedostupné** (`isAvailable=false`), ak existuje aspoň jedna prekrývajúca sa rezervácia so statusom:
- `CONFIRMED`
- `PICKED_UP`

Rezervácie so statusom `CANCELLED` ignoruj.

Rezervácie so statusom `UNKNOWN` alebo chýbajúcim statusom:
- **neblokujú** auto (auto môže zostať dostupné),
- ale zvyšujú `conflicts` (soft-conflict).

### C) Definícia `conflicts`
`conflicts` je počet **problematických rezervácií** pre dané auto, ktoré spadajú do dotazovaného obdobia:

- **HARD conflict**: prekrýva sa a status je `CONFIRMED` alebo `PICKED_UP`
- **SOFT conflict**: prekrýva sa a status je `UNKNOWN` alebo chýba
- **DATA conflict**: rezervácia má nevalidné dátumy (`to <= from`) alebo chýba `from/to` (rátaj ako konflikt, ale neblokuj auto, bez ohľadu či sa dátumovo prekrýva)

> Inými slovami: `conflicts` = koľko varovaní/problémov vieš nájsť okolo dostupnosti auta v danom intervale.

### D) Pricing (`estimatedPrice`)
Ak je auto dostupné, vypočítaj cenu:

`estimatedPrice = days * pricePerDay`

- `pricePerDay` nájdeš v `data/cars.json` pri aute.
- Ak `pricePerDay` chýba alebo je `<= 0` → `estimatedPrice = null` a zaloguj warning.
- Ak auto nie je dostupné → `estimatedPrice = null` (nepočítaj cenu na nedostupné auto).

---

## 2) Frontend (Next.js)

Vytvor stránku `/availability`:

- dva date inputy `from`, `to`
- tlačidlo „Check availability“
- tabuľka výsledkov:
  - Model
  - Available / Not available
  - Estimated price (alebo `N/A`)
  - Conflicts (ak > 0, zvýrazni – napr. badge)

Povinné stavy:
- loading
- error (zobraz zmysluplnú hlášku z backendu)

---

## 3) Povinné: DECISIONS.md (max 1 strana)

Napíš krátky „decision log“:
- ktoré problémy v dátach si našiel,
- ako si ich riešil,
- čo by si spravil inak v produkcii (monitoring, databáza, cache, testy…).

Toto je dôležité – v dobe AI hodnotíme vlastné rozmýšľanie :)

---

## 4) Minimálne testy

Aspoň 1–2 testy v backende (napr. overlap logika alebo výpočet konfliktov).

---

## Dataset a verejné scenáre

V `data/scenarios_public.json` sú 3 verejné scenáre s očakávanými výsledkami pre rýchlu kontrolu.
Cieľom nie je „spraviť to na 100% podľa scenárov“, ale aby to bolo konzistentné s pravidlami vyššie.

---

## Odovzdanie

- link na repo (alebo zip), v prípade repa nerobit fork z tohto repa, aby si kandidáti nevideli medzi sebou riešenia!
- ako spustiť backend a frontend
- `DECISIONS.md`