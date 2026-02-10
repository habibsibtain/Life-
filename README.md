# LIFE

## Project Overview

**LIFE** is a short‑form video social media application implemented with Next.js and MongoDB.
From the codebase, the primary purpose is:

- **Short‑form video feed**: Users can upload vertical videos, which are displayed in a scrollable, TikTok‑style feed with autoplay behavior.
- **Social interactions**: Users can like posts, comment on videos, and follow/unfollow other users.
- **User accounts**: Users can register and log in via a custom JWT‑based authentication system, then access protected features such as uploading and managing their own content.
- **Profile views**: Each user has a profile page showing their posts and follower/following counts.

Any aspects not evident from the code (such as precise product goals, branding, or non‑functional requirements) are **not documented here**, because they are not present in the repository.

## Tech Stack

### Languages and Frameworks

- **Language**: TypeScript
- **Frontend / Full‑stack framework**: Next.js 15 (App Router, `app` directory)
- **UI library**: React (Next.js integrated React)
- **Styling**:
  - **Tailwind CSS** (with `tailwind.config.js`)
  - **DaisyUI** for pre‑built component styling

### Backend, Data, and Realtime

- **Database**: MongoDB (via **Mongoose**)
- **Authentication**:
  - Custom JWT‑based auth using `jsonwebtoken`
  - Password hashing with `bcryptjs`
  - NextAuth type augmentation exists but is not wired into any runtime logic
- **Media**:
  - Cloudinary SDK configured in `lib/cloudinary.ts`
  - Direct client‑side Cloudinary upload in the video upload component
- **Realtime / WebSockets**:
  - Server‑side Socket.IO (`server/socketServer.ts`)
  - Client‑side Socket.IO (inline setup in profile component; `lib/socketClient.ts` also exists but is unused)

### Tooling and Infrastructure

- **HTTP client**: Axios (used on the client to call API routes)
- **Environment management**: `dotenv` (via Next.js environment handling)
- **Linting**:
  - ESLint (`eslint`, `eslint-config-next`)
- **Build / Dev**:
  - Next.js scripts: `dev`, `build`, `start`, `lint`

Resend SDK appears in dependencies but is **not used** in the code inspected.

## High‑Level Architecture

### Architectural Style

- **Full‑stack monolith** built on **Next.js App Router**:
  - Client UI, API routes, and server logic co‑located in the same codebase.
  - Backend endpoints use Next.js `route.ts` files under `app/api`.
- **Three main layers**:
  - **Presentation layer**: React components in `app` and `components`.
  - **Application/Domain logic layer**: API route handlers under `app/api`, middleware in `authenticate.ts`, and high‑level workflows (upload, like, follow).
  - **Data access layer**: Mongoose models in `models` and database connection logic in `lib/dbConnect.ts`.

### Primary Data Flows

- **Auth flow**: Login/Signup → JWT issuance → JWT in `localStorage` → Bearer token in `Authorization` header → Auth middleware → Protected API route → MongoDB.
- **Feed flow**: Client loads home page → `FeedCard` fetches posts and auth user → Posts rendered as an auto‑playing video list → Users can like/comment → State persisted via API routes.
- **Upload flow**: User selects a video → `VideoUpload` uploads file to Cloudinary → Cloudinary returns `secure_url` → Client posts metadata (`caption`, `mediaURL`) to `/api/posts/upload` → Post persisted in MongoDB.
- **Social graph flow**: User triggers follow/unfollow → `/api/users/follow` or `/api/users/unfollow` updates both users’ follower/following arrays → Intended to integrate with a Socket.IO server for real‑time follower updates (not actually wired up).

## Folder Structure Explanation

### Root

- `package.json`: Declares dependencies (Next.js, React, TypeScript, Mongoose, JWT, bcryptjs, axios, Tailwind, DaisyUI, Socket.IO, etc.) and scripts:
  - `dev`, `build`, `start`, `lint`.
- `next.config.js`: Next.js configuration (e.g., allowed image domains such as DaisyUI CDN).
- `tailwind.config.js`, `postcss.config.js`: Tailwind and PostCSS configuration.
- `tsconfig.json`: TypeScript configuration.
- `authenticate.ts`: Central JWT authentication middleware used by multiple API routes.

### `app/` (Next.js App Router)

- `app/layout.tsx`:
  - Root layout; defines global structure and fonts.
- `app/page.tsx`:
  - Home page, wraps content in `MainLayout` and shows the main video feed (`FeedCard` component).
