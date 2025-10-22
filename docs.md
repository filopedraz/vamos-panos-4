# Idealista Clone - Requirements Document

## Product Overview
A real estate marketplace platform focused on house listings in Barcelona. Users can list houses for sale or rent, search and browse properties with AI-powered recommendations, view detailed property information with image galleries, and contact property owners. The platform is completely free with analytics for property owners.

## Core Functionalities

### 1. Property Listing Management
- **Create Listings**: Users can create house listings with comprehensive details
- **Property Types**: Houses only (sale or rent)
- **Multiple Images**: Upload multiple property images with primary image designation
- **Property Details**:
  - Transaction type (sale/rent)
  - Price (€)
  - Location (Barcelona neighborhoods/districts)
  - Size (m²)
  - Number of rooms (bedrooms, bathrooms)
  - Floor number
  - Build year
  - Property condition (new, renovated, good, needs renovation)
- **Amenities** (comprehensive list):
  - Parking/garage
  - Elevator
  - Terrace/balcony
  - Garden
  - Swimming pool
  - Storage room
  - Furnished/unfurnished
  - Pet-friendly
  - Air conditioning
  - Heating (type)
  - Energy rating (A-G)
  - Accessibility features
  - Security systems
  - Orientation (N/S/E/W)
- **Listing Status**: Active, sold, rented, expired
- **Edit/Delete**: Full CRUD operations for own listings

### 2. Search & Discovery
- **AI-Powered Search**: Natural language queries with intelligent recommendations
- **Advanced Filters**:
  - Transaction type (sale/rent)
  - Price range
  - Number of bedrooms
  - Number of bathrooms
  - Size range (m²)
  - Barcelona neighborhoods/districts
  - Amenities (multiple selection)
  - Property condition
  - Energy rating
  - Floor number
  - Build year range
- **Server-Side Filtering**: All filters applied at database level via tRPC
- **Sort Options**: Price (low/high), date posted, size, relevance
- **Map View**: Interactive map showing property locations
- **List View**: Grid/list toggle for property cards
- **Save Searches**: Persist user search preferences
- **Favorite Properties**: Save properties for later viewing
- **Recently Viewed**: Track user browsing history

### 3. Property Details Page
- **Image Gallery**: Carousel with multiple images, fullscreen view
- **Comprehensive Information**:
  - Title and description
  - Price prominently displayed
  - Key specifications (size, rooms, floor, year)
  - Full amenities list with icons
  - Energy rating badge
  - Property condition indicator
- **Interactive Map**: Exact or approximate location with Google Maps
- **Contact Form**: Inquiry form with user details and message
- **Share Property**: Social sharing and copy link
- **Analytics Display**: View count for the property
- **Similar Properties**: AI-powered recommendations

### 4. User Management
- **Authentication**: Clerk-based auth (email, social login)
- **User Roles**: Single role - all users can list and browse
- **User Profile**:
  - Personal information
  - Contact details
  - Profile image
  - Account settings
- **My Listings Dashboard**:
  - View all own listings
  - Analytics per listing (views, inquiries)
  - Edit/delete listings
  - Quick status updates
- **Saved Properties**: View favorited properties
- **Saved Searches**: Manage search preferences
- **Inquiry Management**: View received inquiries on own listings

### 5. Analytics & Insights
- **Property Owner Analytics**:
  - Total views per listing
  - Daily/weekly view trends
  - Number of inquiries received
  - Time on market
  - Comparison to similar properties
- **Dashboard Widgets**:
  - Overview statistics
  - Performance charts
  - Recent activity feed

### 6. AI-Powered Features
- **Natural Language Search**: Parse user queries like "3 bedroom house with garden near Gracia"
- **Intelligent Recommendations**: Suggest similar properties based on user behavior
- **Smart Filters**: Auto-suggest relevant filters based on search context

## Technical Architecture

### Tech Stack
- **Frontend**: Next.js 15 (App Router), React 19, TypeScript
- **Styling**: Tailwind CSS, Shadcn UI
- **Authentication**: Clerk
- **Database**: PostgreSQL with Drizzle ORM
- **API Layer**: tRPC (type-safe API)
- **Storage**: Vercel Blob (property images)
- **Email**: Resend (inquiry notifications)
- **Maps**: Google Maps API
- **AI/ML**: Python FastAPI engine microservice (search, recommendations)
- **Deployment**: Vercel
- **Monitoring**: Sentry

### Application Structure
```
app/
├── (logged-in)/
│   ├── dashboard/              # User dashboard
│   ├── properties/
│   │   ├── new/               # Create listing
│   │   ├── [id]/edit/         # Edit listing
│   │   └── my-listings/       # Manage own listings
│   ├── favorites/             # Saved properties
│   └── inquiries/             # Received inquiries
├── (logged-out)/
│   ├── properties/
│   │   ├── [id]/             # Property details (public)
│   │   └── search/           # Search results
│   └── page.tsx              # Homepage
└── api/
    ├── trpc/                 # tRPC routes
    └── clerk/                # Clerk webhooks

lib/
├── db/
│   └── schema.ts             # Database schema
├── trpc/
│   ├── routers/
│   │   ├── properties.ts     # Property CRUD
│   │   ├── search.ts         # Search & filters
│   │   ├── inquiries.ts      # Contact management
│   │   └── analytics.ts      # View tracking
│   └── schemas/              # Zod validation
└── engine/
    └── client.ts             # AI service client

engine/
└── src/
    └── api/
        ├── search.py         # NLP search processing
        └── recommendations.py # Property recommendations
```

