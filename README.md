# Phantasmagoria — Alumni Analytics Dashboard and API

A university project for managing alumni profiles and displaying graduate data through an interactive dashboard.

The project builds on my earlier Alumni Influencers API, adding charts, filters and profile CSV export. The original bidding and featured-alumnus features remain available through the backend API.

## Features

- Registration, account verification and login.
- Create and update an alumni profile.
- View charts showing industries, programmes, graduation years and common skills.
- Filter dashboard results.
- Export your own profile as a CSV file.
- Generate and revoke API keys through backend endpoints.
- View API usage logs.
- Explore API documentation using Swagger.

## Technologies

- **Frontend:** HTML, CSS, JavaScript and Chart.js.
- **Backend:** Node.js and Express.js.
- **Database:** MongoDB Atlas and Mongoose.
- **Other tools:** bcrypt, express-session, Swagger and GitHub.

## Application Links

- [Frontend](https://alexandru-rx.github.io/w1887688-Alumni-Analytics-Dashboard/)
- [Backend Health Check](https://w1887688-advanced-server-side.onrender.com/health)
- [API Documentation](https://w1887688-advanced-server-side.onrender.com/api-docs/)

Registration requires a Westminster email address by default. Verification links currently appear in the backend console instead of being sent by email, so hosted registration requires developer access to the server logs.

## Running Locally

### 1. Install the dependencies

Install Node.js and npm. The current Mongoose dependency requires Node.js 20.19 or later.

Clone the repository and install its dependencies:

```bash
git clone https://github.com/alexandru-rx/w1887688-Alumni-Analytics-Dashboard.git
cd w1887688-Alumni-Analytics-Dashboard
npm ci
```

### 2. Configure the backend

Create a `.env` file in the project root:

```dotenv
PORT=3000
MONGO_URI=your_mongodb_connection_string
UNIVERSITY_DOMAIN=westminster.ac.uk
BASE_URL=http://localhost:3000
VERIFY_TOKEN_SECRET=your_long_random_verification_secret
SESSION_SECRET=your_different_long_random_session_secret
NODE_ENV=development
```

Replace the placeholders with your own database connection string and two different random secrets. Do not upload `.env` to GitHub.

### 3. Configure the frontend

In `docs/app.js`, change the API address for local development:

```js
const API = "http://localhost:3000";
```

### 4. Start the application

Start the backend:

```bash
npm start
```

Check that this address returns an API-running message:

```text
http://localhost:3000/health
```

Use VS Code Live Server to serve the frontend on port `5500`. If serving the repository root, open:

```text
http://localhost:5500/docs/index.html
```

Use `localhost` for both the frontend and backend to avoid cookie issues.

### 5. Try the features

1. Register with an email matching the configured university domain.
2. Open the verification link printed in the backend terminal.
3. Return to the frontend and log in.
4. Complete your profile.
5. Open the dashboard and try the filters and CSV export.

Passwords require at least eight characters, an uppercase letter and a number. Programme and industry filters match the stored values exactly.

**Optional sample data:** `npm run seed` deletes all existing profiles in the configured database and inserts sample profiles. Only use it with a disposable development database. It does not create login accounts.

## API Documentation and Usage Logs

Local Swagger documentation is available at:

```text
http://localhost:3000/api-docs/
```

Swagger currently points requests at the local backend, even when viewed through the hosted documentation link.

The client endpoint uses an API key with the `read:alumni_of_day` permission:

```http
GET /client/participatingAlumnus/tomorrow
Authorization: Bearer YOUR_API_TOKEN
```

Requests that pass the API-key checks create a usage record containing the key, endpoint, method and time. The dashboard reloads these logs when refreshed or when **Apply Filters** is clicked.

Login and profile updates are not included in this usage table.

## Project Structure

```text
docs/          Frontend pages, JavaScript and styles
models/        Database models
routes/        API endpoints
middleware/    Authentication and API-key checks
server.js      Backend entry point
db.js          Database connection
seed.js        Sample profile data
swagger.js     API documentation configuration
```

## Known Limitations

- Account verification currently uses links printed in server logs rather than email delivery.
- The dashboard Logout link opens the login page without ending the server session.
- CSV values containing commas, quotes or line breaks need proper escaping.
- API keys and usage logs are not separated by account, and keys are stored as plaintext.
- The retained winner-selection endpoint needs an access restriction.
- The frontend API address is configured manually.
- Automated tests are not yet configured.

This is a university coursework project that I am continuing to review and improve.
