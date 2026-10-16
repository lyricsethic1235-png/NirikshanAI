# NirikshanAI 🕳️
> **AI-Assisted Civic Infrastructure Inspection & GIS Dispatch Platform**

NirikshanAI is a production-grade full-stack civic inspection system built for municipal road engineers and field officers. It combines on-site smartphone hazard capture with automated GPS, recorded officer voice notes, and AI defect detection using **Roboflow Universe**, **Local YOLO (best.pt)**, or **Simulated Demo Mode**.

---

## 🚀 Quick Start (Running the Application)

### 1. Start the FastAPI Backend
Open a terminal in the root directory:
```bash
# Using uv (fastest)
uv run --directory backend --with fastapi --with uvicorn --with python-multipart --with python-dotenv --with requests --with pillow uvicorn main:app --host 127.0.0.1 --port 8000 --reload

# Or using standard python virtual environment:
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python main.py
```
*Backend runs on `http://127.0.0.1:8000` with interactive docs at `http://127.0.0.1:8000/docs`.*

### 2. Start the Vite Frontend
In another terminal:
```bash
npm install
npm run dev
```
*Frontend runs on `http://localhost:5173/`.*

### Test the Field Officer Portal on a Phone

Camera and microphone access require HTTPS on mobile. For a temporary same-device test:

1. Keep the backend running on port `8000` and start Vite with `npm run dev`.
2. In another terminal, run `npx --yes localtunnel --port 5173`.
3. Open the generated `https://...loca.lt` URL on your phone and allow camera/microphone access.

The tunnel exposes the sign-in page to the public internet. Use demo data only, keep the tunnel running only while testing, and stop it with `Ctrl+C` afterward. Do not use it for real officer or citizen data.

### Portal Accounts

The app has separate Admin and Field Officer accounts. Copy `backend/.auth.env.example` to `backend/.auth.env` and configure both password hashes, a random `AUTH_SECRET`, and the officer's `OFFICER_ID`. The real auth file is gitignored. An officer Gmail address can be used as `OFFICER_EMAIL`, but its password must be a separate NirikshanAI app password; never enter or store a Google account password here. These app accounts do not verify Gmail ownership or use Google OAuth.

---

## 🧠 Switchable AI Detector Modes (`backend/.env`)

The detector can be changed at any time by editing `backend/.env`:

```env
# Choose one of: "roboflow" | "local" | "demo"
DETECTOR_MODE=roboflow

# If using "roboflow":
ROBOFLOW_API_KEY=your_roboflow_api_key_here
ROBOFLOW_MODEL_ID=pothole-6-detect/1

# If using "local":
LOCAL_MODEL_PATH=models/best.pt
```

### Automatic Fallback Protection
If the selected mode fails (for example, if your Roboflow API key is missing or your `best.pt` file is not found):
- The backend **automatically falls back to Demo mode**.
- The frontend shows a **visible warning badge**:
  > `⚠️ DETECTOR MODE FALLBACK: Requested 'roboflow' failed (Invalid API key). Fell back to Demo mode.`
- Inspections and severity calculations continue without crashing.

---

## 🔑 How to Get a Roboflow API Key & Pothole Model (Step-by-Step)

Roboflow Universe has free, pre-trained computer vision models for potholes and road defects.

### Step 1: Create a Free Roboflow Account
1. Go to **[https://app.roboflow.com/](https://app.roboflow.com/)**.
2. Click **Sign Up** (you can sign up for free using Google, GitHub, or email).

### Step 2: Copy Your Roboflow Private API Key
1. Once logged in, click your workspace name or avatar in the top-left or top-right corner.
2. Go to **Settings** → **API Keys** (or go to [https://app.roboflow.com/settings/api](https://app.roboflow.com/settings/api)).
3. Under **Roboflow API Key**, click **Copy Private Key**.
4. Paste it into your `backend/.env` file:
   ```env
   ROBOFLOW_API_KEY=your_actual_key_here
   ```

### Step 3: Pick a Pothole Model from Roboflow Universe
1. Go to **[https://universe.roboflow.com/search?q=pothole](https://universe.roboflow.com/search?q=pothole)**.
2. Select any public pothole dataset that has a trained model (for example: **[pothole-6-detect](https://universe.roboflow.com/anuj-chaudhary-eqf5b/pothole-6-detect)** or **[pothole-detection-yolov8](https://universe.roboflow.com/roboflow-100/pothole-detection-yolov8)**).
3. Look at the URL or the model overview page:
   - Notice the project ID and version number (e.g. `pothole-6-detect/1`).
4. Paste that model ID into your `backend/.env`:
   ```env
   ROBOFLOW_MODEL_ID=pothole-6-detect/1
   ```
5. Set `DETECTOR_MODE=roboflow` and restart your backend.

---

## 🗺️ How to Get and Enable a Google Maps API Key (Step-by-Step)

NirikshanAI includes a native Google Maps interface with satellite aerial photography and road views.

### Step 1: Go to Google Cloud Console
1. Go to **[https://console.cloud.google.com/](https://console.cloud.google.com/)**.
2. Sign in with your Google account.
3. In the top bar, click the project selector dropdown and click **New Project** (e.g. named `NirikshanAI`).

### Step 2: Enable the Maps JavaScript API
1. In the search bar at the top, type **"Maps JavaScript API"** and select it.
2. Click the blue **Enable** button.
   *(Direct link: [https://console.cloud.google.com/apis/library/maps-backend.googleapis.com](https://console.cloud.google.com/apis/library/maps-backend.googleapis.com))*

*(Optional for address lookup)*:
Search for **"Geocoding API"** and click **Enable** so officer GPS coordinates convert to street names.

### Step 3: Create Your API Key
1. In the left navigation menu, go to **APIs & Services** → **Credentials**.
2. Click **+ Create Credentials** at the top → choose **API Key**.
3. A popup will appear showing: `Your API key: AIzaSy...`.
4. Copy this key.

### Step 4: Add the Key to NirikshanAI
Open the `.env` file in the project root and add your key:
```env
VITE_GOOGLE_MAPS_API_KEY=AIzaSyYourActualKeyHere
```
Save the file and refresh your browser.

> **Note:** If your Google key does not have billing enabled or hasn't finished activating, NirikshanAI provides a 1-click **OpenStreetMap / Satellite** toggle so your inspection map always works.

---

## 📂 Project Structure

```text
Nirakshan AI/
├── backend/
│   ├── models/                # Place your local best.pt here
│   ├── .env                   # DETECTOR_MODE, ROBOFLOW_API_KEY, ROBOFLOW_MODEL_ID
│   ├── detector.py            # Switchable detector engine (Local, Roboflow, Demo)
│   ├── main.py                # FastAPI endpoints (/api/detect, /api/inspections)
│   └── requirements.txt
├── src/
│   ├── components/
│   │   ├── OfficerCapture.jsx # Mobile-first Field Officer Portal with camera & audio
│   │   ├── AdminMapDashboard.jsx # Dual-engine GIS Map (Google Maps + OSM) & Metrics
│   │   ├── PriorityQueue.jsx  # High-to-low prioritized repair dispatch table
│   │   ├── InspectionDetailModal.jsx # Full inspection dossier view
│   │   └── Navbar.jsx
│   ├── services/
│   │   └── api.js             # API service with offline fallback
│   └── App.jsx
├── .env                       # Frontend Google Maps key
└── package.json
```

---

## 🛡️ License
Built for municipal civic works departments, road maintenance squads, and urban infrastructure monitoring.
