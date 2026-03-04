# Planning App Prototype - Setup Instructies

## Wat zit er in dit prototype?

✅ **Agenda weergave** (dag/week overzicht)
✅ **Afspraak toevoegen** met automatische reistijdberekening  
✅ **Reistijdblokken** als aparte afspraken in de agenda
✅ **Live verkeer check** + automatische update (1 uur voor vertrek)

---

## Technische Stack

- **Frontend**: React + date-fns voor datum-handling
- **Backend**: Firebase (Firestore + Functions + Hosting)
- **API's**: 
  - Google Maps Directions API (reistijd + live verkeer)
  - Google Calendar API (voor latere sync)

---

## Setup Stappen

### 1. Firebase Project Aanmaken

```bash
# Installeer Firebase CLI
npm install -g firebase-tools

# Login bij Firebase
firebase login

# Maak nieuw project aan via Firebase Console
# https://console.firebase.google.com
```

In Firebase Console:
1. Maak nieuw project: **planning-app-prototype**
2. Enable **Firestore Database** (start in test mode)
3. Enable **Cloud Functions**
4. Enable **Firebase Hosting**
5. Enable **Authentication** → Google Sign-In

### 2. Firebase Config Invullen

Kopieer je Firebase config van:
`Project Settings → Your apps → Firebase SDK snippet`

Update **src/firebase.js**:
```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### 3. Google Maps API Setup

**Prototype (demo.html):** Vul je API key in `config.js`:
```javascript
window.PLANNING_APP_CONFIG = {
  googleMapsApiKey: 'JOUW_API_KEY'
};
```

**Firebase app:** De API key staat in `src/firebase.js`:
```javascript
export const GOOGLE_MAPS_API_KEY = 'AIzaSyD1cTqK8IwuOxQNWQmmbDpk71kkmgAfZqB0';
```

Zorg dat deze API key de volgende APIs enabled heeft in Google Cloud Console:
- **Places API** (voor adres-autocomplete)
- **Directions API** of **Routes API** (voor reistijdberekening)
1. Ga naar: https://console.cloud.google.com
2. APIs & Services → Library
3. Zoek "Directions API" → Enable
4. Voeg ook toe: "Distance Matrix API" (voor live verkeer)

### 4. Installeer Dependencies

```bash
# Frontend
npm install

# Firebase Functions
cd functions
npm install
cd ..
```

### 5. Firebase Functions Config

Stel de Google Maps API key in voor Functions:

```bash
firebase functions:config:set google.maps_api_key="AIzaSyD1cTqK8IwuOxQNWQmmbDpk71kkmgAfZqB0"
```

### 6. Local Development

```bash
# Start React app
npm start

# In een andere terminal: start Firebase emulators (optioneel)
firebase emulators:start
```

App draait op: **http://localhost:3000**

### 7. Deploy naar Firebase

```bash
# Build de React app
npm run build

# Deploy alles (hosting + functions)
firebase deploy

# Of alleen hosting:
firebase deploy --only hosting

