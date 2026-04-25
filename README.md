# Attendify

Smart, secure, and scalable attendance management powered by geofencing, liveness-aware face verification, and fraud-resistant backend validation.

## 🚀 Overview
Attendify is a full-stack attendance platform designed to reduce proxy attendance and improve trust in classroom check-ins. It combines:
- Location verification (GPS + geofence + anomaly checks)
- Face recognition with liveness safeguards
- Backend-driven validation and replay protection
- Queue-based processing for high-concurrency spikes

---

## 1) Fraud Prevention & Security (Core Value Proposition)

### Q: GPS spoofing — how do you handle sophisticated mock-location apps?
**A:**
- **Native API checks:** On supported mobile clients, Attendify checks mock-provider indicators (for example, `isMock` location flags) to identify tampered GPS signals.
- **IP cross-referencing:** Submitted coordinates are cross-checked against approximate IP geolocation to detect integrity mismatches.
- **Velocity heuristics:** A time-to-travel rule flags impossible movement between distant check-ins within unrealistic time windows.

### Q: Biometric bypassing — how do you prevent photo/screen attacks?
**A:**
- **Randomized liveness challenges:** Users are prompted with micro-actions (blink, smile, slight head turn) instead of accepting a static image.
- **Temporal validation:** A short frame sequence is analyzed for motion/depth consistency to verify a live 3D subject.
- **Identity + liveness sync:** Identity matching (YOLO/FaceNet pipeline) is accepted only when temporal liveness checks pass, reducing high-resolution replay/display attacks.

### Q: Token security — can success requests be replayed?
**A:**
- **Backend-driven success:** The client never submits a “success” flag. It only submits an encrypted payload (image + signed GPS + timestamp).
- **Replay prevention:** Every request includes a unique, short-lived nonce and timestamp, making packet reuse invalid.
- **JWT issuance post-validation:** A short-lived JWT is issued only after backend verification, preventing forged client confirmations.

---

## 2) Computer Vision & ML (YOLO + Face Recognition)

### Q: In-the-wild reliability — crowded classrooms and inconsistent lighting?
**A:**
- **Bounding-box filtering:** A two-stage filter isolates the largest, most central face and suppresses background faces.
- **Preprocessing:** Histogram equalization normalizes illumination across varied classroom conditions.
- **ROI-first inference:** Only the isolated face crop is sent to recognition, reducing noise and improving accuracy.

### Q: Edge vs cloud — where does inference happen and how is tampering avoided?
**A:**
- **Hybrid execution:** Face detection and liveness run on edge (TensorFlow.js) to validate capture quality before upload.
- **Payload optimization:** The client crops/compresses to a small grayscale face fragment (KB-scale), not full-size images.
- **Server-side identity recognition:** Final identity matching runs on Python backend to protect model weights and core logic.

---

## 3) Geofencing & Location Services

### Q: GPS drift indoors — how do you balance false rejects vs fraud risk?
**A:**
- **Dynamic radius:** Geofence tolerance adapts using the Geolocation API accuracy metric (for example, temporary widening under poor accuracy).
- **Multi-factor proximity:** On low-confidence GPS, Attendify can validate campus network context (SSID/IP range) as a secondary signal.
- **Smoothing:** Multiple pings over a short interval (~3 seconds) are averaged to remove reflection-induced jumps.

### Q: API cost/performance — how do you avoid expensive geocoding calls?
**A:**
- **Haversine checks on backend:** Node.js computes distance directly against static classroom coordinates.
- **No reverse-geocoding dependency in attendance path:** Avoids per-check-in third-party API calls.
- **Coordinate caching:** Classroom coordinate metadata is cached (Redis) to minimize repeated DB reads.

---

## 4) Full-Stack & Database Architecture

### Q: Morning-rush concurrency — 2,000 students at once?
**A:**
- **Message queuing:** BullMQ (Redis) buffers check-ins so API ingestion remains responsive.
- **Async workers:** Background workers perform ML validation and MongoDB writes at controlled throughput.
- **Stateless horizontal scaling:** Multiple Node.js API instances can scale out during peak windows.

### Q: MERN state integrity — what if face passes but GPS fails?
**A:**
- **Atomic pending-attempt model:** A `PendingAttempt` document is created first; status becomes `Present` only when both `Location_Verified` and `Face_Verified` are true.
- **No partial commits:** Final attendance is not committed until the full validation chain completes.
- **Reason-coded failures:** Failed attempts are stored with explicit codes (e.g., `FAIL_LOCATION`) for analytics and fraud reporting.

---

## 5) Deployment & Scalability

### Q: Why Firebase + MongoDB together?
**A:**
- **Polyglot persistence:** MongoDB stores core domain data (students/classes/attendance state), while Firebase supports real-time delivery patterns.
- **Push notifications (FCM):** Students receive immediate updates when pending attendance is approved.
- **Temporary encrypted media storage:** Firebase Storage is used for short-lived encrypted check-in media to keep operational DB size lean.

---

## 6) Behavioral / L2 Technical Guidance Mindset

### Q: A junior suggests removing GPS checks to improve UX — what do you do?
**A:**
- **Lead with evidence:** Use fraud logs to show the proportion of blocked proxy attempts attributable to location mismatch signals.
- **Coach on trade-offs:** Explain why L2 ownership means balancing user friction against platform integrity.
- **Improve, don’t remove:** Keep security controls while improving UX (better status feedback, progressive retries, network-assisted fallback checks).

---

## 🏗️ Build and Run

### Frontend
```sh
docker build -t my-react-app .
docker run -p 80:80 my-react-app
```

### Backend
```sh
docker build -t facereco-backend .
docker run -p 5000:5000 facereco-backend
```

### Full Stack (Docker Compose)
```sh
docker-compose up --build
```

---

## ✅ Key Outcomes
- Strong reduction in proxy attendance attempts through multi-layer validation
- Better trustworthiness than single-factor attendance systems
- Production-oriented architecture with queueing, caching, and horizontal scalability
