# Phantasmagoria — Alumni Analytics Dashboard and API

A university coursework project for collecting alumni profile data and exploring it through an interactive dashboard. Users can register, create a profile, filter charts by programme, industry and graduation year, and export their own profile as a CSV file.

This version builds on the earlier Alumni Influencers API. It retains the original bidding and featured-alumnus backend routes and adds the analytics dashboard. Bidding features are available through the API rather than the current dashboard interface.

## Main Features

- Account registration, verification and session-based login.
- Alumni profiles containing programme, graduation year, industry, current role, skills and certifications.
- Dashboard summaries and charts showing industry distribution, programmes, graduation years and common profile skills.
- Filters for programme, industry sector and graduation year.
- CSV export of the signed-in user's profile.
- API key generation, listing and revocation through backend endpoints.
- API usage logs displayed in the dashboard.
- A rate-limited client endpoint for retrieving tomorrow's featured alumnus.
- Swagger API documentation.

## Technologies

| Area | Technologies |
| --- | --- |
| Frontend | HTML, CSS, JavaScript, Chart.js, Chart.js Data Labels plugin |
| Backend | Node.js, Express.js |
| Database | MongoDB Atlas, Mongoose |
| Authentication | bcrypt, express-session, connect-mongo |
| API tooling | swagger-jsdoc, swagger-ui-express, express-rate-limit |
| Hosting | GitHub Pages and Render |

## Hosted Application

