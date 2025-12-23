# Registracijų sistema (formų kūrimas + vietų kontrolė + el. laiškai)

## Apžvalga
Šis projektas – **registracijų sistema**, skirta kurti ir valdyti registracijos formas su keliomis dienomis (pvz., pirmadienis–sekmadienis) ir kiekvienai dienai priskiriamomis veiklomis. Kiekviena veikla turi **laiką** ir **vietų (capacity) limitą**, o dalyviai gali registruotis per unikalų viešą nuorodos URL. Sistema automatiškai valdo vietų užimtumą, siunčia patvirtinimus el. paštu ir suteikia administratoriui pilną registracijų peržiūrą bei valdymą.

---

## Pagrindinės galimybės

### 1) Formų kūrimas ir valdymas
- Administratorius gali sukurti naują formą, nurodydamas:
  - formos pavadinimą
  - formos datą / laikotarpį (pagal poreikį)
  - dienas (pirmadienis, antradienis, …) ir jų veiklas
- Kiekvienai dienai galima pridėti **tiek veiklų, kiek reikia**, pvz.:
  - 10:00 – Sensorika (8 vietos)
  - 12:00 – Judesio veikla (10 vietų)
  - 14:00 – Kūrybinės dirbtuvės (6 vietos)
- Administratorius gali:
  - peržiūrėti visas sukurtas formas viename sąraše
  - redaguoti formą (pavadinimą, datas, dienas, veiklas, vietas)
  - ištrinti formą

### 2) Unikalus viešas formos linkas (UUID)
- Kiekviena forma automatiškai gauna unikalų **UUID** ir atskirą viešą URL.
- Šį URL galima:
  - atsidaryti registracijai
  - dalintis su kitais (pvz., tėvams / dalyviams)
- Linkas veikia kaip „public entry point“ į konkrečios formos registraciją.

### 3) Vietų kontrolė (capacity) ir pasirinkimų ribojimas
- Dalyvis gali pasirinkti tik tas veiklas, kuriose dar yra laisvų vietų.
- Jei veikla užsipildo:
  - ji tampa **nepasirenkama** (disabled / hidden pagal UI logiką)
  - sistema neleidžia pateikti registracijos į užpildytą veiklą net ir bandant apeiti UI (server-side tikrinimas)
- Vietų likutis atnaujinamas pagal registracijų kiekį (skaičiuojant pasirinkimus prie konkrečios veiklos).

### 4) Registracija (public flow)
- Dalyvis atsidaro formos URL ir užpildo registraciją:
  - kontaktinė informacija (pagal formos reikalavimus)
  - pasirenkamos veiklos pagal dienas ir laikus
- Po sėkmingos registracijos:
  - dalyvis gauna **patvirtinimo el. laišką**
  - laiške pateikiama aiški suvestinė: į ką ir kada užsiregistravo

### 5) Administratoriaus registracijų valdymas
- Administratorius gali peržiūrėti visus užsiregistravusius:
  - sąrašas su paieška / filtrais
  - filtravimas pagal pasirinktą veiklą (pvz., matyti tik „Sensorika 10:00“)
- Galima atlikti veiksmus su registracijomis:
  - redaguoti
  - ištrinti
  - peržiūrėti registracijos detales

### 6) Komunikacija ir atšaukimas
- Administratorius gali greitai susisiekti su dalyviu (pvz., „parašyti iškart“ veiksmu).
- Dalyvio patvirtinimo laiške yra galimybė **atšaukti registraciją**:
  - atšaukus – atsilaisvina vietos atitinkamose veiklose
- Administratorius taip pat gali atlikti atšaukimą iš admin pusės.

---

## Architektūra (aukšto lygio)

