# Career-ai-app
Career app with ai support

An AI-powered career assistant platform. It helps users analyze and build resumes,
match jobs, and prepare for interviews. The backend uses Google Gemini for AI
features, Google OAuth for authentication, and Razorpay for subscription payments.

## Tech Stack

| Layer    | Technology |
| -------- | ---------- |
| Backend  | Node.js, Express 5, TypeScript, Mongoose (MongoDB), JWT |
| AI       | Google Gemini (`@google/genai`) |
| Auth     | Google OAuth 2.0 (`googleapis`) |
| Payments | Razorpay |
| Frontend | React 19, Vite, TypeScript, Tailwind CSS 4, React Router 7 |
| Tooling  | Docker, Docker Compose, concurrently, ESLint |

## Repository Structure

```
ai-career/
├── docker-compose.yml
├── .env                      # Docker Compose secrets (not committed)
├── .env.example              # Template for the above
├── README.md
├── ai-career-backend-master/ # Express + TypeScript API
│   ├── src/
│   │   ├── index.ts          # App entrypoint
│   │   ├── config/           # db, google oauth, gemini prompts
│   │   ├── controllers/      # user, ai, payment logic
│   │   ├── middlewares/      # isAuth, trycatch
│   │   ├── models/           # Mongoose models
│   │   └── routes/           # /api/user, /api/ai, /api/payment
│   ├── Dockerfile
│   └── package.json
└── ai-career-frontend-master/ # Vite + React SPA
    ├── src/
    │   ├── pages/            # Home, Login, Account, Analyse, ...
    │   ├── components/       # navbar, footer, pricing, ...
    │   ├── context/          # AppContext (auth state)
    │   └── main.tsx          # Entry + API base URL
    ├── Dockerfile
    ├── nginx.conf
    └── package.json
```

## Features

- Google OAuth sign-in with JWT sessions (15-day expiry).
- Resume analysis and scoring.
- AI resume builder.
- Job matcher.
- Interview preparation.
- Freemium model: each user gets 3 free AI requests, then a Razorpay-backed
  subscription is required.

---

## Prerequisites

Choose one of the two approaches below.

### Option A: Docker (recommended)

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Compose).

No Node.js, MongoDB, or other local tools required. Everything runs in containers.

### Option B: Manual / local development