- `app/login/page.tsx`: Login page, posts to `/api/auth/login`.
- `app/signup/page.tsx`: Signup page, posts to `/api/auth/signup`.
- `app/profile/page.tsx`: Current user profile page (uses `/api/users/me`).
- `app/profile/[id]/page.tsx`: Other user profile page; fetches `/api/users/all` and filters by user id.
- `app/upload/page.tsx`:
  - Upload page: includes `VideoUpload` component; layout includes navbar.
- `app/api/`:
  - API endpoints (described in detail below).
- `app/fonts/`:
  - Geist VF font files, imported in `layout.tsx`.

### `app/api/` – API Routes

- `app/api/auth/login/route.ts`: Login endpoint.
- `app/api/auth/signup/route.ts`: Signup endpoint.
- `app/api/posts/allPosts/route.ts`: Fetch all posts (GET) and toggle likes (POST).
- `app/api/posts/upload/route.ts`: Upload post metadata after Cloudinary upload.
- `app/api/posts/comment/route.ts`: Create new comment.
- `app/api/posts/comment/[postId]/route.ts`: Fetch single post plus populated comments.
- `app/api/posts/adminPosts/route.ts`: Get posts created by the current user.
- `app/api/users/all/route.ts`: Get all users (except current) and all posts.
- `app/api/users/me/route.ts`: Get current user and their posts.
- `app/api/users/follow/route.ts`, `unfollow/route.ts`: Follow/unfollow logic.

### `components/`

- `FeedCard.tsx`:
  - Core home feed: handles fetching posts, infinite/snap scrolling, video playback, likes, and comments through an inner `VideoCard` component.
- `Navbar.tsx`:
  - Top navigation (desktop) and bottom bar (mobile); includes:
    - App logo/title.
    - Search input (non‑functional).
    - Upload button.
    - Profile avatar and dropdown (Profile, Logout).
- `Sidebar.tsx`:
  - Left sidebar with three tabs: For You, Following, Trending (stateful only).
  - List of suggested users from `/api/users/all` with follow/unfollow controls.
- `VideoUpload.tsx`:
  - File input, preview, caption input, and upload button.
  - Performs Cloudinary upload and then posts media metadata to `/api/posts/upload`.
- `SwitchTabs.tsx`:
  - Simple client component for switching between “Videos” and “Posts” within profile views.
- `Signin.tsx`:
  - Placeholder for Google Sign‑In; actual NextAuth logic is commented out and not active.

### `Layouts/`

- `Layouts/MainLayout.tsx`:
  - Composes `Navbar`, `Sidebar`, and the main content area.
  - Used by major application pages (home, profiles, etc.).

### `lib/`

- `lib/dbConnect.ts`:
  - Singleton pattern for connecting to MongoDB using `MONGODB_URI` and database name `"LIFE"`.
- `lib/cloudinary.ts`:
  - Configures Cloudinary with `CLOUDINARY_*` environment variables.
  - Exposes `uploadOnCloudinary(mediaURL)` (currently unused by upload page).
- `lib/socketClient.ts`:
  - Exports a Socket.IO client pointing to `https://life-black.vercel.app/api`.
  - Not used in current components; profile page sets up its own Socket.IO connection inline.

### `models/`

- `models/userSchema.ts`:
  - Mongoose model for `User`.
- `models/postSchema.ts`:
  - Mongoose model for `Post`.
- `models/commentSchema.ts`:
  - Mongoose model for `Comment`.

### `server/`

- `server/socketServer.ts`:
  - Exports `initializeSocketServer(server)` to attach Socket.IO to a Node HTTP server.
  - Defines events (`connection`, `joinRoom`, `disconnect`) and emits `followerUpdate` on follow.
  - Not integrated into a running server in this repository.

### `types/`

- `types/next-auth.d.ts`:
  - Declares module augmentation for NextAuth (user fields like `id`, `email`, etc.).
  - NextAuth itself is not operational in the codebase; this is unused.

### `public/`

- Static assets such as icons (e.g., for the app logo).

## Core Modules and Features

### Authentication Middleware (`authenticate.ts`)

- **Purpose**:
  - Provide reusable JWT verification for API routes.
- **Behavior**:
  - Reads `Authorization` header, expects `Bearer <token>`.
  - Verifies JWT using `JWT_SECRET`.
  - Extracts `userId` from token payload.
  - Queries `User` via Mongoose (`User.findById`).
  - On success: attaches user to the request and allows handler to continue.
  - On failure: responds with appropriate 401/404 JSON.

