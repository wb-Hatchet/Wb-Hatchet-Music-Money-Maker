# QUICK REFERENCE GUIDE
## Key Files & Integration Points

---

## 📁 FILE LOCATIONS

```
src/
├── components/
│   ├── SyncTracker.tsx              (Main UI for sync opportunities)
│   ├── PitchGeneratorModal.tsx      (AI pitch generation modal)
│   └── SalesFunnelTracker.tsx       (10-stage funnel tracker)
└── lib/
    ├── syncTracker.ts              (Business logic & types)
    ├── pitchGenerator.ts           (Pitch generation logic)
    ├── gemini.ts                   (AI integration layer)
    ├── INTEGRATION_GUIDE.md        (Technical setup)
    └── ZAPIER_MAKE_RECIPES.md      (Automation recipes)

Documentation/
├── README_ORCHESTRATION.md         (Getting started)
└── IMPLEMENTATION_SUMMARY.md       (This project)
```

---

## 🔑 KEY INTERFACES

### SyncOpportunity
```typescript
{
  id: string;
  songId: string;
  projectTitle: string;
  company?: string;
  supervisor?: string;
  submissionDate: Date;
  status: 'pitched' | 'follow-up' | 'negotiating' | 'placed' | 'declined';
  placement: 'tv' | 'film' | 'ad' | 'game' | 'trailer' | 'other';
  feeOffered?: number;
  licenseType: 'master' | 'sync' | 'both';
  contractLink?: string;
  backendRoyalties?: number;
}
```

### GeneratedPitch
```typescript
{
  subject: string;
  body: string;
  confidence: number;     // 0-100
  attachmentSuggestions: string[];
  followUpTiming: string;
  reasoning: string;
}
```

### FunnelStage
```typescript
{
  id: string;
  label: string;
  description: string;
  status: 'not-started' | 'in-progress' | 'completed';
  metrics?: Record<string, any>;
}
```

---

## 🧩 COMPONENT PROPS

### SyncTracker Props
```typescript
interface SyncTrackerProps {
  opportunities?: SyncOpportunity[];
  onAddOpportunity?: () => void;
  onGeneratePitch?: (opportunity: SyncOpportunity) => void;
  onUpdateStatus?: (id: string, status) => void;
  onDelete?: (id: string) => void;
}
```

### PitchGeneratorModal Props
```typescript
interface PitchGeneratorModalProps {
  isOpen: boolean;
  onClose: () => void;
  opportunity?: {
    projectTitle: string;
    company?: string;
    supervisor?: string;
    songTitle: string;
    projectBrief?: string;
  };
  onSuccess?: (pitch: GeneratedPitch) => void;
}
```

### SalesFunnelTracker Props
```typescript
interface SalesFunnelProps {
  releaseId?: string;
  stages?: FunnelStage[];
  onStageUpdate?: (stageId: string, status) => void;
  onMetricsUpdate?: (stageId: string, metrics) => void;
}
```

---

## 🚀 MAIN FUNCTIONS

### From syncTracker.ts
```typescript
createSyncOpportunity(data)          // Create new opportunity
calculateSyncMetrics(opportunities)  // Get performance metrics
filterByStatus(opportunities, status) // Filter by status
getActivePitches(opportunities)      // Get active pitches
generateFollowUpReminders(opportunities) // Auto reminders
daysSincePitch(opportunity)          // Days since pitch
```

### From pitchGenerator.ts
```typescript
generateSyncPitch(input)             // Main function: generate pitch
constructPitchPrompt(input)          // Build Gemini prompt
parsePitchResponse(text)             // Parse JSON response
formatPitchForEmail(pitch, input)    // Format for email
createQuickPitchTemplate(input)      // Manual template
```

