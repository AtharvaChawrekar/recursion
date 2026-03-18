🗺️ SyncRoute — Project Memory & Knowledge Base 
> **Stack:** Next.js 16 · React 19 · TypeScript · Supabase · Groq (LLaMA 3.3-70B) · Tailwind CSS v4 · Framer Motion
---
📌 Table of Contents
What is SyncRoute?
Tech Stack Overview
Project Structure
Application Flow (End-to-End)
Pages & Routes
API Routes
Hooks & State Management
Services
Key Components
Database Schema (Supabase)
AI Intelligence Layer
Feature Summary Table
PPT Presentation Slides
---
1. What is SyncRoute?
SyncRoute is an AI-powered collaborative travel itinerary planning platform. It is not a traditional travel agency — it is an intelligent planner that:
Lets users chat naturally with an AI assistant named @Safar (powered by Groq's LLaMA 3.3-70B)
Generates day-by-day itineraries from conversational requests
Monitors real-time travel disruptions (weather, flight delays) and automatically replans affected activities
Supports collaborative trip planning — multiple users can chat in the same trip workspace
Exports finalized itineraries as PDF documents
Provides an immersive map preview using Leaflet.js with satellite/dark map tiles
---
2. Tech Stack Overview
Layer	Technology
Framework	Next.js 16 (App Router)
Language	TypeScript 5
UI	React 19, Tailwind CSS v4, shadcn/ui, Radix UI
Animation	Framer Motion 12
Icons	Lucide React
Backend / DB	Supabase (PostgreSQL + Auth + Realtime)
AI Model	Groq API — `llama-3.3-70b-versatile`
AI Streaming	SSE (Server-Sent Events) via Next.js API route
Maps	Leaflet.js (dynamically imported, client-only)
PDF Export	jsPDF + file-saver
Weather API	OpenWeatherMap
Flight API	AviationStack
Fonts	Poppins (body), Bebas Neue (headings)
Toasts	react-hot-toast
Theme	next-themes (light/dark/system)
---
3. Project Structure
```
src/
├── app/
│   ├── page.tsx                  ← Landing / Marketing page
│   ├── layout.tsx                ← Root layout (fonts, theme, toaster)
│   ├── globals.css               ← Global styles
│   ├── onboarding/page.tsx       ← 5-step onboarding + sign-up
│   ├── dashboard/page.tsx        ← Main 3-pane app (1587 lines)
│   ├── immersive-preview/page.tsx← Leaflet map + itinerary virtual tour
│   └── api/
│       ├── safar/route.ts        ← Streaming AI chat (SSE, Groq)
│       ├── safar-itinerary/route.ts ← Structured JSON itinerary (Groq)
│       └── generate-pdf/route.ts ← PDF generation (jsPDF)
├── components/
│   ├── auth-buttons.tsx          ← Login / Signup modal (Supabase Auth)
│   ├── mode-toggle.tsx           ← Light/dark toggle
│   ├── theme-provider.tsx        ← next-themes wrapper
│   └── ui/                       ← shadcn components (button, card, dialog…)
├── hooks/
│   ├── useSyncRoute.ts           ← Core data hooks (auth, trips, messages, invitations, typing)
│   └── useItinerary.ts           ← AI itinerary generation, persistence, live alerts
├── lib/
│   ├── supabase.ts               ← Supabase client singleton
│   └── utils.ts                  ← Tailwind class merge utility
└── services/
    └── liveTravelData.ts         ← OpenWeatherMap + AviationStack API wrappers
```
---
4. Application Flow (End-to-End)
```
[Landing Page]
     │
     ├── New User → [Onboarding (5 steps)] → Supabase Auth signUp + profile insert → [Dashboard]
     │
     └── Existing User → Auth Modal (Login) → Supabase Auth signInWithPassword → [Dashboard]
                                                                                        │
                          ┌─────────────────────────────────────────────────────────────┤
                          │                                                               │
              [Left Sidebar]                  [Center: Chat]              [Right: Itinerary Panel]
             ─────────────────           ───────────────────           ──────────────────────────
             • Workspace (Safar DM)      • Real-time messages          • AI-generated day-by-day
             • Your Trips (list)           (Supabase Realtime)           itinerary (from Supabase)
             • New Trip creation         • @Safar mention triggers      • Live travel alerts
             • Collaborator invites        Groq AI response              (weather + flight)
             • Invitation inbox          • Voice input (Web Speech)     • One-click Replan
             • Profile + Settings        • Swipe-to-reply               • Download PDF
                                         • Typing indicators
                                                  │
                                     @Safar plan a trip to Goa
                                                  │
                                    ┌─────────────▼──────────────┐
                                    │   /api/safar-itinerary      │
                                    │   Groq LLaMA 3.3-70B        │
                                    │   Returns structured JSON   │
                                    └─────────────┬──────────────┘
                                                  │
                                    Saved to Supabase trips.itinerary_data
                                    Displayed live in Right Panel
                                    Live alerts auto-fetched from APIs
```
---
5. Pages & Routes
`/` — Landing Page (`src/app/page.tsx`)
Navbar: Logo, nav links (Destinations, About, Contact), Auth buttons (Login/Sign Up), Dark/Light toggle
Hero Section: Full-bleed image grid with animated overlays, tagline "Create Unforgettable Memories"
Search Bar (floating): Inputs for Destination, Date, Price — purely visual/UI placeholder
About Section: Stats (100+ itineraries, 10K+ POIs), description of the product
Gallery Section: 4-image adventure gallery
Testimonial Section: User review with star rating
Footer: Logo, social links, quick links, contact info, copyright
`/onboarding` — Onboarding (`src/app/onboarding/page.tsx`)
A 5-step animated glassmorphic wizard on a travel-photo background:
Step	Title	Captures
1	Set your travel rhythm	Daily budget (₹500–₹25,000), travel pace (slider), preferred hours (custom time picker)
2	How do you move and eat?	Dietary preference (Veg/Egg/Non-Veg/Vegan/Jain), preferred transport modes
3	Who do you explore with?	Travel style (Solo / With Friends / With Family)
4	What makes a trip memorable?	Interests (15 tags: Adventure, Food, Trekking, etc.)
5	Create Account	Username, Email, Password — saved to Supabase Auth + `users` table
On submit: Creates Supabase auth user → inserts profile with preferences JSON → redirects to `/dashboard`.
`/dashboard` — Main App (`src/app/dashboard/page.tsx`)
The core 3-pane layout:
Left Sidebar (collapsible)
Search bar (filter trips by name)
"New Trip" button → creates trip in Supabase
Workspace section: pinned "Safar AI" private DM (auto-created on first login)
Your Trips list: color-coded avatars, private/shared indicator, 3-dot context menu (Pin, Share, Rename, Delete)
Invitation inbox button with badge count
User profile snippet at bottom with Settings modal
Center: Chat
Real-time messages (Supabase Realtime `INSERT` subscription)
AI messages styled with gradient + Sparkles icon (streaming via SSE)
Swipe-to-reply on incoming messages (Framer Motion drag)
Typing indicator (Supabase Broadcast channel — no DB writes)
Input bar with: voice input (Web Speech API), @mention autocomplete popup, send button
Optimistic message updates (temp ID → real ID swap)
Right Panel: Itinerary
Shows AI-generated itinerary fetched from Supabase
Live travel alerts (weather + flight) auto-fetched after itinerary loads
"Replan Itinerary" button per alert
Day cards with timeline, activity icons, cost, disruption warnings
Glassmorphic loading overlay while AI generates
3-dot menu → "Download PDF" (triggers POST to `/api/generate-pdf`)
"Virtual Tour" button → navigates to `/immersive-preview`
`/immersive-preview` — Virtual Tour (`src/app/immersive-preview/page.tsx`)
Leaflet.js map (satellite + dark tile layers, switchable)
Hardcoded demo itinerary for Goa (3 days, 9 stops)
Animated route polyline connecting all stops
Circle markers per stop (color-coded by day)
Left panel: expandable day list, activity cards with vibeRating + distance
"Fly To" animation — smooth pan/zoom to selected location on the map
Mobile-responsive "phone mockup" overlay at bottom
---
6. API Routes
`POST /api/safar` — Streaming AI Chat
Purpose: Handles all `@Safar` conversational messages
Model: `llama-3.3-70b-versatile` (Groq), `temperature: 0.75`, `max_tokens: 512`
Streaming: `stream: true` — pipes Groq's SSE response directly to the browser
System prompt: Safar is a concise, friendly travel planner; never reveals it's an LLM
Error fallback: Returns a graceful message string if Groq is down
`POST /api/safar-itinerary` — Structured Itinerary Generation
Purpose: Generates or replans a full multi-day itinerary as structured JSON
Model: Same as above, `stream: false`, `response_format: { type: "json_object" }`
Temperature: `0.75` for generation, `0.6` for replanning (more deterministic)
Modes: `"generate"` (new itinerary) or `"replan"` (disruption-aware rebuild)
Output schema:
```json
  {
    "chat_reply": "Friendly confirmation message",
    "itinerary_data": [
      {
        "day": "Day 1",
        "title": "Day theme",
        "items": [
          { "time": "10:00 AM", "activity": "...", "cost": "₹1,200", "icon_type": "camera", "warning": null }
        ]
      }
    ]
  }
  ```
Warning field: Populated only when an activity is changed due to a disruption
`POST /api/generate-pdf` — PDF Export
Input: `tripName` + `itinerary` array (from form POST or JSON body)
Library: `jsPDF` (Helvetica, multi-page support)
Output: Binary PDF download via `Content-Disposition: attachment` header
Safety: Strips non-Latin-1 characters from all strings
---
7. Hooks & State Management
`useCurrentUser()` — `src/hooks/useSyncRoute.ts`
Subscribes to `supabase.auth.onAuthStateChange`
Loads user profile from `public.users` table
Exposes: `user`, `profile`, `loading`, `updateProfile`
`useTrips(userId)` — `src/hooks/useSyncRoute.ts`
Fetches all trips the user belongs to (via `trip_members` join)
Provides: `trips`, `addTrip`, `deleteTrip`, `addCollaborator`, `ensureSafarDM`, `refetchTrips`
`ensureSafarDM`: creates the private "Safar AI" workspace trip if it doesn't exist
`useChatMessages(tripId, currentUser, isWorkspace)` — `src/hooks/useSyncRoute.ts`
Fetches message history, subscribes to Realtime `INSERT` events
Handles optimistic message updates (temp ID → real DB ID swap)
`sendMessage`: inserts to DB, then calls `callSafar` if `@Safar` mentioned or in workspace
`callSafar`: streams SSE from `/api/safar`, renders tokens in real-time in a streaming bubble
`useInvitations(userId)` — `src/hooks/useSyncRoute.ts`
Fetches pending invitations with Supabase join (trip title, inviter username)
Realtime subscription on `invitations` table
Provides: `sendInvite`, `acceptInvitation`, `declineInvitation`
`useTyping(tripId, currentUsername)` — `src/hooks/useSyncRoute.ts`
Uses Supabase Broadcast (zero DB writes) to share typing presence
Auto-clears typing status after 2.5s of no keystrokes (debounced)
Auto-expires remote users after 3s (safety timeout)
`useItinerary(tripId)` — `src/hooks/useItinerary.ts`
Loads persisted itinerary from `trips.itinerary_data` (JSONB column)
`generateItinerary(chatContext, userText)`: calls `/api/safar-itinerary`, saves to Supabase
`replanFromAlert(alert)`: rebuilds itinerary using the live alert as context
Auto-fetches live alerts (weather + flight) whenever `itineraryData` updates
Provides: `itineraryData`, `isGeneratingItinerary`, `activeDisruption`, `disruptionReport`, `liveAlerts`, `alertsLoading`
---
8. Services
`liveTravelData.ts` — `src/services/liveTravelData.ts`
`fetchWeather(city: string) → WeatherData`
API: OpenWeatherMap `/data/2.5/weather`
Returns: temp, feels_like, description, humidity, wind_speed, `is_rainy`, `is_stormy`
Fallback: Returns simulated rainy data if API fails (for demo)
`fetchFlightStatus(flightIata?: string) → FlightData`
API: AviationStack `/v1/flights`
Default flight: `6E-2135` (IndiGo, Mumbai → Goa)
Returns: airline, flight number, airports, status, delay_minutes, `is_delayed`
Fallback: Returns non-delayed neutral data if API fails
`checkWeatherDisruption(weather) → DisruptionReport | null`
`checkFlightDisruption(flight) → DisruptionReport | null`
Helper functions used to classify disruption severity (`"high"` / `"medium"`)
`fetchDistance(origin, destination) → DistanceData`
Uses hardcoded Goa distance table (no Google Maps API key required for demo)
---
9. Key Components
`AuthButtons` — `src/components/auth-buttons.tsx`
Renders "Log In" / "Sign Up" buttons in the navbar
Opens an animated `Dialog` modal with 4 views: `login`, `signup`, `forgot`, `sent`
Handles Supabase `signInWithPassword`, `signUp`, `resetPasswordForEmail`
Validates username uniqueness before sign-up
`ModeToggle` — `src/components/mode-toggle.tsx`
Light/Dark/System theme switcher using `next-themes`
`SwipeableMessage` — (inside `dashboard/page.tsx`)
Framer Motion draggable message row
Swipe right > 60px reveals reply icon and triggers reply-to state
`Modal` — (inside `dashboard/page.tsx`)
Reusable animated dialog wrapper (spring animation)
`SafarTag` / `renderWithSafar`
Renders `@Safar` mentions as styled blue badge chips in chat messages
---
10. Database Schema (Supabase)
```sql
-- Core tables inferred from code

public.users
  id            uuid (FK → auth.users)
  username      text UNIQUE
  email         text
  preferences   jsonb  -- { budget, pace, startHour, endHour, dietaryPref,
                       --   transport[], travelStyle, interests[],
                       --   avatar_color }
  created_at    timestamptz

public.trips
  id            uuid PK
  title         text
  vibe          text
  theme_color   text
  created_by    uuid FK → users
  is_workspace  boolean   -- true = private Safar DM
  itinerary_data jsonb    -- ItineraryDayData[] array
  created_at    timestamptz

public.trip_members
  trip_id       uuid FK → trips
  user_id       uuid FK → users
  joined_at     timestamptz
  PRIMARY KEY (trip_id, user_id)

public.messages
  id            uuid PK
  trip_id       uuid FK → trips
  sender_id     uuid FK → users (null = AI)
  content       text
  is_ai         boolean
  created_at    timestamptz

public.invitations
  id            uuid PK
  trip_id       uuid FK → trips
  inviter_id    uuid FK → users
  invitee_id    uuid FK → users
  status        text  -- 'pending' | 'accepted' | 'declined'
  created_at    timestamptz
```
---
11. AI Intelligence Layer
@Safar — The AI Travel Assistant
Identity: Safar is SyncRoute's built-in AI — never reveals it is an LLM
Trigger Conditions:
Any message in the Workspace (private DM) goes to AI
Any message mentioning `@Safar` in a group trip
Any message matching itinerary keywords (e.g. "plan a trip", "create itinerary")
Itinerary Generation Flow
```
User types: "@Safar plan a 3-day trip to Goa"
     │
     ├── isItineraryRequest() → true
     ├── sendMessage() with skipAI=true (prevents duplicate streaming response)
     ├── generateItinerary(chatContext, userText)
     │       → POST /api/safar-itinerary { mode: "generate" }
     │       → Groq LLaMA returns structured JSON
     │       → Saved to trips.itinerary_data in Supabase
     │       → setItineraryData() updates right panel in real-time
     └── AI confirmation message inserted to chat
```
Disruption Replanning Flow
```
Itinerary loaded
     │
     └── Auto-fetch: fetchWeather(destination) + fetchFlightStatus()
               │
               ├── Rain detected → LiveAlert { severity: "danger", canReplan: true }
               ├── Flight delayed → LiveAlert { severity: "danger", canReplan: true }
               └── Clear weather → LiveAlert { severity: "info", canReplan: false }
                         │
                    User clicks "Replan Itinerary"
                         │
                    replanFromAlert(alert)
                         → POST /api/safar-itinerary { mode: "replan" }
                         → System message includes full current itinerary JSON +
                           disruption details + instructions to swap activities
                         → Returns updated itinerary with warning fields populated
                         → Saved to Supabase + displayed in panel with "DISRUPTED" badges
```
@Mention Autocomplete
Typing `@` in the chat input shows a popup with:
`@Safar` — Your AI trip planner
`@update` — Rebuild the itinerary
`@budget` — Optimize trip costs
`@delay` — Handle a disruption
`@optimize` — Fine-tune the plan
---
## 📋 Feature Summary

| Feature | Implementation | Status |
|---|---|---|
| Landing page (marketing) | `page.tsx` | ✅ Complete |
| Dark / Light theme | next-themes + Tailwind | ✅ Complete |
| User sign-up (5-step onboarding) | `onboarding/page.tsx` + Supabase Auth | ✅ Complete |
| User login / forgot password | `auth-buttons.tsx` + Supabase | ✅ Complete |
| Trip creation | `useTrips.addTrip` + Supabase | ✅ Complete |
| Real-time group chat | Supabase Realtime INSERT | ✅ Complete |
| AI chat (@Safar) | Groq SSE streaming | ✅ Complete |
| Typing indicators | Supabase Broadcast (no DB) | ✅ Complete |
| Swipe-to-reply | Framer Motion drag | ✅ Complete |
| Voice input | Web Speech API | ✅ Complete |
| @mention autocomplete | Custom popup with keyboard nav | ✅ Complete |
| Itinerary generation | Groq JSON mode + Supabase | ✅ Complete |
| Itinerary persistence | `trips.itinerary_data` JSONB | ✅ Complete |
| Live weather alerts | OpenWeatherMap API | ✅ Complete |
| Live flight alerts | AviationStack API | ✅ Complete |
| AI disruption replanning | Groq + mode:"replan" | ✅ Complete |
| Collaborative invitations | Supabase invitations table | ✅ Complete |
| PDF export | jsPDF + form POST | ✅ Complete |
| Immersive map preview | Leaflet.js satellite/dark tiles | ✅ Complete |
| User profile / preferences | Supabase users.preferences JSONB | ✅ Complete |