## Database Schema

### Core Tables

#### `properties`
```typescript
{
  id: serial
  clerkUserId: text (property owner)
  title: text
  description: text
  transactionType: enum('sale', 'rent')
  price: integer (euros)
  pricePerMonth: integer (for rent, nullable)

  // Location
  address: text
  neighborhood: text (Barcelona districts)
  latitude: decimal
  longitude: decimal

  // Specifications
  sizeM2: integer
  bedrooms: integer
  bathrooms: decimal (allows 1.5, 2.5)
  floor: integer (nullable for ground floor)
  buildYear: integer
  propertyCondition: enum('new', 'renovated', 'good', 'needs_renovation')
  energyRating: enum('A', 'B', 'C', 'D', 'E', 'F', 'G', 'not_available')
  orientation: enum('north', 'south', 'east', 'west', 'northeast', 'northwest', 'southeast', 'southwest')

  // Amenities (boolean flags)
  hasParking: boolean
  hasElevator: boolean
  hasTerrace: boolean
  hasGarden: boolean
  hasPool: boolean
  hasStorageRoom: boolean
  isFurnished: boolean
  isPetFriendly: boolean
  hasAirConditioning: boolean
  heatingType: enum('central', 'individual', 'none', 'underfloor')
  hasAccessibility: boolean
  hasSecurity: boolean

  // Status
  status: enum('active', 'sold', 'rented', 'expired')
  viewCount: integer (default 0)

  // Timestamps
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### `propertyImages`
```typescript
{
  id: serial
  propertyId: integer (FK to properties)
  imageUrl: text (Vercel Blob URL)
  isPrimary: boolean
  displayOrder: integer
  createdAt: timestamp
}
```

#### `savedProperties`
```typescript
{
  id: serial
  clerkUserId: text
  propertyId: integer (FK to properties)
  createdAt: timestamp

  // Unique constraint: (clerkUserId, propertyId)
}
```

#### `propertyInquiries`
```typescript
{
  id: serial
  propertyId: integer (FK to properties)
  inquirerUserId: text (Clerk user making inquiry)
  inquirerName: text
  inquirerEmail: text
  inquirerPhone: text (optional)
  message: text
  status: enum('new', 'read', 'replied')
  createdAt: timestamp
}
```

#### `savedSearches`
```typescript
{
  id: serial
  clerkUserId: text
  searchName: text
  filters: jsonb (serialized search criteria)
  notificationsEnabled: boolean
  createdAt: timestamp
  updatedAt: timestamp
}
```

#### `propertyViews`
```typescript
{
  id: serial
  propertyId: integer (FK to properties)
  viewerUserId: text (nullable - track anonymous)
  viewedAt: timestamp

  // For analytics aggregation
}
```

### Indexes
- `properties.clerkUserId` - Owner listings lookup
- `properties.status` - Active listings filter
- `properties.transactionType` - Sale/rent filter
- `properties.neighborhood` - Location search
- `properties.price` - Price range queries
- `properties.bedrooms` - Room count filter
- `properties.createdAt` - Sort by date
- `savedProperties.clerkUserId` - User favorites
- `propertyInquiries.propertyId` - Inquiry lookup
- `propertyViews.propertyId, viewedAt` - Analytics queries

## User Flows

### 1. Create Property Listing Flow
1. User clicks "List Property" (authenticated only)
2. Multi-step form:
   - Step 1: Basic info (transaction type, title, description)
   - Step 2: Location (address, neighborhood with autocomplete)
   - Step 3: Specifications (size, rooms, floor, year, condition)
   - Step 4: Amenities (checkboxes for all features)
   - Step 5: Images (upload multiple, select primary)
   - Step 6: Pricing and review
3. Validation with Zod schemas
4. Submit creates property + images in database
5. Redirect to property detail page
6. Show success message

### 2. Search & Browse Flow
1. Homepage: Featured properties + search bar
2. User enters natural language query OR uses filters
3. AI engine processes query (if NLP search)
4. Server-side filtering via tRPC
5. Results displayed in grid/list with map toggle
6. User can:
   - Apply additional filters (real-time results update)
   - Sort results
   - Toggle map/list view
   - Save search
   - Favorite properties
7. Click property → Detail page

### 3. Property Detail Flow
1. View property images in gallery
2. Read full description and specifications
3. View amenities with visual icons
4. Check location on interactive map
5. See view count and analytics
6. View similar properties (AI recommendations)
7. Actions:
   - Save to favorites (if logged in)
   - Share property
   - Contact owner (opens inquiry form)

### 4. Contact Owner Flow
1. User clicks "Contact Owner" on property detail
2. Form opens (modal or section):
   - Pre-filled user info (if logged in)
   - Message textarea
   - Phone number (optional)
3. Submit inquiry
4. Notification email sent to property owner (Resend)
5. Inquiry saved in database
6. Success message to inquirer
7. Property owner sees inquiry in dashboard

### 5. My Listings Dashboard Flow
1. User navigates to "My Listings"
2. See all own properties with:
   - Thumbnail, title, price
   - Status badge
   - View count
   - Inquiry count
   - Quick actions (edit, delete, change status)
3. Click "View Analytics" → Detailed stats page
4. Click "Edit" → Edit form (pre-filled)
5. Click "Delete" → Confirmation modal → Soft delete

## API Endpoints (tRPC Routers)

### `properties` Router
```typescript
properties.list(filters?) → Property[]
properties.getById(id) → Property | null
properties.create(data) → Property
properties.update(id, data) → Property
properties.delete(id) → void
properties.getMyListings() → Property[]
properties.incrementViewCount(id) → void
properties.toggleStatus(id, status) → Property
```

### `search` Router
```typescript
search.query(naturalLanguageQuery, filters?) → Property[]
search.recommendations(propertyId) → Property[]
search.saveSearch(name, filters) → SavedSearch
search.getSavedSearches() → SavedSearch[]
search.deleteSavedSearch(id) → void
```

### `favorites` Router
```typescript
favorites.add(propertyId) → SavedProperty
favorites.remove(propertyId) → void
favorites.list() → Property[]
favorites.check(propertyId) → boolean
```

### `inquiries` Router
```typescript
inquiries.create(propertyId, data) → PropertyInquiry
inquiries.getReceived() → PropertyInquiry[]
inquiries.markAsRead(id) → PropertyInquiry
inquiries.getForProperty(propertyId) → PropertyInquiry[]
```

### `analytics` Router
```typescript
analytics.getPropertyStats(propertyId) → PropertyStats
analytics.getDashboardStats() → DashboardStats
analytics.getViewTrend(propertyId, period) → ViewTrendData[]
```

## Implementation Notes

### Phase 1: Core Infrastructure (Foundation)
- Database schema setup with Drizzle migrations
- Clerk authentication integration
- tRPC setup with routers structure
- Base UI components (layouts, navigation)
- Homepage with basic property grid

### Phase 2: Property Management
- Create listing form (multi-step)
- Image upload with Vercel Blob
- Property detail page
- Edit/delete functionality
- My Listings dashboard

### Phase 3: Search & Discovery
- Filter sidebar component
- Server-side search implementation
- Google Maps integration
- Saved searches
- Favorites system

### Phase 4: Communication
- Contact form
- Email notifications (Resend)
- Inquiry management dashboard
- Notification preferences

### Phase 5: Analytics
- View tracking system
- Property statistics dashboard
- View trend charts
- Dashboard widgets

### Phase 6: AI Features
- Python FastAPI engine setup
- Natural language search processing
- Property recommendations algorithm
- Search query parser

### Phase 7: Polish & Optimization
- Skeleton loading states
- Error handling
- Performance optimization
- SEO optimization
- Responsive design refinement
- Testing

### Barcelona Neighborhoods Reference
Implementation should include these districts/neighborhoods:
- Ciutat Vella (Gothic Quarter, El Raval, Barceloneta)
- Eixample (Dreta, Esquerra)
- Sants-Montjuïc
- Les Corts
- Sarrià-Sant Gervasi
- Gràcia
- Horta-Guinardó
- Nou Barris
- Sant Andreu
- Sant Martí

### Image Upload Guidelines
- Max file size: 5MB per image
- Supported formats: JPEG, PNG, WebP
- Compress images on upload
- Generate thumbnails for grid views
- Max 20 images per property
- Require at least 3 images for listing

### Google Maps Integration
- Display property location with marker
- Embed interactive map on detail page
- Option for approximate location (privacy)
- Neighborhood boundary visualization
- Nearby amenities (optional: schools, transport)

### AI Search Features (Engine Microservice)
- NLP query parsing: Extract filters from text
- Intent detection: Determine user search intent
- Entity recognition: Identify neighborhoods, amenities
- Query expansion: Suggest related search terms
- Collaborative filtering: User behavior-based recommendations
- Content-based filtering: Similar property features

### Responsive Design Priorities
- Mobile-first approach
- Touch-friendly controls
- Optimized image loading
- Simplified filters on mobile
- Sticky search bar
- Swipeable image galleries

### Performance Considerations
- Image lazy loading
- Infinite scroll for search results
- Debounced search input
- Cached map tiles
- Optimistic UI updates
- Server-side pagination

### Security & Privacy
- Validate all user inputs (Zod schemas)
- Sanitize user-generated content
- Rate limiting on API endpoints
- Image upload virus scanning (optional)
- Email address obfuscation (optional)
- Approximate location option for privacy

---

**Next Steps**: Ready to begin implementation. Should we start with Phase 1 (Core Infrastructure)?
