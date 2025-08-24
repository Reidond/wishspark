# WishSpark Project Implementation Plan (Revised)

## Project Overview

### Project Names

- **Display Name**: WishSpark (A platform that sparks joy through personalized birthday wishlists.)
- **Internal Name**: wishspark (Used for repository naming, environment variables, and internal references.)

### Description

WishSpark is a web application for managing birthday wishlists, with exclusive admin access to create, edit, delete, and view wishlists. In response to the preference for self-hosted solutions and the assessment that Supabase may introduce unnecessary complexity for this use case, the data management options have been adjusted. The application will support static file-based storage using JSON files for simplicity, or a full-fledged self-hosted database. For the database option, PocketBase is recommended over a standalone PostgreSQL instance, as it provides an integrated backend solution including a lightweight SQLite database, built-in authentication, RESTful API, and realtime capabilities—all self-hostable with minimal setup. If PostgreSQL is preferred, it can be used with Drizzle ORM for database interactions, though this requires additional configuration for authentication and API exposure.

This revised plan maintains the focus on a functional MVP, built with Nuxt 4 and Nuxt UI, while ensuring all components are self-hosted to align with the specified requirements.

### Goals

- Deliver an intuitive interface for wishlist management.
- Ensure secure, self-hosted admin access without reliance on cloud services.
- Support scalability for future enhancements.
- Provide a deployable MVP suitable for self-hosting on a server or local environment.

## Tech Stack

- **Framework**: Nuxt 4 (for server-side rendering, API routes, and static generation).
- **UI Library**: Nuxt UI (for pre-built components such as buttons, modals, tables, and forms).
- **Data Storage Options**:
  - **Static Files**: JSON files stored in the project's `/data` directory, managed via Nuxt's Nitro server routes.
  - **Database**: PocketBase (self-hosted backend with SQLite) for persistent storage and authentication. Alternatively, self-hosted PostgreSQL with Drizzle ORM if a relational database is strictly required.
- **Authentication**: Built-in email/password via PocketBase (for database mode) or a simple session-based password for static files. Google OAuth is omitted to prioritize self-hosting; if needed, it could be implemented via a self-hosted identity provider, but this is beyond the MVP scope.
- **State Management**: Pinia (integrated with Nuxt).
- **Styling**: Tailwind CSS (included with Nuxt UI).
- **Other Tools**:
  - TypeScript for type safety.
  - ESLint and Prettier for code quality.
  - Self-hosting: Deployable on a VPS (e.g., via Docker) or locally; no cloud-specific dependencies.

## Features

### Core Features

1. **Authentication**:
   - Admin login using email/password (PocketBase mode) or a simple session-based password (static mode).
   - Logout functionality.
   - Redirect unauthorized users to the login page.

2. **Wishlist Management**:
   - Create new wishlists with fields: title, description, date (birthday), items (array of {name, link, price, notes}).
   - Edit existing wishlists.
   - Delete wishlists.
   - View all wishlists in a dashboard with search and filter options (by date or title).

3. **Dashboard**:
   - Overview page displaying all wishlists in a table or card layout.
   - Summary statistics (e.g., total wishlists, upcoming birthdays).

4. **Additional Functionality**:
   - Export wishlists to PDF or CSV.
   - Basic validation for form inputs.
   - Responsive design for mobile and desktop.

### Non-Functional Requirements

- Performance: Leverage server-side rendering for efficient loads.
- Security: Protect API routes; use environment variables for sensitive data; ensure PocketBase or PostgreSQL is secured with proper access controls.
- Accessibility: Adhere to ARIA standards via Nuxt UI components.
- Error Handling: Present user-friendly messages for issues such as network failures.

## Architecture

- **Frontend**: Nuxt 4 pages and components for the UI.
- **Backend**:
  - For static: Nuxt server routes (`/server/api`) to handle JSON file operations.
  - For database: PocketBase as a self-hosted server, accessed via its JS SDK for CRUD operations. If using PostgreSQL, integrate directly via Drizzle in Nuxt server routes.
- **Data Flow**:
  - User authenticates → Access dashboard → Perform CRUD on wishlists via API calls.
