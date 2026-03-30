-# Notion Integration Setup Guide

## Quick Start: Connect Your Movie Music Maker App to Notion

This guide walks you through setting up bidirectional sync between your app and Notion database.

---

## Step 1: Create Notion Integration

### 1.1 Go to Notion Developer Portal
- Visit: https://www.notion.so/my-integrations
- Login to your Notion account

### 1.2 Create New Integration
- Click "New integration"
- Name it: **"Movie Music Maker"**
- Select workspace
- Click "Create integration"

### 1.3 Get Your API Key
- Copy the **Internal Integration Token** (displayed on the integration page)
- Add to `.env.local`:
  ```
  VITE_NOTION_API_KEY=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

### 1.4 Grant Database Access
- In Notion, open the page where you want your database
- Click the integrations menu (three dots) → "Connections"
- Find and connect "Movie Music Maker"

---

## Step 2: Create Notion Database

### 2.1 Create Database Template
In Notion, create a new database with these **exact property names**:

| Property Name | Type | Description |
|---|---|---|
| Song Title | Title | Song name (primary field) |
| Project Title | Text | Project/opportunity name |
| Status | Select | Pitched / Follow-up / Negotiating / Placed / Declined |
| License Type | Multi-select | Master / Sync / Master/Sync |
| Placement | Select | TV / Film / Ad / Game / Trailer / Other |
| Company | Text | Company name |
| Supervisor | Text | Project supervisor |
| Fee Offered | Number | Amount offered |
| Submission Date | Date | When you pitched it |
| Deadline | Date | Opportunity deadline |
| Notes | Text | Additional notes |
| Contract Link | URL | Link to contract document |
| Backend Royalties | Number | Expected backend royalties |
| Cue Sheet | Text | Cue sheet URL or reference |
| Generated Pitch | Text | AI-generated pitch text |
| Pitch Confidence | Number | Confidence score (0-100) |
| Last Pitch Generated | Date | When pitch was generated |

### 2.2 Get Database ID
- Open your Notion database
- Copy the URL: `https://www.notion.so/[WORKSPACE-ID]/[DATABASE-ID]?v=[VIEW-ID]`
- Extract the **DATABASE-ID** (32 characters after the first `/`)
- Add to `.env.local`:
  ```
  VITE_NOTION_DATABASE_ID=youhttps://www.notion.so/4d66748555664d64ab1889a921f21f3b?v=8b189838db784e8589a246a82afcc6ca&source=copy_link
  ```

---

## Step 3: Update Environment Variables

Edit `.env.local` in your project root:

```env
# Gemini API Key (required for pitch generation)
VITE_GEMINI_API_KEY=your_gemini_api_key_here

# Notion Configuration
VITE_NOTION_API_KEY=ntn_your_integration_token_here
VITE_NOTION_DATABASE_ID=your_32_character_database_id

# Firebase Configuration (if using)
VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
```

---

## Step 4: Install Dependencies

```bash
npm install
```

This installs `@notionhq/client` and all other required packages.

---

## Step 5: Start Your App

```bash
npm run dev
```

The app will:
1. ✅ Verify Notion connection
2. ✅ Load opportunities from Notion
3. ✅ Enable auto-sync (every 60 seconds by default)

---

## How It Works

### Bidirectional Sync

**Local → Notion:**
- When you update an opportunity status in the app, it syncs to Notion
- New opportunities added in the app are created in Notion
- Deletions in the app remove entries from Notion

**Notion → Local:**
- Every 60 seconds, the app checks Notion for changes
- New opportunities from Notion are pulled into your local state
- Auto-sync can be disabled in the component props

### Sync Indicators

- **Toast Notifications**: Success/error messages appear when syncing
- **Loading State**: Status dropdowns disable during sync operations
- **Last Sync**: Tracked internally (visible in browser console)

---

## Component Integration

### In App.tsx

```typescript
import { useNotionSync } from './lib/useNotionSync';

export default function App() {
  const [opportunities, setOpportunities] = useState<SyncOpportunity[]>([]);

  // Setup Notion sync
  const notionSync = useNotionSync(opportunities, setOpportunities, {
    enabled: true,
    autoSync: true,
    syncInterval: 60000, // 1 minute
  });

  // On component mount, load initial data from Notion
  useEffect(() => {
    notionSync.syncFromNotion();
  }, []);

  // When adding opportunity
  const handleAddOpportunity = async (newOpp: SyncOpportunity) => {
    setOpportunities(prev => [...prev, newOpp]);
    await notionSync.syncToNotion(newOpp, true);
  };

  // When updating status
  const handleUpdateStatus = async (id: string, status: SyncOpportunity['status']) => {
    const updated = opportunities.map(o => 
      o.id === id ? { ...o, status, updatedAt: new Date() } : o
    );
    setOpportunities(updated);
    await notionSync.intelligentSync(updated.find(o => o.id === id)!);
  };

  return (
    <SyncTracker
      opportunities={opportunities}
      onUpdateStatus={handleUpdateStatus}
      syncWithNotion={true}
    />
  );
}
```