- [Frontend](https://alexandru-rx.github.io/w1887688-Alumni-Analytics-Dashboard/)
- [Backend Health Check](https://w1887688-advanced-server-side.onrender.com/health)
- [Swagger Documentation](https://w1887688-advanced-server-side.onrender.com/api-docs/)

Registration defaults to `@westminster.ac.uk` addresses. Verification links are printed in the backend console rather than sent by email. Completing registration on the hosted version therefore requires developer access to the server logs.

The hosted application is not currently a self-service public demo. To explore it independently, run a local copy using the instructions below.

The Swagger configuration currently points requests at `http://localhost:3000`, including when its documentation is viewed on the hosted backend.

## Running Locally

### Requirements

- Node.js compatible with the dependencies in `package-lock.json`; Mongoose requires Node.js 20.19 or later.
- npm.
- Your own MongoDB development database, either locally or through Atlas.
- A static web server for the frontend, such as the VS Code Live Server extension.

### 1. Clone the Repository

```bash
git clone https://github.com/alexandru-rx/w1887688-Alumni-Analytics-Dashboard.git
cd w1887688-Alumni-Analytics-Dashboard
npm ci
```

### 2. Configure Environment Variables

Create a `.env` file in the project root:

```dotenv
PORT=3000
MONGO_URI=mongodb://127.0.0.1:27017/phantasmagoria_dev
UNIVERSITY_DOMAIN=westminster.ac.uk
BASE_URL=http://localhost:3000
VERIFY_TOKEN_SECRET=replace-with-a-long-random-secret
SESSION_SECRET=replace-with-a-different-long-random-secret
NODE_ENV=development
```

The example connection string requires a local MongoDB server. Replace it with your own connection string if using MongoDB Atlas.

Replace both secret placeholders with different, long random values. Do not commit `.env` to GitHub.

Ensure the root `.gitignore` includes:

```gitignore
.env
node_modules/
```

| Variable | Purpose |
| --- | --- |
| `PORT` | Backend port; defaults to `3000` |
| `MONGO_URI` | Database connection string for application data and sessions |
| `UNIVERSITY_DOMAIN` | Allowed registration domain |
| `BASE_URL` | Backend address used to construct verification links |
| `VERIFY_TOKEN_SECRET` | Secret used when hashing verification tokens |
| `SESSION_SECRET` | Secret used to sign session cookies |
| `NODE_ENV` | Controls production cookie settings |

The current `.env.example` contains a duplicated `BASE_URL=` prefix. Use the corrected example above.

`INDEX_URL` is listed in `.env.example` but is not currently referenced by the application code.

### 3. Configure the Frontend

In `docs/app.js`, change the API address for your local copy:

```js
const API = "http://localhost:3000";
```

The committed value points at the hosted Render backend. Changing `.env` alone does not update this frontend address.

### 4. Start the Backend

From the project root, run:

```bash
npm start
```

Alternatively, start development mode with automatic backend restarts:

```bash
npm run dev
```

Check that the following URL returns an API-running message:

```text
http://localhost:3000/health
```

### 5. Start the Frontend

Serve the frontend on port `5500`.

With VS Code Live Server serving the repository root, open:

```text
http://localhost:5500/docs/index.html
```

If your static server serves `docs/` directly, open:

```text
http://localhost:5500/index.html
```

Use `localhost` for both the frontend and backend. Mixing `localhost` and `127.0.0.1`, or using another frontend port, can cause cookie or CORS problems.

### 6. Explore the Application

1. Register with an address matching `UNIVERSITY_DOMAIN`.
2. Use a password containing at least eight characters, an uppercase letter and a number.
3. Open the verification link printed in your local backend terminal.
4. Return to the frontend and log in.
5. Open **Profile**, enter alumni details and save.
6. Open **Dashboard** to explore the charts and filters.
7. Use the export button to download your profile as CSV.

Verification links expire after one hour and can only be used once. This development flow does not send an email or confirm mailbox ownership.

Programme and industry filters use exact matching, so enter values as stored in profiles.

### Optional Sample Data

The following command deletes **all existing Profile documents** in the configured database and inserts seven sample profiles:

```bash
npm run seed
```

Run it only against a disposable development database.

The seed script does not create matching user accounts. Sample profiles populate the analytics but cannot be used as login credentials.

## API Usage Logs

A usage record is created when a request passes the API-key and permission checks for the client endpoint.

Each record stores:

- API key reference.
- Endpoint.
- HTTP method.
- Access time.

The dashboard reloads this table when the page loads or **Apply Filters** is clicked. It does not update live.

Login, profile updates and analytics requests are not recorded in this API-key usage table.

A valid authorised client request is logged even if the endpoint subsequently returns `404` because no featured alumnus or linked profile is available.

## Selected API Endpoints

Session-protected routes require a successful login and the session cookie.

| Method | Endpoint | Purpose | Access |
| --- | --- | --- | --- |
| POST | `/auth/register` | Register an account | Public |
| GET | `/auth/verify?token=...` | Complete verification | Verification token |
| POST | `/auth/login` | Start a session | Public |
| POST | `/auth/logout` | End a session | Current session, if any |
| POST | `/profile` | Create or update own profile | Session |
| GET | `/profile/me` | Retrieve own profile | Session |
| GET | `/analytics/summary` | Dashboard summary | Session |
| GET | `/analytics/by-industry` | Group profiles by industry | Session |
| GET | `/analytics/by-programme` | Group profiles by programme | Session |
| GET | `/analytics/by-graduation-year` | Group profiles by graduation year | Session |
| GET | `/analytics/skills` | Most common profile skills | Session |
| POST | `/apiKeys/generate` | Generate an API key | Session |
| GET | `/apiKeys` | List API keys | Session |
| PUT | `/apiKeys/revoke/:id` | Revoke an API key | Session |
| GET | `/apiKeys/usage` | Retrieve API usage logs | Session |
| GET | `/client/participatingAlumnus/tomorrow` | Retrieve tomorrow's featured profile | Bearer token with required permission |

Analytics endpoints accept optional `programme`, `industrySector` and `graduationYear` query parameters.

### Client Authentication

For the client endpoint, send:

```http
Authorization: Bearer YOUR_API_TOKEN
```

The key requires the `read:alumni_of_day` permission.

The endpoint has an IP-based limit of 100 requests per minute. It requires a featured-alumnus record and linked profile for tomorrow, returning `404` if either is unavailable.

Original bidding and winner-selection routes remain under `/bids` and `/participatingAlumnus`.

For further details, open the local Swagger documentation:

```text
http://localhost:3000/api-docs/
```

The Swagger annotations provide an overview. Consult the route handlers for complete request bodies and authentication requirements.

## Architecture

The frontend sends HTTP requests to the Express API. The backend handles application logic and accesses MongoDB through Mongoose. Sessions are also stored in MongoDB.

### Project Structure

```text
docs/            Frontend pages, charts, CSV export and styles
models/          Mongoose models
routes/          Authentication, profiles, analytics and other API routes
middleware/      Session and API key checks
db.js            Database connection
server.js        Express application and session configuration
seed.js          Optional sample-profile seed script
swagger.js       OpenAPI documentation configuration
```

### Profiles

A registered user initially has no profile and can create one after registration. The profile schema declares a unique `userId`, and the save route updates or inserts the profile associated with the signed-in user.

### Analytics

MongoDB aggregation groups profile data for the charts.

For skills, the API separates skill arrays into individual entries, counts occurrences and returns the top eight. These counts describe skills recorded by alumni, rather than employer demand.

## Current Limitations and Next Improvements

- Verification links appear in the backend console; email delivery remains to be implemented.
- API key management and usage records are shared across authenticated users rather than scoped to individual owners.
- API keys are stored as plaintext. Owner-based access and hashed storage would improve the implementation.
- The retained winner-selection endpoint currently has no authentication middleware and needs an appropriate access restriction before broader deployment.
- The dashboard's **Logout** link opens the login page without calling `/auth/logout`, so it does not terminate the server session.
- CSV export does not currently escape commas, quotes or line breaks, which can affect spreadsheet column alignment.
- The frontend API address is hard-coded, and the Swagger server address is local-only.
- No automated test script is configured. Tests for authentication, profile updates, analytics filters and API permissions would be useful additions.

This repository is a university coursework application and a project for continued learning and improvement.