### Auth API

- `app/api/auth/signup/route.ts`:
  - Receives signup data (username, fullname, email, password).
  - Hashes password with `bcryptjs`.
  - Checks for existing user by email; if found, logic does not clearly return an error (limitation).
  - Creates a new user document and saves.
  - Issues a JWT with user id and an `expiresIn` of 1 hour.
- `app/api/auth/login/route.ts`:
  - Receives login credentials (username and password; the login form also collects email but API uses username).
  - Verifies user existence and password (`bcryptjs.compare`).
  - Issues a JWT (no `expiresIn` set).
  - Returns `{ success, user, token }`.

### Post Management

- `app/api/posts/upload/route.ts`:
  - Protected by `authenticate.ts`.
  - Expects `caption` and `mediaURL` in request body.
  - Creates a `Post` with current user’s id, username, fullname, and the provided media URL.
- `app/api/posts/allPosts/route.ts`:
  - **GET**:
    - Protected by `authenticate.ts`.
    - Returns all posts sorted (newest first).
  - **POST**:
    - Protected by `authenticate.ts`.
    - Toggles like on a post:
      - Body includes `postId` and `userId`.
      - If user is in `likedBy`, removes them; else adds them.
      - Updates `likes` count as the length of `likedBy`.
- `app/api/posts/adminPosts/route.ts`:
  - Protected by `authenticate.ts`.
  - Returns posts created by the authenticated user.
- `app/api/posts/comment/route.ts`:
  - **POST**:
    - Not protected by `authenticate.ts`.
    - Accepts `{ userId, postId, comment }`.
    - Creates a `Comment` document and pushes its id into the `Post.comments` array.
- `app/api/posts/comment/[postId]/route.ts`:
  - **GET**:
    - Fetches a single `Post` by `postId`.
    - Uses `.populate("comments")` to return full comment objects.
    - No auth check.

### User Management

- `app/api/users/me/route.ts`:
  - Protected by `authenticate.ts`.
  - Retrieves current user from auth middleware.
  - Fetches user and their posts from MongoDB.
- `app/api/users/all/route.ts`:
  - Protected by `authenticate.ts`.
  - Returns:
    - All users except the current user.
    - All posts (for cross‑referencing in the UI).
- `app/api/users/follow/route.ts`:
  - Protected by `authenticate.ts`.
  - Body: `{ targetedUserId }`.
  - Adds `currentUser._id` to the target’s `followers` array and adds `targetedUserId` to current user’s `following` array.
  - Includes Socket.IO emission of `followerUpdate` (but server initialization is missing).
- `app/api/users/unfollow/route.ts`:
  - Protected by `authenticate.ts`.
  - Symmetric to follow: removes ids from `followers` and `following` arrays.

### Feed and Video Components

- `components/FeedCard.tsx`:
  - Fetches:
    - Posts from `/api/posts/allPosts` (with token).
    - Current user from `/api/users/me`.
  - Represents posts as a vertical feed of videos.
  - Uses an `IntersectionObserver` to manage auto‑play/pause behavior for each `VideoCard` based on viewport visibility.
  - Passes callbacks to toggle likes and open comment modals, making API calls with `axios`.
- `VideoCard` (inner component):
  - Renders a `video` element with `mediaURL`.
  - Shows username, caption, like count, and comment count.
  - Handles:
    - Tap/click to play/pause video.
    - Like button calling `POST /api/posts/allPosts`.
    - Comment button to open comments and call `/api/posts/comment` APIs.

### Upload Component

- `components/VideoUpload.tsx`:
  - Selects a video file from the local device.
  - Previews it in a `<video>` element.
  - On submit:
    - Uploads to Cloudinary using `fetch` to:
      - `https://api.cloudinary.com/v1_1/dcphrjb6y/video/upload`
      - with `upload_preset: "LifeMedia"`.
    - Takes `secure_url` from Cloudinary response.
    - Sends POST to `/api/posts/upload` with caption and `secure_url`.
    - Requires Authorization header with JWT.
  - Uses hardcoded Cloudinary cloud name and preset rather than `lib/cloudinary.ts`.

### Layout and Navigation

- `Layouts/MainLayout.tsx`:
  - Wraps content in `Navbar` and `Sidebar`.
- `components/Navbar.tsx`:
  - Handles:
    - Navigation to `/upload`, `/profile`, and home.
    - Display of current user avatar (currently using a generic DaisyUI URL).
    - Logout: clears `localStorage` token and redirects to `/login`.
