# Notion Integration - Complete Implementation Guide

## 📋 What You've Got

You now have a complete Notion integration for your Movie Music Maker app with:

✅ **Bidirectional sync** - Changes in your app sync to Notion, and vice versa  
✅ **Auto-sync polling** - Checks Notion every 60 seconds automatically  
✅ **Rich data model** - All opportunity fields stored with proper types  
✅ **Error handling** - Toast notifications for all sync operations  
✅ **Connection health check** - Verify Notion is accessible  
✅ **Manual sync controls** - Trigger sync anytime from UI  

---

## 🎯 Quick Start (5 Minutes)

### Step 1: Notion Setup (2 minutes)
1. Go to https://www.notion.so/my-integrations
2. Create "Movie Music Maker" integration
3. Copy your API key (starts with `ntn_`)
4. Create database with the fields listed below
5. Share database with your integration
6. Copy database ID from URL

### Step 2: Environment Setup (1 minute)
Edit `.env.local`:
```env
VITE_NOTION_API_KEY=ntn_your_key_here
VITE_NOTION_DATABASE_ID=your_database_id_here
```

### Step 3: Install Dependencies (1 minute)
```bash
npm install
```

### Step 4: Restart Dev Server (1 minute)
```bash
npm run dev
```

---

## 📚 Files Created/Modified

### New Files Created:
- **`src/lib/notionService.ts`** - Notion API wrapper (450+ lines)
  - Functions for CRUD operations
  - Data conversion between app and Notion formats
  - Connection health checks
  - Search and filter utilities

- **`src/lib/useNotionSync.ts`** - React hook for sync (300+ lines)
  - Auto-sync polling
  - Intelligent bidirectional sync
  - Sync event callbacks
  - Connection management

- **`.env.local`** - Environment variables template
  - Notion credentials
  - Firebase config (if using)
  - Gemini API key

- **`NOTION_SETUP_GUIDE.md`** - Detailed Notion setup instructions
- **`NOTION_APP_INTEGRATION.md`** - App.tsx integration code snippets

### Modified Files:
- **`src/components/SyncTracker.tsx`** - Added Notion sync handlers
  - Status update with Notion sync
  - Delete with Notion sync
  - Pitch saving to Notion
  - Sync loading states

- **`package.json`** - Added `@notionhq/client` dependency

---

## 🗄️ Notion Database Schema

Create a Notion database with these properties:

| Property | Type | Notes |
|----------|------|-------|
| **Song Title** | Title | Primary field |
| **Project Title** | Text | Opportunity name |
| **Status** | Select | Pitched, Follow-up, Negotiating, Placed, Declined |
| **License Type** | Multi-select | Master, Sync, Master/Sync |
| **Placement** | Select | TV, Film, Ad, Game, Trailer, Other |
| **Company** | Text | Contact company |
| **Supervisor** | Text | Music supervisor name |
| **Fee Offered** | Number | Amount in USD |
| **Submission Date** | Date | When pitched |
| **Deadline** | Date | Opportunity deadline |
| **Notes** | Text | Additional notes |
| **Contract Link** | URL | Link to contract |
| **Backend Royalties** | Number | Expected royalties |
| **Cue Sheet** | Text | Cue sheet reference |
| **Generated Pitch** | Text | AI-generated pitch |
| **Pitch Confidence** | Number | 0-100 score |
| **Last Pitch Generated** | Date | When pitch was generated |

---

## 🚀 How to Use

### In Your UI:

```typescript
<SyncTracker
  opportunities={opportunities}
  onAddOpportunity={handleAddOpportunity}
  onUpdateStatus={handleUpdateStatus}
  onDelete={handleDeleteOpportunity}
  onGeneratePitch={handleGeneratePitch}
  syncWithNotion={true}  // Enable Notion sync
/>
```

### Automatic Features:

1. **Status Changes** - When you change status, it syncs to Notion immediately
2. **Creating Opportunities** - New entries appear in Notion within seconds
3. **Deleting Opportunities** - Deletion removes from both your app and Notion
4. **Auto-polling** - Every 60 seconds, checks Notion for updates
5. **Connection Check** - On app load, verifies Notion is accessible

### Manual Controls:

```typescript
// Get all opportunities from Notion
await notionSync.syncFromNotion();

// Push opportunity to Notion
await notionSync.syncToNotion(opportunity, isNew);

// Check if Notion is accessible
const connected = await notionSync.checkConnection();

// Start/stop auto-sync
notionSync.startAutoSync();
notionSync.stopAutoSync();

// Get sync status
const status = notionSync.getSyncStatus();
// Returns: { isSyncing, lastSyncTime, timeSinceLastSync }
```

---

## 🔗 API Reference

### notionService.ts - Core Functions

```typescript
// Load all opportunities
getOpportunitiesFromNotion(): Promise<SyncOpportunity[]>

// Create new opportunity
addOpportunityToNotion(opportunity): Promise<SyncOpportunity>

// Update existing opportunity
updateNotionOpportunity(pageId, updates): Promise<SyncOpportunity>

// Delete opportunity
deleteNotionOpportunity(pageId): Promise<void>

// Search by text
searchNotionOpportunities(query): Promise<SyncOpportunity[]>

// Filter by status
getNotionOpportunitiesByStatus(status): Promise<SyncOpportunity[]>

// Save AI-generated pitch
savePitchToNotion(pageId, pitchText, confidence): Promise<SyncOpportunity>

// Verify connection
checkNotionConnection(): Promise<boolean>
```

### useNotionSync - Hook Functions

```typescript
const {
  syncFromNotion,      // Pull all from Notion
  syncToNotion,        // Push to Notion
  intelligentSync,     // Smart directional sync
  startAutoSync,       // Enable polling
  stopAutoSync,        // Disable polling
  getSyncStatus,       // Get current status
  checkConnection,     // Verify accessible
} = useNotionSync(opportunities, setOpportunities, options);
```

---

## 🛠️ Configuration Options

### useNotionSync Hook Options:

```typescript
{
  enabled: boolean,           // Default: true
  autoSync: boolean,          // Default: true
  syncInterval: number,       // ms between checks, Default: 60000 (1 min)
  onSyncStart?: () => void,   // Called when sync starts
  onSyncComplete?: () => void, // Called when sync finishes
  onSyncError?: (error) => void, // Called if sync fails
}
```

---

## 🔍 Monitoring & Debugging

### Browser Console:

```javascript
// Check connection
notionSync.checkConnection(); // → Promise<boolean>

// Get sync info
notionSync.getSyncStatus(); 
// → {
//     isSyncing: boolean,
//     lastSyncTime: Date,
//     timeSinceLastSync: number (ms)
//   }

// Manual sync from Notion
notionSync.syncFromNotion(); // → Promise<SyncOpportunity[]>
```

### Environment Check:

```javascript
// In browser console:
console.log('API Key:', import.meta.env.VITE_NOTION_API_KEY?.substring(0, 8) + '...');
console.log('DB ID:', import.meta.env.VITE_NOTION_DATABASE_ID);
```

### Toast Notifications:

Your app will show toast messages for:
- ✅ "Status updated and synced to Notion"
- ✅ "Opportunity created and synced to Notion"
- ✅ "Pitch saved to Notion"
- ❌ "Failed to update status: [reason]"
- ❌ "Sync failed: [reason]"

---

## 🎓 Common Use Cases

### Add a New Opportunity:

```typescript
const handleAddOpportunity = async () => {
  const newOpp: SyncOpportunity = {
    id: `sync-${Date.now()}`,
    songTitle: "My Song",
    projectTitle: "Netflix Series",
    status: "pitched",
    // ... other fields
  };
  
  await notionSync.syncToNotion(newOpp, true); // true = it's new
};
```