- [Node.js](https://nodejs.org/) v20 or later (v22 recommended).
- [MongoDB](https://www.mongodb.com/try/download/community) running locally, or a
  free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster.

---

## Required API Keys

The app integrates with several third-party services. Obtain the following:

| Variable | Where to get it |
| -------- | --------------- |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | [Google Cloud Console > Credentials](https://console.cloud.google.com/apis/credentials) (OAuth 2.0 Client ID) |
| `API_KEY_GEMINI` | [Google AI Studio](https://aistudio.google.com/app/apikey) |
| `Razorpay_Key` / `Razorpay_Secret` | [Razorpay Dashboard > API Keys](https://dashboard.razorpay.com/app/keys) |
| `JWT_SEC` | Any long random string you generate yourself |

> Google OAuth is configured with the `postmessage` redirect flow and the frontend
> uses `@react-oauth/google`. Make sure your OAuth client allows the origin where
> the frontend runs (e.g. `http://localhost` and `http://localhost:5173`).

---

## Setup: Docker (Recommended)

All commands are run from the repository root (`ai-career/`).

### 1. Create the environment file

```bash
cp .env.example .env
```

Edit `.env` and fill in real values:

```env
JWT_SEC=change_me_to_a_long_random_string
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
API_KEY_GEMINI=your_gemini_api_key
Razorpay_Key=your_razorpay_key_id
Razorpay_Secret=your_razorpay_key_secret
VITE_SERVER_URL=http://localhost:5000
```

`VITE_SERVER_URL` is baked into the frontend build and must be the URL the
**browser** uses to reach the backend.

### 2. Build and start everything

```bash
docker compose up -d --build
```

This starts three containers:

| Service  | Container            | URL / Port            |
| -------- | -------------------- | --------------------- |
| Mongo    | `ai-career-mongo`    | `localhost:27017`     |
| Backend  | `ai-career-backend`  | `http://localhost:5000` |
| Frontend | `ai-career-frontend` | `http://localhost`    |

MongoDB data is persisted in the `mongo_data` Docker volume.

### 3. Open the app

Visit **http://localhost** in your browser.

### Common Docker commands

```bash
# View logs
docker compose logs -f backend
docker compose logs -f frontend

# Rebuild after changing code or .env
docker compose up -d --build

# Stop containers (keeps data)
docker compose down

# Stop and delete the database volume
docker compose down -v
```

---

## Setup: Manual / Local Development

### 1. Backend

```bash
cd ai-career-backend-master
npm install
```

Create `ai-career-backend-master/.env`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ai-career
JWT_SEC=your_jwt_secret
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
API_KEY_GEMINI=your_gemini_api_key
Razorpay_Key=your_razorpay_key_id
Razorpay_Secret=your_razorpay_key_secret
```

> For MongoDB Atlas, set `MONGO_URI` to your connection string, e.g.
> `mongodb+srv://<user>:<password>@<cluster>/?retryWrites=true&w=majority`.

Run in development (TypeScript watch + server auto-restart):

```bash
npm run dev
```

Or build and run the compiled output:

```bash
npm run build
npm start
```

### 2. Frontend

```bash
cd ai-career-frontend-master
npm install
```

Point the frontend at your local backend. Set `VITE_SERVER_URL` in a
`ai-career-frontend-master/.env` file:

```env
VITE_SERVER_URL=http://localhost:5000
```

Or edit the fallback directly in `src/main.tsx`. Start the dev server:

```bash
npm run dev
```

Open the printed URL (default **http://localhost:5173**).

---

## Environment Variables Reference

### Backend (`ai-career-backend-master/.env`)

| Variable | Required | Description |
| -------- | -------- | ----------- |
| `PORT` | Yes | Port the API listens on (e.g. `5000`) |
| `MONGO_URI` | Yes | MongoDB connection string |
| `JWT_SEC` | Yes | Secret used to sign JWT tokens |
| `GOOGLE_CLIENT_ID` | Yes | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | Yes | Google OAuth client secret |
| `API_KEY_GEMINI` | Yes | Google Gemini API key |
| `Razorpay_Key` | Yes | Razorpay key ID |
| `Razorpay_Secret` | Yes | Razorpay key secret |

### Frontend (`ai-career-frontend-master/.env`)

| Variable | Required | Description |
| -------- | -------- | ----------- |
| `VITE_SERVER_URL` | No | Backend base URL. Falls back to the deployed Render URL if unset. |

---

## API Reference

Base URL: `http://localhost:5000`

### User

| Method | Endpoint | Auth | Body | Description |
| ------ | -------- | ---- | ---- | ----------- |
| `POST` | `/api/user/login` | No | `{ "code": "<google auth code>" }` | Exchange Google code for a JWT |
| `GET`  | `/api/user/me` | Yes | — | Get the current user profile |

Authenticated requests must include the header `Authorization: Bearer <token>`.

### AI

| Method | Endpoint | Auth | Description |
| ------ | -------- | ---- | ----------- |
| `POST` | `/api/ai/analyse` | Yes | Analyze and score a resume |
| `POST` | `/api/ai/resume-build` | Yes | Generate/build a resume |
| `POST` | `/api/ai/job-matcher` | Yes | Match a profile against jobs |
| `POST` | `/api/ai/interview` | Yes | Generate interview questions |

### Payment

| Method | Endpoint | Auth | Description |
| ------ | -------- | ---- | ----------- |
| `POST` | `/api/payment/checkout` | Yes | Create a Razorpay order |
| `POST` | `/api/payment/verify` | Yes | Verify a Razorpay payment signature |

---

## Frontend Routes

| Route | Access | Page |
| ----- | ------ | ---- |
| `/` | Public | Home |
| `/login` | Public (redirects if logged in) | Google login |
| `/account` | Protected | Account |
| `/analyse` | Protected | Resume analysis |
| `/jobmatcher` | Protected | Job matcher |
| `/interviewprep` | Protected | Interview prep |
| `/resumebuilder` | Protected | Resume builder |

---

## Scripts

### Backend (`ai-career-backend-master`)

| Script | Command | Description |
| ------ | ------- | ----------- |
| `npm run dev` | `concurrently "tsc --watch" "node --watch dist/index.js"` | Dev mode with live reload |
| `npm run build` | `npm install && tsc` | Install deps and compile TypeScript |
| `npm start` | `node dist/index.js` | Run the compiled server |

### Frontend (`ai-career-frontend-master`)

| Script | Command | Description |
| ------ | ------- | ----------- |
| `npm run dev` | `vite` | Dev server with HMR |
| `npm run build` | `tsc -b && vite build` | Type-check and build for production |
| `npm run preview` | `vite preview` | Preview the production build |
| `npm run lint` | `eslint .` | Lint the codebase |

---

## Troubleshooting

- **Port already in use**: change the published port in `docker-compose.yml`
  (e.g. `"8080:80"`) or stop the conflicting process.
- **Backend crashes on start**: ensure all required backend env vars are set.
  Razorpay and MongoDB will fail fast if their values are missing or invalid.
- **Frontend can't reach the API**: verify `VITE_SERVER_URL` matches the backend
  URL reachable from the browser, then rebuild the frontend
  (`docker compose up -d --build` or restart `npm run dev`).
- **Google login fails**: confirm the OAuth client in Google Cloud allows your
  frontend origin as an authorized JavaScript origin, and that
  `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` match.
- **No AI response / quota errors**: check `API_KEY_GEMINI` is valid and has
  remaining quota. Non-subscribed users are limited to 3 free requests.

---

## Notes

- The backend has a keep-alive routine that pings the deployed Render URL every
  30 seconds. It is harmless locally but you can remove it from
  `ai-career-backend-master/src/index.ts` if you don't need it.
- `.env` files are git-ignored. Never commit real secrets.
