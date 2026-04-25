# Attendify

A full-stack facial-recognition attendance system with a React frontend and a FastAPI backend, containerized with Docker for simple local deployment.

Attendify allows users to:

- Register a face profile
- Mark **IN** attendance using face verification
- Mark **OUT** attendance using face verification
- Export attendance logs as a ZIP file

---

## ✨ Features

- **Face-based authentication** using `face_recognition` embeddings
- **User registration** from uploaded face images
- **Login / Logout attendance marking**
- **Daily CSV attendance logs** (`IN`/`OUT` events)
- **Downloadable attendance archive** (`/get_attendance_logs`)
- **Dockerized frontend and backend**
- **CORS-enabled API** for frontend-backend communication

---

## 🧱 Tech Stack

### Frontend
- React 18
- Axios
- React Scripts

### Backend
- FastAPI
- OpenCV (`opencv-python-headless`)
- `face-recognition` (dlib-based embeddings)
- Uvicorn
- Python Multipart (file uploads)

### Deployment / Runtime
- Docker
- Docker Compose

---

## 📁 Project Structure

```text
Attendify/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── public/
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── docker-compose.yml
└── README.md
```

---

## ⚙️ How It Works

1. **Register User**
   - Endpoint: `POST /register_new_user`
   - Input: face image + username (`text`)
   - Output:
     - image saved in `./db/<username>.png`
     - embedding saved in `./db/<username>.pickle`

2. **Login**
   - Endpoint: `POST /login`
   - Input: face image
   - Backend compares uploaded embedding with stored embeddings.
   - On successful match, writes:
     - `<username>,<timestamp>,IN` to `./logs/<YYYYMMDD>.csv`

3. **Logout**
   - Endpoint: `POST /logout`
   - Same recognition flow as login
   - On successful match, writes:
     - `<username>,<timestamp>,OUT`

4. **Export Logs**
   - Endpoint: `GET /get_attendance_logs`
   - Zips the `./logs` directory and returns it as `out.zip`

---

## 🚀 Running the Application

### Option 1: Docker Compose (Recommended)

From the repository root:

```sh
docker-compose up --build
```

Services:
- Frontend: `http://localhost:80`
- Backend: `http://localhost:5000`

---

### Option 2: Run Services Separately

#### Frontend

```sh
docker build -t my-react-app -f frontend/Dockerfile .
docker run -p 80:80 my-react-app
```

#### Backend

```sh
docker build -t facereco-backend -f backend/Dockerfile .
docker run -p 5000:5000 facereco-backend
```

---

## 🔌 API Reference

### `POST /register_new_user`
Registers a new face profile.

- **Form fields**
  - `file`: image file
  - `text`: username

- **Response**
```json
{ "registration_status": 200 }
```

---

### `POST /login`
Marks attendance as `IN` after face match.

- **Form fields**
  - `file`: image file

- **Response**
```json
{ "user": "username_or_status", "match_status": true_or_false }
```

---

### `POST /logout`
Marks attendance as `OUT` after face match.

- **Form fields**
  - `file`: image file

- **Response**
```json
{ "user": "username_or_status", "match_status": true_or_false }
```

---

### `GET /get_attendance_logs`
Downloads zipped attendance logs.

- **Response**
  - `out.zip` (application/zip)

---

## 🗃️ Data & Storage

- `./db/`
  - Stores registered face images and embedding pickles
- `./logs/`
  - Stores attendance CSV files by date (`YYYYMMDD.csv`)

---

## ⚠️ Current Limitations

This repository currently implements core face-recognition attendance flow, but not all advanced anti-fraud controls yet.

Not fully implemented in current codebase:
- GPS/geofencing verification
- Liveness challenge-response (blink/smile/head movement)
- JWT-based session/attestation flow
- Anti-replay nonce pipeline
- Queue-based ingestion (BullMQ/Redis)
- Firebase notification/storage integration

---

## 🛡️ Security & Fraud-Prevention Roadmap

Based on product goals, the following enhancements are planned to strengthen fraud resistance:

1. **Location Integrity**
   - Mock-location detection
   - IP-to-location consistency checks
   - Impossible-travel velocity heuristics

2. **Biometric Hardening**
   - Liveness challenge prompts
   - Temporal frame validation against photo/screen replay

3. **Transport & Token Security**
   - Signed payloads + nonce + timestamp validation
   - Short-lived JWT issuance only after server-side verification

4. **Scalability & Reliability**
   - Redis-backed queue workers for peak attendance windows
   - Multi-factor fallback checks under low-confidence signals

---

## 🧪 Development Notes

- Backend currently allows CORS from all origins (`allow_origins=["*"]`) for easier integration during development.
- For production:
  - Restrict CORS to trusted domains
  - Add auth + rate-limiting
  - Encrypt sensitive artifacts and enforce secure storage lifecycle
  - Add robust monitoring/log retention policy

---

## 📌 Summary

Attendify provides a practical and extensible foundation for face-based attendance management with clear pathways to enterprise-grade security and scale.