### Update Status from Dropdown:

```typescript
const handleStatusChange = async (id: string, newStatus: string) => {
  const opp = opportunities.find(o => o.id === id);
  await notionSync.intelligentSync({
    ...opp,
    status: newStatus,
    updatedAt: new Date(),
  });
};
```

### Sync on App Load:

```typescript
useEffect(() => {
  const init = async () => {
    const connected = await notionSync.checkConnection();
    if (connected) {
      const opps = await notionSync.syncFromNotion();
      setOpportunities(opps);
    }
  };
  init();
}, []);
```

### Handle Pitch Generation:

```typescript
const handleGeneratePitch = async (opportunity: SyncOpportunity) => {
  // Generate pitch with Gemini API
  const pitch = await generatePitchWithGemini(opportunity);
  
  // Save to Notion
  await savePitchToNotion(opportunity.id, pitch.text, pitch.confidence);
};
```

---

## ⚠️ Troubleshooting

### "Notion not configured" Message

**Cause:** Missing or invalid credentials  
**Fix:**
1. Check `.env.local` exists in project root
2. Verify `VITE_NOTION_API_KEY` starts with `ntn_`
3. Verify `VITE_NOTION_DATABASE_ID` is 32 characters
4. Restart dev server: `npm run dev`

### Changes aren't syncing to Notion

**Check:**
1. Is database shared with integration? (Share → Connections)
2. Are property names exact? (Check capitalization)
3. Is API key valid? (Test in browser console)
4. Check browser console for errors (F12)

### "Permission denied" errors

**Cause:** Database not shared with integration  
**Fix:**
1. Open your Notion database
2. Click share button (top right)
3. Find "Connections" section
4. Add your "Movie Music Maker" integration
5. Refresh your app

### Sync only works one direction

**Cause:** May be expected behavior for certain fields  
**Fix:**
- Check `notionService.ts` for field mappings
- Verify property types in Notion match expected values
- Some fields may be read-only by design

### Rate Limit Errors

**Cause:** Notion API limits (3 requests/second on free tier)  
**Fix:**
- Default 60-second sync interval is safe
- Avoid rapid status changes
- Implement exponential backoff for retries

---

## 📊 Data Flow Diagram

```
Your App (React)
    ↓ ↑
Local State (opportunities)
    ↓ ↑
Notion Sync Hook (useNotionSync)
    ↓ ↑
Notion Service (notionService.ts)
    ↓ ↑
Notion API
    ↓ ↑
Notion Database
```

---

## 🔐 Security Notes

- **API Key**: Stored in `.env.local` (never in git)
- **Database ID**: Safe to share, only useful with valid API key
- **Data**: All sync happens via https
- **Auth**: Uses Notion API key for authentication
- **.env.local**: Add to `.gitignore` immediately!

---

## 📝 Next Steps

1. ✅ Create Notion integration and database
2. ✅ Copy credentials to `.env.local`
3. ✅ Run `npm install` (already done)
4. ✅ Start app: `npm run dev`
5. ✅ Test create/update/sync operations
6. ✅ Check browser console for any errors
7. 📈 Add UI for manual sync controls
8. 🚀 Deploy to production

---

## 🎉 You're All Set!

Your app now has:
- ✅ Complete Notion database integration
- ✅ Bidirectional sync
- ✅ Auto-sync polling
- ✅ Error handling with toast notifications
- ✅ Connection health checks
- ✅ React hooks for easy data management

**Happy syncing!** 🚀

---

## 📞 Support Resources

- [Notion API Docs](https://developers.notion.com/)
- [Notion SDK GitHub](https://github.com/makenotion/notion-sdk-js)
- [SQLite Typeface Guide](https://www.notion.so/help/guides/working-with-databases)
- Browser Console (F12) for debugging
- Check `.env.local` if sync fails