# Of alleen functions:
firebase deploy --only functions
```

---

## Hoe het werkt

### 1. **Afspraak Toevoegen**

Klik op **"+ Afspraak"** knop:
- Vul titel, locatie, start/eindtijd in
- Optioneel: Project ID (voor koppeling met urenregistratie)

**Wat gebeurt er achter de schermen:**
1. App zoekt naar vorige afspraak van die dag met locatie
2. Als die er niet is: gebruikt thuisadres (Zilveren Florijnlaan 7)
3. Google Maps berekent reistijd (met verkeer)
4. Reistijd wordt afgerond op 15 minuten
5. **2 afspraken** worden aangemaakt:
   - 🚗 Reistijdblok (bijv. "13:15 - 14:00 Reis naar Chanel")
   - 📅 Hoofdafspraak (bijv. "14:00 - 15:00 Meeting Chanel")

### 2. **Live Verkeer Check**

**Cloud Function draait elke 15 minuten:**
- Checkt alle reistijdblokken die **binnen 1 uur** starten
- Haalt actuele verkeersinformatie op via Google Maps
- Als verschil ≥ 15 minuten: update reistijdblok automatisch
- Description wordt geupdate: **"LIVE UPDATE"**

**Handmatige check:**
- Klik op 🔄 icoon rechtsboven
- Checkt direct alle aankomende reistijdblokken

### 3. **Dag/Week Overzicht**

**Dag view:**
- Timeline van 6:00 - 22:00
- Afspraken op exacte tijden
- Reistijdblokken in geel/oranje

**Week view:**
- 7 kolommen (ma-zo)
- Compacte lijst per dag
- Vandaag gemarkeerd met blauwe border

---

## Firestore Data Structuur

### Collection: `appointments`

```javascript
{
  // Hoofdafspraak
  title: "Meeting Chanel",
  location: "P.C. Hooftstraat 123, Amsterdam",
  startTime: Timestamp,
  endTime: Timestamp,
  description: "Bespreking Q1 doelen",
  projectId: "project-123",  // Voor urenregistratie-koppeling
  isTravelBlock: false,
  createdAt: Timestamp
}

{
  // Reistijdblok
  title: "🚗 Reis naar Meeting Chanel",
  location: "Van Zilveren Florijnlaan 7 naar P.C. Hooftstraat 123",
  startTime: Timestamp,
  endTime: Timestamp,
  description: "Reistijd: 45 min (42 km)",
  relatedAppointmentId: "appointment-id",  // Link naar hoofdafspraak
  isTravelBlock: true,
  travelInfo: {
    origin: "Zilveren Florijnlaan 7, 3541 HA Utrecht",
    destination: "P.C. Hooftstraat 123, Amsterdam",
    durationSeconds: 2700,
    distanceMeters: 42000
  },
  projectId: "project-123",  // Zelfde project = declarabel
  createdAt: Timestamp,
  updatedAt: Timestamp  // Bij live verkeer update
}
```

---

## Volgende Stappen (Roadmap)

### ✅ Fase 1-3: KLAAR
- Agenda weergave ✓
- Afspraak toevoegen met reistijd ✓
- Live verkeer check ✓

### 🚧 Fase 4: Google Calendar Sync
- OAuth flow voor Google Calendar
- Bidirectionele sync (lezen + schrijven)
- Conflict-resolutie

### 🚧 Fase 5: Outlook Calendar Sync
- Microsoft Graph API
- Zelfde logica als Google Calendar

### 🚧 Fase 6: Koppeling Urenregistratie
- API naar https://urenregistratie-beta.vercel.app
- Automatisch uren boeken bij afspraak
- Reverse sync: uren → agenda

### 🚧 Fase 7: Taken-app Integratie
- API naar http://127.0.0.1:3750
- Gecombineerd overzicht: afspraken + taken

---

## Troubleshooting

### "Google Maps API error: REQUEST_DENIED"
→ Check of Directions API enabled is in Google Cloud Console

### "Firebase: Missing or insufficient permissions"
→ Update Firestore rules of gebruik test mode

### Cloud Function werkt niet
→ Check Firebase Functions logs: `firebase functions:log`

### Reistijd wordt niet berekend
→ Check console voor errors, verifieer API key in Firebase config

---

## Demo Data Toevoegen

Voor testing kun je handmatig afspraken toevoegen via de app, of via Firestore Console:

```javascript
// Voorbeeld afspraak
{
  title: "UX Review bij Chanel",
  location: "P.C. Hooftstraat 123, 1071 DC Amsterdam",
  startTime: Firebase.Timestamp.fromDate(new Date('2024-03-15T14:00:00')),
  endTime: Firebase.Timestamp.fromDate(new Date('2024-03-15T16:00:00')),
  description: "Q1 design system review",
  projectId: null,
  isTravelBlock: false,
  createdAt: Firebase.Timestamp.now()
}
```

---

## Contact

Voor vragen of issues: check de code comments of Firebase logs.

**Volgende demo:** Bidirectionele sync met urenregistratie-app! 🚀
