# Eco Bloom

> A plantation and environmental-action platform for Dehradun: join plantation drives, donate, and report environmental issues, with an admin dashboard on a Node.js microservice backend.

![React](https://img.shields.io/badge/React_19-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Redux](https://img.shields.io/badge/Redux_Toolkit-764ABC?style=for-the-badge&logo=redux&logoColor=white)
![MUI](https://img.shields.io/badge/MUI-007FFF?style=for-the-badge&logo=mui&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-635BFF?style=for-the-badge&logo=stripe&logoColor=white)

## About

Eco Bloom helps an environmental organisation run plantation drives and track environmental issues. Citizens sign up as users to enrol in upcoming plantation drives, donate through Stripe, and file reports (pollution, deforestation, waste dumping and more) with photos and a Dehradun location. Admins create drives, review donations and reports, assign tasks to members, and see report analytics. The backend is split into small Express microservices behind an API gateway.

## Features

### Users
- Sign up and log in (JWT), with role-based routing to the user, member or admin area.
- Browse upcoming plantation drives and enrol in them.
- Donate through a Stripe payment flow (Payment Intents).
- Report an environmental issue: type (15 categories such as pollution, deforestation, waste dumping, forest fire), description, Dehradun area and supporting images (multipart upload).

### Admins
- Create, update and delete plantation drives.
- Review all donations and approve or reject them.
- Review reports, update their status, record an admin action, or delete them.
- Assign tasks to members and see each member's task count.
- Analytics dashboard (Recharts): total reports, reports by location, by type, by status, and over time.

### Platform
- Public landing page with hero, "why choose us", how-to-donate, team and call-to-action sections.
- API gateway that proxies `/api/auth`, `/user`, `/admin` and `/member` to separate services.
- The member area currently has only a placeholder layout.

## Architecture

```
                 ┌──────────────── API Gateway :5000 ────────────────┐
React client ──▶ │ /api/auth → authService   :5002                   │
 (Vite)          │ /user     → userService   :5003  (donations, reports, drive enrolment, Stripe)
                 │ /admin    → adminService  :5004  (drives, donations, reports, analytics)
                 │ /member   → memberService :5005  (tasks)          │
                 └───────────────────────────────────────────────────┘
                                         │
                                      MongoDB
```

Mongoose models, the MongoDB URL, the CORS config and the upload middleware come from the shared npm packages `models-pms` and `models-aryan`.

## Tech Stack

| Layer | Tools |
|---|---|
| Frontend | React 19, Vite, React Router 7, Redux Toolkit, Axios, MUI 6, Tailwind CSS 4, styled-components, Framer Motion, Recharts, react-slick, Stripe.js (`@stripe/react-stripe-js`) |
| Backend | Node.js (ES modules), Express 4, http-proxy-middleware, Mongoose 8, JSON Web Tokens, bcrypt, multer, Stripe, morgan |
| Database | MongoDB |

## Project Structure

```
ECO-BLOOM/
├── backend/
│   ├── apiGateway/           # server.js: proxies requests to the services
│   └── services/
│       ├── authService/      # login, signup, profile
│       ├── userService/      # donations (Stripe), reports, drive enrolment
│       ├── adminService/     # plantation drives, donation & report review, analytics
│       └── memberService/    # tasks
└── client/
    └── src/
        ├── App.jsx           # routes for /, /auth, /user, /member, /admin
        ├── app/              # landing page, auth forms
        ├── pages/            # user/, admin/, member/ areas
        ├── redux/            # store, user slice and thunks
        ├── libs/constants.js # report types, Dehradun areas
        └── config/config.js  # BASE_URL (the gateway)
```

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB, plus access to the `models-pms` package, which supplies the database URL
- A Stripe test account for donations

### Backend

Each service is its own npm project. Install and start each one in a separate terminal:

```bash
cd backend/apiGateway && npm install && npx nodemon server.js
cd backend/services/authService && npm install && npm run dev
cd backend/services/userService && npm install && npm run dev
cd backend/services/adminService && npm install && npm run dev
cd backend/services/memberService && npm install && npm run dev
```

Optional environment variables: `API_GATEWAY_PORT` (gateway, default `5000`) and `PORT` (user, admin and member services; defaults `5003`, `5004`, `5005`). The auth service always uses `5002`, and the gateway expects the default ports. The Stripe keys are currently set in `userService/controllers/donationController.js` and `client/src/pages/user/components/Payment.jsx`; move them to environment variables before deploying.

### Frontend

```bash
cd client
npm install
npm run dev
```

The client talks to the gateway at `BASE_URL` in `client/src/config/config.js` (`http://localhost:5000`). Other scripts: `npm run build`, `npm run preview`, `npm run lint`.

## API Endpoints

All paths go through the gateway on port 5000.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/login` | Log in |
| `POST` | `/api/auth/signup` | Register (roles: `user`, `member`, `admin`) |
| `GET` | `/api/auth/profile` | Current user (auth required) |
| `POST` | `/user/donations` | Create a donation |
| `GET` | `/user/donations`, `/user/donations/:id` | List or get donations |
| `PUT` | `/user/donations/:id/status` | Update a donation's status |
| `POST` | `/user/donations/create-payment-intent` | Create a Stripe Payment Intent |
| `POST` | `/user/donations/webhook` | Stripe webhook |
| `POST` | `/user/reports` | File a report with `supportingImages` uploads |
| `GET` | `/user/reports`, `/user/reports/:id` | List or get reports |
| `PATCH` | `/user/reports/:id/status` | Update a report's status |
| `DELETE` | `/user/reports/:id` | Delete a report |
| `POST` | `/user/drive/enroll/:driveId` | Enrol a user in a plantation drive |
| `POST` / `GET` | `/admin/drive` | Create or list plantation drives |
| `GET` / `PUT` / `DELETE` | `/admin/drive/:id` | Get, update or delete a drive |
| `GET` | `/admin/donations`, `/admin/donations/:id` | Review donations |
| `PUT` | `/admin/donations/:id` | Approve or reject a donation |
| `GET` | `/admin/reports`, `/admin/reports/:id` | Review reports |
| `PUT` | `/admin/reports/:id/status`, `/admin/reports/:id/action` | Update status or take action |
| `DELETE` | `/admin/reports/:id` | Delete a report |
| `GET` | `/admin/data/reports` | Report analytics (totals, by location, type, status, over time) |
| `GET` / `POST` | `/member/tasks` | List or create tasks |
| `GET` | `/member/tasks/count` | Members with their task counts |
| `GET` / `PUT` / `DELETE` | `/member/tasks/:id` | Get, update or delete a task |

## Author

**Aryan Bhandari**: [GitHub](https://github.com/aryanbha123)
