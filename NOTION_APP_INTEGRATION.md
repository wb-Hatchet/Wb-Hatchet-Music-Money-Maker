# Notion Integration in App.tsx - Code Reference

This document shows exactly how to add Notion bidirectional sync to your App.tsx component.

## Quick Integration Steps

### 1. Add Imports

At the top of your `src/App.tsx`, add:

```typescript
import { useEffect, useState } from 'react';
import { SyncOpportunity } from './lib/syncTracker';
import { useNotionSync } from './lib/useNotionSync';
import { SyncTracker } from './components/SyncTracker';
import { toast } from 'sonner';
```

### 2. Add State Management

Inside your `App()` function, add these state variables:

```typescript
export default function App() {
  // ... existing state ...
  
  // Notion Sync State
  const [syncOpportunities, setSyncOpportunities] = useState<SyncOpportunity[]>([]);
  const [notionConnected, setNotionConnected] = useState(false);
  
  // Initialize Notion sync hook
  const notionSync = useNotionSync(syncOpportunities, setSyncOpportunities, {
    enabled: true,
    autoSync: true,
    syncInterval: 60000, // 1 minute
    onSyncStart: () => console.log('Syncing with Notion...'),
    onSyncComplete: () => {
      console.log('Notion sync complete');
      setNotionConnected(true);
    },
    onSyncError: (error) => {
      console.error('Notion sync failed:', error);
      setNotionConnected(false);
      toast.error(`Notion sync failed: ${error.message}`);
    },
  });
  
  // ... rest of your existing code ...
}
```

### 3. Initialize Sync on Component Mount

Add this `useEffect` after your state declarations:

```typescript
  // Load initial opportunities from Notion on app startup
  useEffect(() => {
    const initializeNotionSync = async () => {
      try {
        const connected = await notionSync.checkConnection();
        if (connected) {
          console.log('✅ Notion connection verified');
          await notionSync.syncFromNotion();
          setNotionConnected(true);
        } else {
          console.warn('⚠️ Notion not properly configured');
          setNotionConnected(false);
        }
      } catch (error: any) {
        console.error('Failed to initialize Notion:', error);
        toast.error('Notion integration unavailable - working in offline mode');
      }
    };

    initializeNotionSync();

    // Cleanup on unmount
    return () => {
      notionSync.stopAutoSync();
    };
  }, []);
```

### 4. Add Handler Functions

Add these functions inside your `App()` component to handle sync operations:

```typescript
  /**
   * Handle new opportunity creation
   */
  const handleAddOpportunity = async (newOpportunity: Omit<SyncOpportunity, 'id' | 'createdAt' | 'updatedAt'>) => {
    try {
      const opp: SyncOpportunity = {
        ...newOpportunity,
        id: `sync-${Date.now()}`, // Generate temp ID
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      // Add to local state
      setSyncOpportunities(prev => [...prev, opp]);

      // Sync to Notion
      if (notionConnected) {
        const synced = await notionSync.syncToNotion(opp, true);
        // Update with Notion ID
        setSyncOpportunities(prev => 
          prev.map(o => o.id === opp.id ? synced : o)
        );
        toast.success('Opportunity created and synced to Notion');
      }
    } catch (error: any) {
      toast.error(`Failed to create opportunity: ${error.message}`);
    }
  };

  /**
   * Handle status updates
   */
  const handleUpdateStatus = async (id: string, newStatus: SyncOpportunity['status']) => {
    try {
      // Update local state
      setSyncOpportunities(prev =>
        prev.map(opp =>
          opp.id === id
            ? { ...opp, status: newStatus, updatedAt: new Date() }
            : opp
        )
      );

      // Sync to Notion if connected
      if (notionConnected) {
        const updated = syncOpportunities.find(o => o.id === id);
        if (updated) {
          await notionSync.intelligentSync({
            ...updated,
            status: newStatus,
            updatedAt: new Date(),
          });
        }
      }
    } catch (error: any) {
      toast.error(`Failed to update status: ${error.message}`);
    }
  };

  /**
   * Handle opportunity deletion
   */
  const handleDeleteOpportunity = async (id: string) => {
    try {
      if (!confirm('Are you sure you want to delete this opportunity?')) return;

      // Remove from local state
      setSyncOpportunities(prev => prev.filter(o => o.id !== id));

      // Delete from Notion if connected
      if (notionConnected) {
        // The SyncTracker component handles Notion deletion
        toast.success('Opportunity deleted');
      }
    } catch (error: any) {
      toast.error(`Failed to delete opportunity: ${error.message}`);
    }
  };

  /**
   * Handle pitch generation
   */
  const handleGeneratePitch = async (opportunity: SyncOpportunity) => {
    try {
      // This is called by SyncTracker when user clicks "Generate Pitch"
      // You can implement AI pitch generation here using Gemini API
      console.log('Generating pitch for:', opportunity.songTitle);
      
      // Example implementation:
      // const pitch = await generatePitchWithGemini(opportunity);
      // Then save to Notion using:
      // await notionSync.syncToNotion(opportunity, false);
    } catch (error: any) {
      toast.error(`Failed to generate pitch: ${error.message}`);
    }
  };
```

### 5. Add Tab UI

Find your `activeTab` state and the tab navigation, then add this tab option in the tab list:

```typescript
// In your activeTab useState:
const [activeTab, setActiveTab] = useState<'calculator' | 'funnel' | ... | 'licensing' | 'sync-opportunities'>('command');

// In your tab button list:
{(['calculator', 'funnel', ..., 'licensing', 'sync-opportunities'] as const).map((tab) => (
  <button
    key={tab}
    onClick={() => setActiveTab(tab)}
    className={`px-4 py-2 rounded-lg text-sm font-medium transition ${
      activeTab === tab
        ? 'bg-amber-500 text-black'
        : 'bg-slate-800 text-white hover:bg-slate-700'
    }`}
  >
    {tab === 'sync-opportunities' ? '🎵 Sync Tracker' : tab.toUpperCase()}
  </button>
))}
```