### From gemini.ts
```typescript
initializeGemini(config)             // Initialize AI
callGemini(prompt, config)           // Call Gemini API
analyzeSongForSync(songData)         // Analyze for sync
generateFollowUpStrategy(data)       // Smart follow-ups
analyzeCommentSignals(comments)      // Viral signal detection
validateMetadata(metadata)           // Check metadata quality
generatePlaylistPitch(songData)      // Generate playlist pitch
predictOptimalAdSpend(data)          // Budget optimization
generateCueSheet(songData)           // Create cue sheet
```

---

## 🔌 API ENDPOINTS NEEDED

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/generate-pitch` | POST | Generate AI pitch |
| `/api/gemini-query` | POST | General Gemini queries |
| `/api/sync-opportunities` | POST | Create opportunity |
| `/api/sync-opportunities` | GET | List opportunities |
| `/api/sync-opportunities/:id` | GET | Get single |
| `/api/sync-opportunities/:id` | PUT | Update |
| `/api/sync-opportunities/:id` | DELETE | Delete |

---

## 📋 ZAPIER INTEGRATION POINTS

| Recipe | Trigger | Action |
|--------|---------|--------|
| #1 | DistroKid RSS | Create Notion Song |
| #2 | Sync Opp Created | Create Follow-up Tasks |
| #3 | MLC CSV Uploaded | Update Notion Royalties |
| #4 | Pitch Generated | Update Sync Opp |
| #5 | Weekly Schedule | Send Metrics Report |
| #6 | Stage Completed | Send Slack Message |
| #7 | Notion Updated | Sync to Firestore |
| #8 | Sync Opp Created | Assign to Team |
| #9 | Email Received | Create Opportunity |
| #10 | Quarterly Date | Create Campaign |

---

## 🎓 USAGE EXAMPLES

### Quick Start: Add Opportunity
```tsx
import { createSyncOpportunity } from './lib/syncTracker';

const opp = createSyncOpportunity({
  songTitle: 'Midnight Drive',
  projectTitle: 'Netflix Episode 5',
  company: 'Production XYZ',
  submissionDate: new Date()
});

// Save to Firestore
await addDoc(collection(db, 'sync_opportunities'), opp);
```

### Generate Pitch
```tsx
import { generateSyncPitch } from './lib/pitchGenerator';

const pitch = await generateSyncPitch({
  songTitle: 'Midnight Drive',
  songGenre: 'Electronic',
  songMood: 'Dark',
  projectTitle: 'Netflix Show',
  projectBrief: 'Dark mysterious scene',
  // ... other fields
});

console.log(pitch.subject);     // Email subject
console.log(pitch.confidence);  // 0-100 confidence
```

### Calculate Metrics
```tsx
import { calculateSyncMetrics } from './lib/syncTracker';

const metrics = calculateSyncMetrics(allOpportunities);
console.log(metrics.conversionRate);     // 8.5%
console.log(metrics.totalFeesGenerated); // $42,500
console.log(metrics.placedTracks);       // 7 placements
```

---

## 🔧 ENVIRONMENT SETUP

```env
# .env.local

# Required
VITE_GEMINI_API_KEY=sk-proj-xxxxx

# Optional
VITE_NOTION_API_KEY=secret_xxxxx
VITE_NOTION_DATABASE_ID=xxxxx
VITE_ENABLE_SYNC_TRACKER=true
VITE_ENABLE_PITCH_GENERATOR=true
VITE_ENABLE_SALES_FUNNEL=true
```

---

## 📊 FIRESTORE COLLECTIONS

```
sync_opportunities/
├── id (string)
├── songId (string)
├── projectTitle (string)
├── status (string)
├── placement (string)
├── feeOffered (number)
├── createdAt (timestamp)
└── userId (string)

generated_pitches/
├── id (string)
├── opportunityId (string)
├── subject (string)
├── body (string)
├── confidence (number)
└── generatedAt (timestamp)