- `components/Sidebar.tsx`:
  - Maintains state of active section: For You / Following / Trending.
  - Fetches suggested users from `/api/users/all`.
  - Provides follow/unfollow interactions via user APIs.

### Profile Pages

- `app/profile/page.tsx` (own profile):
  - Calls `/api/users/me` for current user and their posts.
  - Renders follower/following counts.
  - Uses `SwitchTabs` to show user posts.
- `app/profile/[id]/page.tsx` (other profile):
  - Calls `/api/users/all`.
  - Filters the returned users array to find the matching `id`.
  - Determines follow/unfollow state by checking arrays.
  - Attempts to integrate Socket.IO for real‑time follower updates, but:
    - Event names do not match server.
    - `initializeSocketServer` is not invoked anywhere.

## Database Models and Schemas

All schemas are implemented with Mongoose.

### `User` (`models/userSchema.ts`)

- **Fields**:
  - `fullname: string` (required).
  - `username: string` (required, unique).
  - `email: string` (required, unique).
  - `password: string` (required, hashed).
  - `followers: ObjectId[]` (refs to `User`).
  - `following: ObjectId[]` (refs to `User`).
- **Type mismatch**:
  - Interface includes `profilePic` but schema has no `profilePic` field.

### `Post` (`models/postSchema.ts`)

- **Fields**:
  - `userId: ObjectId` (ref `User`).
  - `username: string`.
  - `fullname: string`.
  - `caption: string`.
  - `mediaURL: string`.
  - `likes: number` (count, derived from `likedBy.length`).
  - `likedBy: ObjectId[]` (refs to `User` who liked the post).
  - `comments: ObjectId[]` (refs to `Comment`).
  - `createdAt: Date` (default `Date.now`).

### `Comment` (`models/commentSchema.ts`)

- **Fields**:
  - `comment: string`.
  - `userId: ObjectId` (ref `User`).
  - `createdAt: Date` (default `Date.now`).

## API Endpoints and Routes

All routes under `app/api` are Next.js App Router route handlers.

### Authentication

- **POST `/api/auth/signup`**
  - **Input**: JSON with at least `fullname`, `username`, `email`, `password`.
  - **Output (success)**: `{ success: true, newUser, token }`, where `token` is a JWT with 1‑hour expiry.
  - **Notes**: If user already exists by email, behavior is ambiguous (no explicit error returned).

- **POST `/api/auth/login`**
  - **Input**: JSON including `username` and `password` (login form also sends email but API uses username).
  - **Output (success)**: `{ success: true, user, token }`, where `token` is a JWT without explicit expiry setting.

### Posts

- **GET `/api/posts/allPosts`**
  - **Auth**: Required (Bearer token).
  - **Output**: JSON array of all posts, newest first.

- **POST `/api/posts/allPosts`**
  - **Auth**: Required.
  - **Input**: `{ postId, userId }`.
  - **Behavior**:
    - Toggles the like state for `userId` on the specified `postId`.
    - Updates `likes` field based on `likedBy.length`.

- **POST `/api/posts/upload`**
  - **Auth**: Required.
  - **Input**: `{ caption, mediaURL }`.
  - **Behavior**:
    - Creates a new post for the authenticated user.

- **POST `/api/posts/comment`**
  - **Auth**: Not enforced.
  - **Input**: `{ userId, postId, comment }`.
  - **Behavior**:
    - Creates a `Comment`, then pushes comment ID into the post’s `comments`.

- **GET `/api/posts/comment/[postId]`**
  - **Auth**: Not enforced.
  - **Output**:
    - The single post with populated `comments` array.

- **GET `/api/posts/adminPosts`**
  - **Auth**: Required.
  - **Output**:
    - Posts created by the authenticated user.

### Users

- **GET `/api/users/me`**
  - **Auth**: Required.
  - **Output**:
    - Current user entity and their posts.

- **GET `/api/users/all`**
  - **Auth**: Required.
  - **Output**:
    - Array of all users except the current user.
    - All posts (for UI cross‑reference).

- **POST `/api/users/follow`**
  - **Auth**: Required.
  - **Input**: `{ targetedUserId }`.
  - **Behavior**:
    - Adds `targetedUserId` to current user’s `following`.
    - Adds current user’s id to targeted user’s `followers`.
    - Emits `followerUpdate` via Socket.IO (server integration missing).