### In SyncTracker Component

The component automatically:
- ✅ Syncs status changes to Notion
- ✅ Handles pitch generation and storage
- ✅ Shows sync loading states
- ✅ Displays toast notifications

---

## Troubleshooting

### "Notion not configured" Error

**Solution:**
1. Check `.env.local` has both keys set
2. Verify API key format: should start with `ntn_`
3. Database ID should be 32 characters
4. Restart dev server: `npm run dev`

### Changes not syncing to Notion

**Check:**
1. Is the database shared with your integration? (Share button → Connections)
2. Do property names in database exactly match the list above?
3. Check browser console for error messages
4. Try manual sync button (if implemented)

### Notion API rate limits

- Free tier: 3 requests/second
- Periodic sync waits 60 seconds between checks
- Add retry logic if needed (exponential backoff)

### Property type mismatch errors

**Common issue**: "Select" properties must match exactly
- In Notion: Status → Select with options: Pitched, Follow-up, Negotiating, Placed, Declined
- Check capitalization matches the code

---

## Advanced Configuration

### Custom Sync Intervals

```typescript
const notionSync = useNotionSync(opportunities, setOpportunities, {
  syncInterval: 30000, // 30 seconds
});
```

### Disable Auto-Sync

```typescript
const notionSync = useNotionSync(opportunities, setOpportunities, {
  autoSync: false,
});

// Manual sync when needed
notionSync.syncFromNotion();
```

### Sync Event Callbacks

```typescript
const notionSync = useNotionSync(opportunities, setOpportunities, {
  onSyncStart: () => console.log('Sync started'),
  onSyncComplete: () => console.log('Sync complete'),
  onSyncError: (error) => console.error('Sync failed:', error),
});
```

---

## Testing Your Setup

### 1. Add Opportunity in App
- Create a new sync opportunity
- Should appear in Notion within 1-2 seconds

### 2. Update Status
- Change status from "Pitched" to "Follow-up"
- Notion database should update immediately

### 3. Add Entry in Notion
- Create new row directly in Notion database
- Refresh app (or wait 60 seconds for auto-sync)
- Should appear in your app

### 4. Check Console
- Open browser DevTools (F12)
- Check console for sync debug messages
- Look for any error logs

---

## API Reference

### notionService.ts Functions

```typescript
// Get all opportunities from Notion
getOpportunitiesFromNotion(): Promise<SyncOpportunity[]>

// Add new opportunity to Notion
addOpportunityToNotion(opportunity): Promise<SyncOpportunity>

// Update existing opportunity
updateNotionOpportunity(id, updates): Promise<SyncOpportunity>

// Delete opportunity from Notion
deleteNotionOpportunity(id): Promise<void>

// Search opportunities
searchNotionOpportunities(query): Promise<SyncOpportunity[]>

// Filter by status
getNotionOpportunitiesByStatus(status): Promise<SyncOpportunity[]>

// Save generated pitch
savePitchToNotion(id, pitchText, confidence): Promise<SyncOpportunity>

// Check connection health
checkNotionConnection(): Promise<boolean>
```

### useNotionSync Hook

```typescript
const {
  syncFromNotion,      // Pull all data from Notion
  syncToNotion,        // Push opportunity to Notion
  intelligentSync,     // Smart directional sync
  startAutoSync,       // Start polling
  stopAutoSync,        // Stop polling
  getSyncStatus,       // Get sync info { isSyncing, lastSyncTime, timeSinceLastSync }
  checkConnection,     // Verify Notion is accessible
} = useNotionSync(opportunities, setOpportunities, options);
```

---

## Next Steps

1. ✅ Set up Notion integration and database
2. ✅ Add API credentials to `.env.local`
3. ✅ Run `npm install` to get Notion SDK
4. ✅ Start app with `npm run dev`
5. ✅ Test create/update/sync operations
6. 🚀 Deploy to production

---

## Support

For issues with:
- **Notion API**: See [Notion API Docs](https://developers.notion.com/)
- **This implementation**: Check console for error messages
- **Sync logic**: Enable browser DevTools to monitor network requests

