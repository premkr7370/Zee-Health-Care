# Doctor Appointment Web

A doctor appointment management application with a patient-facing website, an administrator dashboard, and a REST API. Patients can register, browse doctors, request appointments, and send messages. Administrators can manage appointments and doctors, add administrators, and review submitted messages.

## Features

- Patient registration and login with cookie-based authentication.
- Doctor directory and appointment request form.
- Contact message submission.
- Administrator dashboard for appointment status updates, doctor management, adding administrators, and viewing messages.
- MongoDB persistence through Mongoose.
- Doctor avatar uploads through Cloudinary.
- Separate frontend, dashboard, and backend applications.

## Technology stack

| Area | Technologies |
| --- | --- |
| Patient website | React 19, React Router, Vite, Axios |
| Admin dashboard | React 19, React Router, Vite, Axios |
| API | Node.js, Express 4 |
| Data and authentication | MongoDB, Mongoose, JWT, bcrypt, HTTP cookies |
| Image uploads | Cloudinary, express-fileupload |
| Code quality | ESLint in both Vite applications |

## Architecture

The project is split into three independently installed applications:

1. `frontend/` contains the public patient website and submits registration, appointment, and contact requests.
2. `dashboard/` contains the administrator interface for doctor and appointment administration and message review.
3. `backend/` exposes the Express API, validates and processes requests, connects to MongoDB, and uploads doctor avatars to Cloudinary.

The API is mounted at `/api/v1` and separates user, appointment, and message routes. Patient and administrator authentication use separate HTTP-only cookies.

> **Local development note:** The frontend and dashboard currently contain hard-coded requests to the deployed Render API. Starting the backend locally does not automatically make either Vite app use it. Connecting the web apps to a local backend requires changing those API URLs in the application source; this README does not modify application code.

## Project structure

```text
.
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── Pages/             # Home, appointment, about, login, registration
│   │   ├── components/        # Navigation, forms, hero, departments, footer
│   │   ├── App.jsx
│   │   ├── App.css
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
├── dashboard/
│   ├── public/
│   ├── src/
│   │   ├── components/        # Dashboard, login, doctors, messages, admin/doctor forms
│   │   ├── App.jsx
│   │   ├── App.css
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
├── backend/
│   ├── config/                # Environment template and private runtime config
│   ├── controller/            # User, appointment, and message handlers
│   ├── database/              # MongoDB connection
│   ├── middlewares/           # Authentication and error handling
│   ├── models/                # Mongoose schemas
│   ├── router/                # Express API routes
│   ├── utils/                 # JWT cookie helper
│   ├── app.js
│   ├── server.js
│   └── package.json
├── .gitignore
└── README.md
```

Each application has its own `package.json` and lockfile. Install dependencies separately in all three directories; there is no root npm workspace or root-level npm script.

## Prerequisites

- Node.js and npm (a current Node.js LTS release is recommended).
- A MongoDB connection string, either from a local MongoDB instance or MongoDB Atlas.
- Cloudinary credentials for adding doctors with avatar images.
- Git to clone and contribute to the repository.

## Installation and setup

1. Clone the repository and open a terminal in the project directory.
2. Install each application's dependencies:

   ```sh
   cd backend
   npm ci
   cd ../frontend
   npm ci
   cd ../dashboard
   npm ci
   ```

   `npm ci` installs the exact dependency versions recorded in each application's `package-lock.json`.

3. Configure the backend environment. From the project directory, copy the example file:

   ```sh
   # macOS/Linux
   cp backend/config/.env.example backend/config/config.env
   ```

   On Windows PowerShell:

   ```powershell
   Copy-Item backend\config\.env.example backend\config\config.env
   ```

4. Edit `backend/config/config.env` and set your own MongoDB, JWT, and Cloudinary values. Do not commit this file.
5. Start the backend from its directory, then start each Vite application in a separate terminal as described below.

The backend loads `./config/config.env` relative to its working directory, so run its npm commands from `backend/`.

## Environment variables

The backend template is [`backend/config/.env.example`](backend/config/.env.example). It documents all variables read by the backend:

| Variable | Purpose |
| --- | --- |
| `PORT` | Port on which the Express server listens. |
| `MONGO_URI` | MongoDB connection URI. |
| `FRONTEND_URI` | Allowed patient-site origin for CORS. |
| `DASHBOARD_URI` | Allowed dashboard origin for CORS. |
| `JWT_SECRET_KEY` | Secret used to sign and verify authentication tokens. Use a unique, strong value. |
| `JWT_EXPIRES` | JWT token lifetime (for example, `7d`). |
| `COOKIE_EXPIRE` | Cookie lifetime in days. |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name used for doctor image uploads. |
| `CLOUDINARY_API_KEY` | Cloudinary API key. |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret. |

The example uses local origins `http://localhost:5173` and `http://localhost:5174`. Update the allowed origins if the apps run on different ports or domains. The database connection selects the `Doctor_Appointment_web` database.

## Run locally

Run each command in its own terminal from the named directory:

