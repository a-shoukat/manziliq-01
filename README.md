# MANZILIQ — Unified Property & Society Management Platform

> **Final Year Project** — UET Lahore, Punjab Campus (Session 2023)
> Team: Ayesha Shoukat (2023-CS-512) · Eman Khan (2023-CS-550) · Mehak Eman (2023-CS-541)

MANZILIQ is a unified real-estate platform for Pakistan that combines a **property marketplace**
with **housing-society management**, **dealer CRM**, **booking & installment tracking**,
**AI price estimation**, **interactive maps**, **automated legal documents** and a
**multi-channel notification engine** (In-App, Push/FCM, SMS, Email) — in one system.

## 1. Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19 + Vite 6 + Tailwind CSS 4 |
| Backend | Node.js + Express (REST API) |
| Database | Supabase PostgreSQL (migrations in `supabase/`) |
| Auth | Demo role-based auth (Supabase Auth wiring in `src/lib/supabase.ts`) |
| AI Price Model | Node ensemble (`server/priceEstimator.ts`) + Python Flask microservice (`ml_service/`) with Linear Regression / Random Forest / XGBoost |
| Maps | Google Maps JS API + SVG plot maps (D3) |
| Notifications | In-house orchestrator: SMS gateway, SMTP email, Firebase FCM, in-app |
| Payments | JazzCash / EasyPaisa / Bank-transfer / Pay-order flows (simulated gateway) |
| Voice listings | Browser speech capture + optional Flask transcription (`voice_transcribe.py`) |

## 2. The 12 Proposal Modules → Where They Live

| # | Proposal Module | Implementation |
|---|---|---|
| 1 | Authentication System | `src/views/auth/` (Login, Signup, OTP-style verify, Reset) + 1-click demo accounts; RBAC route guards in `src/App.tsx` |
| 2 | Property Marketplace | `src/views/public/PropertyMarketplaceView.tsx`, `PropertyDetailView.tsx`, `ComparisonView.tsx`, wishlist & compare |
| 3 | Society Management | `src/views/society/` — overview, plot inventory (`SocietyPlotInventoryView`), dealer assignment, booking approvals, financials |
| 4 | Dealer Management | `src/views/dealer/` — overview, assigned lots, Leads CRM (`DealerLeadsCrmView`), commission, listings manager, task automation |
| 5 | Interactive Maps | `src/components/maps/` (GoogleMapView, PropertyLocationPinMap, NearbyFacilitiesPanel) + SVG plot maps (`src/components/society/SocietyPlotsD3Map.tsx`, `InteractiveMasterplanMap`) |
| 6 | Booking System | `src/components/common/BookingModal.tsx` (3-step: select → dealer → payment) + `SocietyBookingApprovalsView` |
| 7 | Payment System | `src/components/common/PaymentSimModal.tsx`, `PaymentModal.tsx`, installment plans (`src/utils/paymentCalculators.ts`), late-fee calc, PDF receipts (`src/utils/pdfGenerator.ts`) |
| 8 | AI Price Prediction | `src/views/public/StandalonePriceEstimatorView.tsx` + `src/components/estimator/`, API `POST /api/price-estimate/` (`server/priceEstimator.ts`), Python ML service (`ml_service/app.py`) |
| 9 | Notification System | `server/notificationOrchestrator.ts` (+ `emailService.ts`, `smsService.ts`, `fcmService.ts`), API `/api/notifications/*`, `NotificationCenterView`, push toasts, 3-tier modal |
| 10 | Admin Panel | `src/views/admin/` — overview, analytics, users, verification queue, content moderation, disputes, audit logs |
| 11 | Legal Documentation | `server/legalDocuments.ts` (`POST /api/documents/generate-transfer-deed`, `/generate-agreement`, `/verify/:code`), `src/services/legalDocumentService.ts`, `DocumentLockerGrid`, PDF export via jsPDF |
| 12 | Analytics & Reporting | `src/views/admin/SuperAdminAnalyticsView.tsx`, `src/components/analytics/*` (7 chart components), `server/analyticsRouter.ts`, `src/services/analyticsService.ts` |

## 3. Run Locally

### Prerequisites
- Node.js 20+ and npm
- (Optional) Python 3.10+ for the ML microservice
- (Optional) A Supabase project if you want cloud persistence; the app runs fully offline on bundled demo data + `localStorage` without it

### 3.1 Install & start the main app
```bash
npm install
cp .env.example .env        # then fill in the keys you have (see §5)
npm run dev                 # Express + Vite on http://localhost:3000
```
- `npm run lint` → `tsc --noEmit` (must be clean)
- `npm run build` → Vite frontend build + esbuild server bundle into `dist/`
- `npm start` → run the production bundle: `node dist/server.cjs`

### 3.2 Start the Python ML microservice (optional)
The Node estimator works standalone; the Flask service adds the retrainable sklearn/XGBoost path.
```bash
cd ml_service
pip install -r requirements.txt
python train_models.py        # trains Linear / RF / XGBoost on ml_service/data, writes model_metrics.json
python app.py                 # serves POST /predict-price on :5000
```
Set the estimator service URL in `.env` if you host it elsewhere.