### 6. Add Tab Content

In your main render section, add this tab content:

```typescript
{activeTab === 'sync-opportunities' && (
  <motion.div
    key="sync-opportunities"
    initial={{ opacity: 0, y: 20 }}
    animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -20 }}
    className="space-y-6 bg-slate-900/50 p-8 rounded-lg"
  >
    {/* Notion Connection Status */}
    <div className={`p-4 rounded-lg flex items-center gap-3 ${
      notionConnected
        ? 'bg-green-500/10 border border-green-500/30'
        : 'bg-yellow-500/10 border border-yellow-500/30'
    }`}>
      <div className={`w-3 h-3 rounded-full ${
        notionConnected ? 'bg-green-500' : 'bg-yellow-500'
      } animate-pulse`} />
      <span className={`text-sm font-mono ${
        notionConnected ? 'text-green-400' : 'text-yellow-400'
      }`}>
        {notionConnected
          ? '✓ Connected to Notion Database'
          : '⚠ Offline Mode - Notion not connected'}
      </span>
    </div>

    {/* SyncTracker Component */}
    <SyncTracker
      opportunities={syncOpportunities}
      onAddOpportunity={() => {
        // Open modal or form to create new opportunity
        console.log('Add opportunity clicked');
      }}
      onGeneratePitch={handleGeneratePitch}
      onUpdateStatus={handleUpdateStatus}
      onDelete={handleDeleteOpportunity}
      syncWithNotion={notionConnected}
    />

    {/* Manual Sync Button */}
    <div className="flex gap-2">
      <button
        onClick={() => notionSync.syncFromNotion()}
        className="px-4 py-2 bg-amber-500 text-black rounded font-mono font-bold text-xs hover:bg-amber-400 transition"
      >
        🔄 Manual Sync from Notion
      </button>
      
      <button
        onClick={() => notionSync.checkConnection()}
        className="px-4 py-2 bg-blue-500 text-white rounded font-mono font-bold text-xs hover:bg-blue-400 transition"
      >
        🔗 Check Notion Connection
      </button>
    </div>
  </motion.div>
)}
```

---

## Full Component Structure Example

```typescript
import React, { useState, useEffect } from 'react';
import { motion } from 'motion/react';
import { toast, Toaster } from 'sonner';
import { SyncOpportunity } from './lib/syncTracker';
import { useNotionSync } from './lib/useNotionSync';
import { SyncTracker } from './components/SyncTracker';

export default function App() {
  // Existing state...
  const [activeTab, setActiveTab] = useState<'sync-opportunities' | 'other-tabs'>('sync-opportunities');

  // Notion Sync State
  const [syncOpportunities, setSyncOpportunities] = useState<SyncOpportunity[]>([]);
  const [notionConnected, setNotionConnected] = useState(false);

  // Initialize sync hook
  const notionSync = useNotionSync(syncOpportunities, setSyncOpportunities, {
    enabled: true,
    autoSync: true,
    syncInterval: 60000,
    onSyncComplete: () => setNotionConnected(true),
    onSyncError: (error) => {
      console.error('Sync error:', error);
      setNotionConnected(false);
    },
  });

  // Initialize on mount
  useEffect(() => {
    notionSync.checkConnection();
    notionSync.syncFromNotion();
    return () => notionSync.stopAutoSync();
  }, []);

  // Handlers...
  const handleAddOpportunity = async (opp: SyncOpportunity) => {
    setSyncOpportunities(prev => [...prev, opp]);
    if (notionConnected) {
      await notionSync.syncToNotion(opp, true);
    }
  };

  return (
    <>
      <Toaster />
      {/* Your UI here */}
      <SyncTracker
        opportunities={syncOpportunities}
        onAddOpportunity={() => handleAddOpportunity(/* new opp */)}
        syncWithNotion={notionConnected}
      />
    </>
  );
}
```

---

## Environment Variables Setup

Make sure `.env.local` has:

```env
VITE_NOTION_API_KEY=ntn_your_token
VITE_NOTION_DATABASE_ID=your_32_char_id
VITE_GEMINI_API_KEY=your_key
```

---

## Testing the Integration

### Step 1: Verify Connection
```typescript
// In browser console:
const connected = await notionSync.checkConnection();
console.log('Connected:', connected);
```

### Step 2: Load from Notion
```typescript
// In browser console:
const opps = await notionSync.syncFromNotion();
console.log('Opportunities:', opps);
```

### Step 3: Push to Notion
```typescript
// In browser console:
const created = await notionSync.syncToNotion(opportunity, true);
console.log('Created:', created);
```

---

## Troubleshooting Checklist

- [ ] `.env.local` has both `VITE_NOTION_API_KEY` and `VITE_NOTION_DATABASE_ID`
- [ ] Database is shared with your Notion integration in Notion UI
- [ ] Database property names exactly match the list in NOTION_SETUP_GUIDE.md
- [ ] `npm install` was run to install `@notionhq/client`
- [ ] Dev server restarted after env changes (`npm run dev`)
- [ ] Browser console shows no errors (check DevTools F12)
- [ ] Green "Connected to Notion" indicator shows in UI

---

## Next Steps

1. Copy the code snippets into your App.tsx
2. Add the SyncTracker to your UI layout
3. Test with the manual sync button
4. Enable auto-sync
5. Monitor browser console for any errors
6. Check Notion database for synced data