| Application | Directory | Command | Local address |
| --- | --- | --- | --- |
| API | `backend/` | `npm start` | `http://localhost:4000` (when `PORT=4000`) |
| Patient website | `frontend/` | `npm run dev -- --port 5173` | `http://localhost:5173` |
| Admin dashboard | `dashboard/` | `npm run dev -- --port 5174` | `http://localhost:5174` |

The Vite commands start the development servers with hot reload. The frontend and dashboard's API requests still target the hosted Render service as noted above; they will not use `http://localhost:4000` without an application-code change. An administrator account must already exist to sign in to the dashboard; the repository does not include an admin seed script.

The backend's `dev` script calls `nodemon`, but `nodemon` is not listed in `backend/package.json`; use `npm start` for a clean install, or provide Nodemon separately if you want automatic restarts.

## Available commands

Run these commands from the corresponding application directory.

| Directory | Command | Description |
| --- | --- | --- |
| `backend/` | `npm run dev` | Start the API with Nodemon for development (Nodemon is not declared in the backend dependencies). |
| `backend/` | `npm start` | Start the API with Node.js. |
| `frontend/` | `npm run dev` | Start the patient website's Vite development server. |
| `frontend/` | `npm run build` | Create the production website build in `dist/`. |
| `frontend/` | `npm run preview` | Preview the production build locally. |
| `frontend/` | `npm run lint` | Run ESLint on the frontend project. |
| `dashboard/` | `npm run dev` | Start the dashboard's Vite development server. |
| `dashboard/` | `npm run build` | Create the production dashboard build in `dist/`. |
| `dashboard/` | `npm run preview` | Preview the production build locally. |
| `dashboard/` | `npm run lint` | Run ESLint on the dashboard project. |

## API

The API base path is `/api/v1`. Authenticated administrator endpoints require the `adminToken` cookie; patient endpoints require `patientToken`.

| Method | Path | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/user/patient/register` | Public | Register a patient. |
| `POST` | `/user/login` | Public | Log in with email, password, confirmation, and role. |
| `GET` | `/user/doctors` | Public | List doctors. |
| `GET` | `/user/patient/me` | Patient | Get the authenticated patient. |
| `GET` | `/user/admin/me` | Administrator | Get the authenticated administrator. |
| `GET` | `/user/patient/logout` | Patient | Clear the patient login cookie. |
| `GET` | `/user/admin/logout` | Administrator | Clear the administrator login cookie. |
| `POST` | `/user/admin/addnew` | Administrator | Add an administrator. |
| `POST` | `/user/doctor/addnew` | Administrator | Add a doctor and upload an avatar. |
| `POST` | `/appointment/post` | Patient | Request an appointment. |
| `GET` | `/appointment/getall` | Administrator | List appointments. |
| `PUT` | `/appointment/update/:id` | Administrator | Update an appointment. |
| `DELETE` | `/appointment/delete/:id` | Administrator | Delete an appointment. |
| `POST` | `/message/send` | Public | Submit a contact message. |
| `GET` | `/message/getall` | Administrator | List submitted messages. |

For example, to submit a contact message to a local API:

```sh
curl -X POST http://localhost:4000/api/v1/message/send \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alex","lastName":"Morgan","email":"alex@example.com","phone":"1234567890","message":"Please contact me about an appointment."}'
```

## Database

The backend connects to MongoDB using `MONGO_URI` and stores application data in the `Doctor_Appointment_web` database. Configure a local MongoDB URI or an Atlas URI in `backend/config/config.env`. No migration or seed command is defined in the backend package scripts.

## Screenshots

No screenshots are currently included. Add screenshots here when they are available.

## Deployment

No deployment configuration or deployment scripts are included in the repository. For a deployment, host the backend and both Vite applications separately, provide the backend environment variables through the hosting provider's secret configuration, allow the deployed frontend origins in `FRONTEND_URI` and `DASHBOARD_URI`, and configure a production MongoDB URI and Cloudinary credentials. The frontend and dashboard currently use a hard-coded hosted API URL, which should be made environment-configurable before deploying against another backend.

## Troubleshooting

- **MongoDB connection fails:** Check that `MONGO_URI` is set, correctly encoded, and allowed by your MongoDB network access rules.
- **Requests fail with a CORS error:** Make sure `FRONTEND_URI` and `DASHBOARD_URI` exactly match the browser origins, including scheme and port.
- **The backend cannot find its configuration:** Run the backend command from `backend/` and confirm `backend/config/config.env` exists.
- **Doctor image uploads fail:** Verify all Cloudinary values and use a supported image format (PNG, JPEG, or WebP).
- **The browser contacts the deployed API instead of localhost:** This is the current frontend/dashboard behavior; their API URLs are hard-coded in application source.
- **Dashboard login is unavailable for a new installation:** An administrator account must be provisioned first; no initial-admin seed command is provided.

## Potential improvements

- Move the frontend and dashboard API base URL into Vite environment configuration.
- Add automated backend tests and a repeatable initial-administrator setup process.
- Add screenshots and deployment configuration once the target hosting setup is selected.

## Contributing

1. Create a branch for your change.
2. Keep changes focused and do not commit environment files, credentials, or build output.
3. Run the relevant frontend or dashboard lint/build commands before opening a pull request.
4. Describe the change and include screenshots for user-interface updates when practical.
#   Z e e - H e a l t h - C a r e  
 