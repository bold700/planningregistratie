# Planning App Prototype

Een slimme planning-app met **automatische reistijdberekening** en **live verkeer checks**.

## 🎯 Kernfunctionaliteit

✅ **Agenda weergave** (dag/week)  
✅ **Afspraken toevoegen** met automatische reistijdberekening  
✅ **Reistijdblokken** als aparte afspraken (declarabel!)  
✅ **Live verkeer** (1 uur voor vertrek)

## 🚀 Quick Start

```bash
# Installeer dependencies
npm install

# Start development server
npm start
```

Zie **SETUP.md** voor volledige installatie-instructies.

## 🏗️ Tech Stack

- **React** + date-fns
- **Firebase** (Firestore + Functions + Hosting)
- **Google Maps Directions API**

## 📂 Project Structuur

```
planning-app/
├── src/
│   ├── components/
│   │   ├── AppointmentForm.js    # Formulier nieuwe afspraak
│   │   ├── DayView.js             # Dagweergave timeline
│   │   └── WeekView.js            # Weekoverzicht
│   ├── services/
│   │   └── appointmentService.js  # Reistijd + CRUD logica
│   ├── App.js                     # Main component
│   ├── App.css                    # Styling
│   └── firebase.js                # Firebase config
├── functions/
│   └── index.js                   # Cloud Function (verkeer check)
├── public/
├── firebase.json
├── firestore.rules
└── SETUP.md                       # Installatie-instructies
```

## 🔑 Key Features Uitgelegd

### 1. Automatische Reistijdberekening

Bij het toevoegen van een afspraak met locatie:

```javascript
// Startpunt bepalen:
// 1. Vorige afspraak van die dag (met locatie)
// 2. Thuisadres: Zilveren Florijnlaan 7, Utrecht

// Google Maps berekent reistijd
const travelTime = await calculateTravelTime(origin, destination);

// Afgerond op 15 minuten
const rounded = Math.ceil(travelTime / 15) * 15;

// Resultaat: 2 afspraken
// - 13:15 🚗 Reis naar Chanel (45 min)
// - 14:00 📅 Meeting Chanel (60 min)
```

### 2. Live Verkeer Check

Cloud Function draait elke 15 minuten:

```javascript
// Check reistijdblokken die binnen 1 uur starten
const upcomingTravels = appointments.filter(
  apt => apt.isTravelBlock && 
  isWithinNextHour(apt.startTime)
);

// Haal live verkeersinformatie
const liveTraffic = await getMapsDataWithTraffic();

// Update als verschil ≥ 15 minuten
if (Math.abs(new - old) >= 15) {
  updateTravelBlock(newStartTime);
}
```

### 3. Declarabele Reistijd

Beide blokken krijgen zelfde `projectId`:

```javascript
// Hoofdafspraak
{ 
  projectId: "chanel-ux-review",
  isTravelBlock: false 
}

// Reistijdblok (aparte factuurregel!)
{ 
  projectId: "chanel-ux-review",
  isTravelBlock: true 
}
```

## 🔄 Toekomstige Integraties

### Google Calendar Sync
- Bidirectionele sync (lezen + schrijven)
- Conflict-resolutie

### Urenregistratie Koppeling
```
Afspraak → Uren (automatisch)
Uren → Agenda (automatisch)
```

### Taken-app Integratie
```
"Wat is mn planning?" 
→ Afspraken + Taken in 1 overzicht
```

## 📊 Data Flow

```
┌─────────────────┐
│ Nieuwe afspraak │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ Vorige locatie ophalen  │
│ (of thuisadres)         │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Google Maps API         │
│ → Reistijd berekenen    │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ 2 afspraken in Firestore│
│ 1. Reistijdblok         │
│ 2. Hoofdafspraak        │
└─────────────────────────┘

┌──────────────────┐
│ Elke 15 minuten  │
└────────┬─────────┘
         │
         ▼
┌─────────────────────────┐
│ Check aankomende        │
│ reistijdblokken         │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Live verkeer ophalen    │
│ → Update indien nodig   │
└─────────────────────────┘
```

## 🧪 Testen

### Lokaal testen
```bash
npm start  # http://localhost:3000
```

### Firebase emulator
```bash
firebase emulators:start
```

### Deploy
```bash
npm run build
firebase deploy
```

## 📝 License

Prototype voor BOLD700 BV - Niet voor productie zonder testing!

## 🤝 Volgende Stappen

1. Test met echte data en afspraken
2. Voeg Google Calendar sync toe
3. Koppel aan urenregistratie-app
4. Implementeer taken-integratie
5. Auth flow + multi-user support

---

**Happy planning!** 🗓️✨