### 3.3 Demo accounts (1-click login on the Login page)
| Role | Email | Lands on |
|---|---|---|
| Public guest | `guest.explorer@manziliq.pk` | Marketplace (read-only) |
| Buyer | `farooq.buyer@gmail.com` | `/buyer/overview` |
| Dealer | `tariq.realtor@manziliq.pk` | `/dealer/overview` |
| Society Admin | `admin@alrehmangarden.pk` | `/society/overview` |
| Super Admin | `ayeshashoukat2023cs512@gmail.com` | `/admin/overview` |

Passwords: use the pre-filled 1-click demo buttons on `/login` (any of the standard demo passwords is accepted in demo mode).

## 4. Key API Endpoints (Express)

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Health check |
| POST | `/api/price-estimate/` | AI price estimate `{property_type, size_marla, society_name, ...}` → price + confidence + range |
| GET | `/api/price-estimate/models-performance` | R² / MAE / RMSE per model |
| POST | `/api/documents/generate-transfer-deed` | Generate transfer deed PDF record |
| POST | `/api/documents/generate-agreement` | Generate sale agreement |
| GET | `/api/documents/verify/:code` | Verify a document |
| GET/POST/PATCH | `/api/notifications/*` | List, dispatch, mark-read, preferences, retries, scheduled jobs |
| POST | `/api/ai/chat` | AI assistant (Gemini) |
| GET/POST | `/api/analytics/*` | Dashboard analytics |
| POST | `/api/seed` | Reset demo dataset |

## 5. Environment Variables (`.env.example` is the template)

| Variable | Required? | Purpose |
|---|---|---|
| `GEMINI_API_KEY` | For AI chat/assistant | Google AI Studio key |
| `VITE_SUPABASE_URL` / `VITE_SUPABASE_ANON_KEY` | For cloud DB | Supabase project URL + anon key |
| `VITE_GOOGLE_MAPS_API_KEY` | For live maps | Google Maps JS + Places API key |
| `SMTP_HOST/PORT/USER/PASS`, `EMAIL_FROM` | For real emails | Gmail App-Password or any SMTP |
| `SMS_GATEWAY_API_KEY/URL`, `SMS_GATEWAY_SENDER_ID` | For real SMS | Pakistan SMS gateway (or Twilio) |
| `FCM_SERVER_KEY`, `FIREBASE_PROJECT_ID` | For push | Firebase Cloud Messaging |
| `OPENAI_API_KEY` | Optional | Whisper voice-listing transcription fallback |
| `FLASK_VOICE_SERVICE_URL` | Optional | External voice microservice |

Without these keys the app still runs: maps fall back to SVG plot maps, payments run in
simulation mode, notifications stay in-app, and the estimator uses the built-in ensemble.

## 6. Database Setup (Supabase)

1. Create a project at supabase.com → copy Project URL + anon key into `.env`.
2. Run the migrations in `supabase/migrations/` (in order) via the Supabase SQL editor.
3. Load demo data: `supabase/seed.sql` (also mirrored at `scripts/seed.sql`).
4. Schema reference: `src/db/schema.ts` (+ `legal_documents_schema.sql`, `analytics_schema.sql`, `google_maps_schema.sql`).

## 7. Deploy to Production (per proposal: Vercel + Railway)

**Frontend (Vercel)**
1. Push this folder to GitHub.
2. Vercel → New Project → import repo. Framework preset: **Vite**.
3. Build command: `npm run build` · Output dir: `dist` · then serve `dist/` statically
   (or deploy the Express server below and point Vercel rewrites at it).
4. Add the `VITE_*` env vars from §5 in Vercel → Settings → Environment Variables.

**Backend API (Railway / any Node host)**
1. Same repo, start command: `npm run build && npm start` (serves API + static frontend on `$PORT`).
2. Add server-side env vars from §5. Railway injects `PORT` automatically — `server.ts` listens on `process.env.PORT || 3000`.
3. Point your domain / Vercel rewrites `/api/*` to this service.

**ML microservice (Railway)**
1. New Railway service rooted at `ml_service/`: `pip install -r requirements.txt && python app.py`
   (add a `Procfile`: `web: python app.py`, or set the start command in Railway).
2. Copy its public URL into the main app's env for the estimator/voice endpoints.

**Google Maps**: enable *Maps JavaScript API* + *Places API*, restrict the key to your domain.

## 8. Launch Checklist

- [ ] `npm run lint` clean, `npm run build` succeeds
- [ ] `.env` filled (at minimum `VITE_GOOGLE_MAPS_API_KEY` for live maps)
- [ ] Supabase migrations + seed applied (or demo localStorage mode accepted for the FYP demo)
- [ ] Demo accounts tested on each role dashboard
- [ ] Booking → approval → installment → receipt flow tested end-to-end
- [ ] AI estimator returns a price + confidence tier
- [ ] A transfer deed generates and verifies via `/verify/:code`
- [ ] Notifications dispatch in-app (SMTP/SMS/FCM keys added for real channels)
- [ ] Domain + HTTPS enabled on the host

## 9. Known Limitations (demo → production hardening)

- Payments are **simulated** (JazzCash/EasyPaisa sandbox-style flow); wire real gateway APIs with webhooks before handling real money.
- Auth is demo role-based; production should enforce Supabase Auth + RLS policies on every table.
- File uploads use local `/uploads`; production should use Supabase Storage or Cloudinary.
- The bundled dataset covers Narowal societies; retrain the ML model (`ml_service/retrain_model.py`) as real transaction data grows.
#   m a n z i l i q - 0 1  
 