- **POST `/api/users/unfollow`**
  - **Auth**: Required.
  - **Input**: `{ targetedUserId }`.
  - **Behavior**:
    - Symmetric removal from follow/follower arrays.

## Business Logic and Workflows

### Authentication Workflow

1. **Signup**:
   - User submits signup form.
   - API checks for existing email and (intended) returns an error if present, but this is incomplete.
   - On success: creates `User` with hashed password and returns JWT plus user data.
2. **Login**:
   - User submits login form.
   - API verifies credentials and returns JWT plus user data.
3. **Token storage**:
   - Client stores JWT (and often `userId`) in `localStorage`.
4. **Authorized requests**:
   - Client attaches `Authorization: Bearer <token>` header to protected calls.
   - `authenticate.ts` decodes JWT, fetches user, and passes control to handlers.

### Feed and Interaction Workflow

1. Home page loads:
   - `FeedCard` component fetches posts via `/api/posts/allPosts` with token.
2. Feed rendering:
   - Posts are mapped into vertical `VideoCard` components.
   - `IntersectionObserver` determines which video is in view to auto‑play/pause.
3. Liking:
   - Clicking like calls `POST /api/posts/allPosts` with `postId` and `userId`.
   - API toggles the like status and returns updated post.
4. Commenting:
   - User opens a comment UI in `VideoCard`.
   - POST `/api/posts/comment` adds comment; GET `/api/posts/comment/[postId]` fetches updated comments.

### Upload Workflow

1. User navigates to `/upload`.
2. `VideoUpload` component:
   - Lets user choose a video file.
   - Previews video.
3. On submission:
   - Component uploads file to Cloudinary via an unsigned upload endpoint.
   - On Cloudinary success, obtains `secure_url`.
   - Sends POST to `/api/posts/upload` with JWT auth.
   - On success, navigates back to home or profile as implemented.

### Follow/Unfollow and Social Graph

1. From `Sidebar` or profile pages:
   - User clicks Follow/Unfollow.
2. The client sends `POST /api/users/follow` or `/api/users/unfollow` with target user id.
3. API updates both user records accordingly.
4. In theory, Socket.IO server should emit follower updates:
   - `server/socketServer.ts` emits `followerUpdate`.
   - Client code on profile page listens to `followersUpdated` and emits `updateFollowers`, which do not match the server’s event names.
5. In practice, real‑time updates do not function because:
   - Socket server is not initialized in any runtime entry point.
   - Event namespaces and signatures differ.

## Configuration and Environment Variables

### Environment Variables

The following variables are referenced in the code (names inferred from usage):

- `MONGODB_URI`
  - Used in `lib/dbConnect.ts`.
  - MongoDB connection string.
- `JWT_SECRET`
  - Used in `authenticate.ts`, login, and signup routes.
  - Secret key to sign and verify JWT tokens.
- `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`
  - Used in `lib/cloudinary.ts`.
  - Configure Cloudinary SDK for server‑side use.

The actual `.env` file is not part of the repository, so exact values and additional environment variables (if any) are not visible.

### Tailwind and Next.js Configuration

- `tailwind.config.js`:
  - Configures Tailwind CSS and DaisyUI themes.
- `next.config.js`:
  - Configures Next.js, including allowed remote image domains (e.g., DaisyUI CDN for placeholder avatars).

## Setup and Installation

Based strictly on observed scripts and dependencies.

1. **Prerequisites**
   - Node.js (version compatible with Next.js 15).
   - MongoDB (local instance or managed cluster).
   - Cloudinary account (for video upload).
2. **Clone the repository**
   - `git clone <repository-url>`
   - `cd Life-`
3. **Install dependencies**
   - `npm install`
4. **Configure environment**
   - Create `.env.local` (or `.env`) with:

     ```bash
     MONGODB_URI="your-mongodb-uri"
     JWT_SECRET="your-secure-jwt-secret"
     CLOUDINARY_CLOUD_NAME="your-cloud-name"
     CLOUDINARY_API_KEY="your-api-key"
     CLOUDINARY_API_SECRET="your-api-secret"
     ```

   - Ensure your Cloudinary account has an unsigned preset named `LifeMedia`, or adjust the upload component.
5. **Optional tooling**
   - Run `npm run lint` to analyze TypeScript and JavaScript files.

## How to Run the Project

### Development

- Start the development server:

  ```bash
  npm run dev
  ```

- By default, the application is served at `http://localhost:3000`.

### Production

1. Create production build:

   ```bash
   npm run build
   ```

2. Start the production server:

   ```bash
   npm run start
   ```