```text
Dalyvis (public)
  └─ Viešas formos URL (UUID)
        ├─ Formos peržiūra (dienos + veiklos + vietos)
        ├─ Pasirinkimai (tik jei yra laisvų vietų)
        └─ Registracijos pateikimas
                ↓
Server-side (Next.js)
  ├─ Formos ir veiklų gavimas iš DB
  ├─ Vietų patikra (capacity) registracijos metu
  ├─ Registracijos įrašymas į DB (Supabase)
  └─ Patvirtinimo el. laiško siuntimas (Nodemailer)
                ↓
Admin panelė (protected)
  ├─ Prisijungimas per Google OAuth (NextAuth)
  ├─ Prieigos kontrolė pagal admins lentelę (allowlist)
  ├─ Formų CRUD (kurti/redaguoti/trinti)
  ├─ Formų veiklų valdymas (dienos/valandos/vietos)
  ├─ Registracijų peržiūra + filtrai
  ├─ Registracijų redagavimas/šalinimas/atšaukimas
  └─ Komunikacija su dalyviais
```

---

## Duomenų bazė

### Pagrindinės registracijų lentelės (4)
Sistemos registracijų dalis modeliuojama per 4 pagrindines lenteles:

1) **`forms`**  
- registracijos formos (pavadinimas, data, UUID ir pan.)

2) **`form_activities`**  
- visos konkrečios formos veiklos (diena, laikas, vietų skaičius / capacity)  
- **ryšys:** `form_activities.form_id` → `forms.id` (viena forma turi daug veiklų)

3) **`registrations`**  
- dalyvių registracijos į konkrečią formą (kontaktai, statusas, created_at ir pan.)  
- **ryšys:** `registrations.form_id` → `forms.id` (viena forma turi daug registracijų)

4) **`registration_activities`**  
- jungiamoji lentelė, nurodanti, kokias veiklas pasirinko registracija  
- **ryšiai:**
  - `registration_activities.registration_id` → `registrations.id`
  - `registration_activities.form_activity_id` → `form_activities.id`

### Admin prieigos lentelė
5) **`admins`**  
- leidžiamų administratorių sąrašas (pvz., el. pašto adresai)
- naudojama kaip **allowlist**, pagal kurią suteikiama prieiga prie admin panelės

### Ryšių logika (santrauka)
- **Forma** (`forms`) turi daug **formos veiklų** (`form_activities`).
- **Forma** (`forms`) turi daug **registracijų** (`registrations`).
- **Registracija** (`registrations`) turi daug pasirinktų **registracijos veiklų** (`registration_activities`).
- `registration_activities.form_activity_id` visada rodo į konkrečią `form_activities` eilutę, todėl:
  - galima tiksliai skaičiuoti užimtumą (kiek pasirinkimų turi konkreti veikla)
  - galima filtruoti registracijas pagal veiklą
  - galima užtikrinti, kad pasirenkamos veiklos priklauso tai pačiai formai

*(Detalios stulpelių schemos ir vidiniai identifikatoriai viešai neaprašomi.)*

---

## Autentifikacija ir prieigos kontrolė
- Admin prisijungimui naudojamas **Google OAuth**
- Autentifikacija realizuota per **NextAuth**
- Prisijungęs vartotojas gauna prieigą tik tuo atveju, jei jo el. paštas yra `admins` lentelėje (DB allowlist principas)

---

## Tech stack
- **Next.js** `15.5.2`
- **React** `19.1.0`
- **Tailwind CSS** `v4`
- **Supabase** (`@supabase/supabase-js`) – duomenų bazė
- **NextAuth** – Google OAuth admin daliai
- **Nodemailer** – transakciniai el. laiškai
- **UUID** – unikalios formų nuorodos
- UI / UX:
  - **Framer Motion**
  - **React Icons**

---

## End-to-end srautas

### Dalyvio registracija
1. Dalyvis atsidaro formos URL su UUID
2. Sistema parodo dienas, veiklas, laikus ir laisvas vietas
3. Dalyvis pasirenka veiklas (tik jei yra vietų)
4. Pateikia registraciją
5. Serveris patikrina vietas ir įrašo registraciją į DB:
   - į `registrations`
   - pasirinktus pasirinkimus į `registration_activities`
6. Išsiunčiamas patvirtinimo el. laiškas su suvestine ir atšaukimo galimybe

### Administratoriaus darbas
1. Prisijungia per Google OAuth
2. Kuria / redaguoja formas ir jų veiklas (`forms` + `form_activities`)
3. Stebi registracijas, filtruoja pagal veiklas
4. Redaguoja / šalina / atšaukia registracijas, susisiekia su dalyviais
5. Vietų likutis atnaujinamas pagal `registration_activities` įrašus
