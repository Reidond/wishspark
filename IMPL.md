# WishSpark Project Implementation Plan (Revised)

## Project Overview

### Project Names

- **Display Name**: WishSpark (A platform that sparks joy through personalized birthday wishlists.)
- **Internal Name**: wishspark-app (Used for repository naming, environment variables, and internal references.)

### Description

WishSpark is a web application designed exclusively for the admin user to manage birthday wishlists. In alignment with the revised requirements, the implementation will utilize only the `@nuxt/content` module of Nuxt 4 for handling content. Wishlists will be stored as static files (e.g., YAML or Markdown) within the `/content` directory, allowing the user to add or edit content directly via file modifications. The application will focus on reading and displaying these wishlists, with no additional database, authentication, or external services required. Management occurs outside the app (e.g., through a code editor or Git), as the user has specified that only they will add content. If in-app editing is desired in the future, it can be extended via server routes, but this is omitted for the MVP to adhere strictly to the constraints.

The application will be built using Nuxt 4 for the framework and Nuxt UI for user interface components, ensuring a responsive and modern design. This setup leverages `@nuxt/content` for querying and rendering content dynamically at build or runtime.

### Goals

- Provide an intuitive interface for viewing and browsing wishlists.
- Enable easy content management through static files, with no runtime write operations.
- Deliver a scalable, self-hosted MVP deployable to platforms like Vercel or a static host.
- Maintain simplicity by avoiding authentication and databases.

## Tech Stack

- **Framework**: Nuxt 4 (for server-side rendering, API routes if needed, and static generation).
- **UI Library**: Nuxt UI (for pre-built components like buttons, modals, tables, and forms).
- **Content Management**: `@nuxt/content` (for querying static files in YAML, Markdown, or JSON format).
- **State Management**: Pinia (integrated with Nuxt, if needed for client-side state).
- **Styling**: Tailwind CSS (included with Nuxt UI).
- **Other Tools**:
  - TypeScript for type safety.
  - ESLint and Prettier for code quality.
  - No external dependencies for auth or DB.

## Features

### Core Features

1. **Wishlist Viewing**:
   - Display all wishlists fetched from static files.
   - View individual wishlist details, including title, description, birthday date, and items (array of {name, link, price, notes}).

2. **Dashboard**:
   - Overview page showing all wishlists in a table or card layout using Nuxt UI components.
   - Search and filter options (e.g., by date or title) implemented via client-side filtering on fetched content.

3. **Additional Functionality**:
   - Export wishlists to PDF or CSV (client-side implementation).
   - Basic validation for display (e.g., formatting dates).
   - Responsive design for mobile and desktop.

### Non-Functional Requirements

- Performance: Utilize static generation where possible for fast loads.
- Security: No auth needed; assume local or private deployment.
- Accessibility: Follow ARIA standards with Nuxt UI components.
- Error Handling: Display user-friendly messages for content loading issues.

## Architecture

- **Frontend**: Nuxt 4 pages and components for UI.
- **Backend/Content**: `@nuxt/content` to query files from `/content/wishlists` directory.
- **Data Flow**:
  - App fetches content at runtime or build time → Display in dashboard → User views details.
- **Content Structure**:
  - Each wishlist as a separate file, e.g., `/content/wishlists/birthday-2025.yaml` with frontmatter or structured data:
    ```yaml
    title: Birthday 2025
    description: Gifts for family
    birthday_date: 2025-08-24
    items:
      - name: Book
        link: https://example.com/book
        price: 20
        notes: Classic edition
    ```