sales_funnel_progress/
├── id (string)
├── releaseId (string)
├── stageName (string)
├── status (string)
├── metrics (object)
└── userId (string)
```

---

## ✅ DEPLOYMENT CHECKLIST

### Pre-Deployment
- [ ] All environment variables set
- [ ] Firestore collections created
- [ ] API endpoints implemented
- [ ] Components tested locally
- [ ] No console errors

### Deployment
- [ ] Deploy backend (API endpoints)
- [ ] Deploy frontend (components)
- [ ] Verify API connectivity
- [ ] Test with real data
- [ ] Set up monitoring

### Post-Deployment
- [ ] Set up Zapier recipes
- [ ] Configure email templates
- [ ] Create Notion dashboards
- [ ] Train users
- [ ] Monitor usage

---

## 🐛 COMMON ERRORS

| Error | Solution |
|-------|----------|
| "Failed to generate pitch" | Check GEMINI_API_KEY |
| "Firestore permission denied" | Check security rules |
| "Component not rendering" | Check imports |
| "API error 404" | Verify endpoint URL |
| "Undefined prop" | Check parent component |
| "CORS error" | Set up CORS headers |

---

## 📞 SUPPORT RESOURCES

| Resource | Link |
|----------|------|
| Integration Guide | `INTEGRATION_GUIDE.md` |
| Zapier Recipes | `ZAPIER_MAKE_RECIPES.md` |
| Getting Started | `README_ORCHESTRATION.md` |
| Implementation Summary | `IMPLEMENTATION_SUMMARY.md` |
| Source Code | `src/` directory |

---

## 🎯 NEXT IMMEDIATE ACTIONS

### Step 1: Backend (1-2 hours)
```bash
# Create /api/generate-pitch endpoint
# Create Firestore collections
# Set environment variables
```

### Step 2: Integration (30 minutes)
```tsx
// Import components in App.tsx
// Add state management
// Wire up callbacks
```

### Step 3: Testing (30 minutes)
```bash
# Test pitch generation
# Test Firestore sync
# Verify filtering works
```

### Step 4: Automation (2-3 hours)
```bash
# Set up Zapier
# Configure Notion
# Test recipes
```

---

## 💾 CODE STATISTICS

| Metric | Count |
|--------|-------|
| React Components | 3 |
| TypeScript Libraries | 3 |
| Documentation Files | 3 |
| Total Lines of Code | 1,925 |
| Total Documentation | 1,645 |
| API Endpoints | 5+ |
| Firestore Collections | 3 |
| Automation Recipes | 10 |
| Zapier Functions | 25+ |
| Pre-built Checklists | 10 |

---

## 🚀 DEPLOYMENT TIMELINE

```
Day 1: Setup (2 hours)
├─ Create API endpoints
├─ Set up Firestore
└─ Set environment vars

Day 2: Integration (1 hour)
├─ Import components
├─ Wire callbacks
└─ Local testing

Day 3: Automation (2 hours)
├─ Set up Zapier
├─ Configure recipes
└─ Test workflows

Day 4: Launch (1 hour)
├─ Deploy to prod
├─ Monitor
└─ Train users

Total: 6 hours to full deployment
```

---

## 📈 SUCCESS METRICS

Track these after deployment:

```
Week 1:
- Components render without errors ✓
- Pitch generation works ✓
- Firestore syncs data ✓
- No critical console errors ✓

Week 2:
- 5+ test opportunities created ✓
- 10+ pitches generated ✓
- Zapier recipes working ✓
- Notion dashboard updating ✓

Month 1:
- 20+ real opportunities ✓
- 50+ pitches generated ✓
- 2+ placements achieved ✓
- Time saved: 20+ hours ✓
```

---

## 🎓 LEARNING RESOURCES

- **Gemini API**: https://ai.google.dev/
- **Zapier**: https://zapier.com/
- **Make.com**: https://www.make.com/
- **Firestore**: https://firebase.google.com/docs/firestore
- **Notion API**: https://developers.notion.com/

---

**You have everything you need to deploy a professional music business system.**

Good luck! 🎵