- **Folder Structure** (Standard Nuxt 4 layout, with adjustments):

  ```
  wishspark/
  ├── app.config.ts          # App configurations
  ├── assets/                # Static assets
  ├── components/            # Reusable components (e.g., WishlistForm.vue)
  ├── composables/           # Custom hooks (e.g., useAuth.ts)
  ├── data/                  # For static JSON files (e.g., wishlists.json)
  ├── layouts/               # Default layout with navigation
  ├── middleware/            # Auth middleware
  ├── pages/                 # Routes (e.g., index.vue for dashboard)
  ├── plugins/               # Plugins (e.g., PocketBase client)
  ├── public/                # Public files
  ├── server/                # API routes (for static or PostgreSQL mode)
  ├── stores/                # Pinia stores (e.g., wishlistStore.ts)
  ├── nuxt.config.ts         # Nuxt configuration
  ├── package.json           # Dependencies
  ├── tsconfig.json          # TypeScript config
  └── README.md              # Documentation
  ```

  - PocketBase would run as a separate self-hosted service (e.g., via Docker), with the Nuxt app connecting to its API endpoint.

## Database Schema (For Database Option)

- **PocketBase Collection: wishlists**
  - id: string (auto-generated)
  - title: string (required)
  - description: text (optional)
  - birthday_date: date (required)
  - items: json (array of objects: {name: string, link: string, price: number, notes: text})
  - created: timestamp (auto)
  - updated: timestamp (auto)
  - Restrict access to admin via PocketBase's record rules.

- If using PostgreSQL, mirror the above schema with an additional `admin_id` for ownership.

## Implementation Steps

### Step 1: Project Setup

1. Install Node.js (v20+ recommended).
2. Initialize Nuxt 4 project:
   ```
   npx nuxi@latest init wishspark
   cd wishspark
   npm install
   ```
3. Install dependencies:
   - Nuxt UI: `npm install @nuxt/ui`
   - Pinia: `npm install pinia @pinia/nuxt`
   - For PocketBase: `npm install pocketbase`
   - For PostgreSQL (alternative): `npm install drizzle-orm drizzle-kit postgres`
   - Other: `npm install -D typescript eslint prettier`
4. Configure `nuxt.config.ts`:
   ```typescript
   export default defineNuxtConfig({
     devtools: { enabled: true },
     modules: ['@nuxt/ui', '@pinia/nuxt'],
     typescript: { strict: true },
   });
   ```
5. Set up environment variables in `.env`:
   ```
   POCKETBASE_URL=http://localhost:8090  # Or your self-hosted URL
   ADMIN_EMAIL=admin@example.com
   ADMIN_PASSWORD=secret_password
   # For PostgreSQL: PG_CONNECTION_STRING=postgres://user:pass@localhost:5432/db
   ```
6. For PocketBase: Download and run PocketBase locally or via Docker; create an admin account and the `wishlists` collection.
7. Initialize Git and commit the setup.

### Step 2: Authentication

- **Static Mode**: Implement middleware and sessions using `useCookie` for password-based auth.
- **PocketBase Mode**: Use the PocketBase JS SDK for admin auth (login with email/password); store auth token in cookies.
- **PostgreSQL Mode**: Handle auth manually (e.g., via JWT in Nuxt server routes).
- Apply middleware to protect dashboard and wishlist routes.

### Step 3: Data Management

- **Static Mode**: Use `/data/wishlists.json` with `fs` in server routes for CRUD.
- **PocketBase Mode**: Integrate SDK in composables for collection operations.
- **PostgreSQL Mode**: Set up Drizzle in `/server/utils/db.ts`; implement CRUD in server routes.

### Step 4: Pages and Components

1. **Login Page** (`/pages/login.vue`): Form for email/password.
2. **Dashboard Page** (`/pages/index.vue`): UTable for wishlists, fetched via store.
3. **Wishlist Detail/Edit Page** (`/pages/wishlists/[id].vue`): UForm for CRUD.
4. **Components**: WishlistCard.vue, WishlistForm.vue.
5. Pinia store `/stores/wishlist.ts` for state management.

### Step 5: API Routes (For Static or PostgreSQL)

- Define routes like `/server/api/wishlists.get.ts` for CRUD, protected by auth.

### Step 6: Testing and Polish

- Add tests with Vitest.
- Implement error handling with UToast.
- Ensure responsiveness.

### Step 7: Deployment

- Self-host: Run Nuxt via `npm run build` and `npm run start`; host PocketBase/PostgreSQL on the same server (e.g., via Docker Compose for orchestration).

## Timeline Estimate (For One Developer)

- Setup and Auth: 1-2 days.
- Data and API: 1-2 days (PocketBase simplifies this).
- UI Pages/Components: 2-3 days.
- Testing/Polish: 1 day.
- Total: 5-8 days for MVP.

## Potential Extensions

- Public sharing of wishlists.
- Birthday reminders via email.
- Multi-admin capabilities.

Proceed sequentially with this revised plan. If preferring PostgreSQL over PocketBase, note that it may require more boilerplate for auth and API management. Document changes in commits.