No Dockerfile or deployment scripts are present. Deployment to platforms such as Vercel or a custom Node server is possible but not codified in this repository, and Socket.IO integration for production is not configured.

## Assumptions and Design Decisions

### Authentication and Security

- **Custom JWT** is used instead of framework‑managed auth (NextAuth is only partially present as typings).
- Tokens are stored client‑side, apparently in `localStorage`.
- Protected endpoints expect `Authorization: Bearer <token>`; no Next.js middleware is used to guard routes at the edge.

### Database and Data Model

- MongoDB and Mongoose are used for persistence.
- Social graph is represented via arrays of ObjectId references (`followers`, `following`).
- Posts denormalize user data by storing `username` and `fullname` with `userId`.

### Media Handling

- Client‑side uploads to Cloudinary offload large media transfers directly from the client.
- The server stores only `mediaURL`, not file content.
- `lib/cloudinary.ts` is configured but unused in the actual upload flow, which uses a direct HTTP POST from the browser with a hardcoded cloud name and upload preset.

### Realtime Design

- Socket.IO is introduced for follower updates but not fully wired.
- `initializeSocketServer` is defined but not invoked.
- Event naming is inconsistent between server and client.

## Limitations and Known Issues

- **Signup handling for existing users is incomplete**:
  - Signup route checks for existing email but does not reliably respond with an explicit error.
- **JWT expirations are inconsistent**:
  - Signup token has `expiresIn: "1h"`; login token does not specify expiration.
- **LocalStorage‑based auth**:
  - Tokens stored in `localStorage` are vulnerable to XSS compared to httpOnly cookies.
- **Comment endpoint lacks auth**:
  - `/api/posts/comment` relies on `userId` from the request body instead of JWT.
- **Profile schema inconsistency**:
  - `profilePic` appears in TypeScript interfaces but is missing from MongoDB schema and signup logic.
- **Socket.IO integration is incomplete**:
  - `server/socketServer.ts` is never initialized, and event names between server and client mismatch.
- **Profile data fetching for other users is inefficient**:
  - `/profile/[id]` loads all users via `/api/users/all` and filters client‑side, instead of using a dedicated endpoint.
- **Search is not implemented**:
  - Navbar search input does not trigger any behavior.
- **Sidebar tabs are not bound to feed logic**:
  - For You / Following / Trending only update local state; they do not change the posts queried or displayed.
- **Cloudinary configuration discrepancy**:
  - `VideoUpload` uses hardcoded Cloudinary values, while `lib/cloudinary.ts` uses environment variables.
- **Lack of tests**:
  - No automated tests (unit, integration, or E2E) are present.
- **No explicit deployment strategy**:
  - No Dockerfile, CI/CD, or server configuration for Socket.IO.

## Future Improvements

- Harden authentication:
  - Align JWT expiry behavior between signup and login.
  - Consider secure httpOnly cookies plus Next.js middleware for route protection.
  - Improve error handling and feedback for duplicate signup attempts.
- Secure comment creation:
  - Protect `/api/posts/comment` with `authenticate.ts` and derive `userId` from JWT.
- Improve user and profile APIs:
  - Introduce `GET /api/users/[id]` for individual profiles.
  - Avoid loading all users when only one is needed.
- Complete realtime integration:
  - Properly initialize `initializeSocketServer` in a server entrypoint.
  - Standardize Socket.IO event names and payloads.
- Consolidate Cloudinary usage:
  - Use `lib/cloudinary.ts` or environment‑driven configuration consistently for uploads.
- Resolve data model inconsistencies:
  - Add `profilePic` to `User` schema or remove it from interfaces.
- Enhance UX and feature set:
  - Implement functional search and feed filtering (For You, Following, Trending).
  - Improve loading and error states in the UI.
- Add tests and tooling:
  - Introduce unit, integration, and E2E tests.
  - Tighten ESLint rules (unused imports, security best practices).

## Conclusion

LIFE is a Next.js, MongoDB‑backed short‑video social platform featuring JWT‑protected user accounts, a TikTok‑like video feed, likes, comments, follow/unfollow, and profile views.
The system is structured as a monolithic Next.js App Router application with clear separation between UI components, API routes, and database models.
Several advanced capabilities—particularly real‑time follower updates, fully robust authentication flows, and richer discovery/search—are partially implemented or implied but not complete.
With targeted work on security, API design, realtime integration, and testing, this codebase can evolve into a production‑ready project suitable for academic evaluation and professional review.