- **Folder Structure** (Standard Nuxt 4 layout):
  ```
  wishspark-app/
  ├── app.config.ts          # App configurations
  ├── assets/                # Static assets (images, etc.)
  ├── components/            # Reusable Nuxt UI-based components (e.g., WishlistCard.vue)
  ├── composables/           # Custom hooks (e.g., useContent.ts)
  ├── content/               # Content directory
  │   └── wishlists/         # Wishlist files (e.g., *.yaml or *.md)
  ├── layouts/               # Default layout with navigation
  ├── pages/                 # Routes (e.g., index.vue for dashboard, wishlists/[slug].vue)
  ├── plugins/               # Plugins if needed
  ├── public/                # Public files
  ├── stores/                # Pinia stores (e.g., wishlistStore.ts, optional)
  ├── nuxt.config.ts         # Nuxt configuration
  ├── package.json           # Dependencies
  ├── tsconfig.json          # TypeScript config
  └── README.md              # Project documentation
  ```

## Implementation Steps

### Step 1: Project Setup

1. Install Node.js (v20+ recommended).
2. Create a new Nuxt 4 project:
   ```
   npx nuxi@latest init wishspark-app
   cd wishspark-app
   npm install
   ```
3. Install dependencies:
   - `@nuxt/content`: `npm install @nuxt/content`
   - Nuxt UI: `npm install @nuxt/ui`
   - Pinia (optional): `npm install pinia @pinia/nuxt`
   - Other: `npm install -D typescript eslint prettier`
4. Configure `nuxt.config.ts`:
   ```typescript
   export default defineNuxtConfig({
     devtools: { enabled: true },
     modules: ['@nuxt/content', '@nuxt/ui', '@pinia/nuxt'],
     content: {
       documentDriven: false, // Optional; enable if using MDX
       sources: {
         wishlists: {
           driver: 'fs',
           prefix: '/wishlists',
           base: 'content/wishlists',
         },
       },
     },
     typescript: { strict: true },
   });
   ```
5. Initialize Git repository and commit initial setup.
6. Create sample content files in `/content/wishlists` to test.

### Step 2: Content Fetching

- Use `$content` composable to query wishlists.
- Create `/composables/useWishlists.ts`:
  ```typescript
  export const useWishlists = async () => {
    const { $content } = useNuxtApp();
    const wishlists = await $content('wishlists').fetch();
    return wishlists;
  };
  ```

### Step 3: Pages and Components

1. **Dashboard Page** (`/pages/index.vue`):
   - Fetch wishlists using the composable.
   - Display in UTable or UCardGroup, with columns for title, date, etc.
   - Implement client-side search/filter using computed properties.

2. **Wishlist Detail Page** (`/pages/wishlists/[slug].vue`):
   - Dynamic route to fetch specific wishlist by slug (e.g., file name without extension).
   - Display details using UCard, UList, etc.
   - Use `useRoute` to get slug and `$content` to fetch.

3. **Components**:
   - WishlistCard.vue: Card for summary display.
   - ItemList.vue: Reusable list for wishlist items.

4. Optional Pinia store `/stores/wishlist.ts` for caching fetched content if needed.

### Step 4: Additional Features

- Implement export: Use libraries like jsPDF (client-side) for PDF export.
  - Install: `npm install jspdf`
  - Add button in dashboard to generate PDF from data.

### Step 5: Testing and Polish

- Add unit tests for components using Vitest (`npm install -D vitest`).
- Implement error handling (e.g., if no content, show message).
- Ensure responsive design with Tailwind.

### Step 6: Deployment

- Build for static hosting: `npm run generate` for static site.
- Deploy to Vercel, Netlify, or self-host via a web server (e.g., Nginx serving the dist folder).

## Timeline Estimate (For One Developer)

- Setup: 1 day.
- Content Fetching and Pages: 1-2 days.
- UI Components and Features: 1-2 days.
- Testing/Polish: 1 day.
- Total: 4-6 days for MVP.

## Potential Extensions

- In-app editing via server routes (if future need arises, using `fs` to write files).
- Integration with Git for versioned content management.
- Public sharing views.

Follow this plan sequentially. Since content addition is manual, include instructions in README.md on how to add/edit files and rebuild/deploy the app. Document any deviations in commit messages.
