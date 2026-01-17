# Workflow Integration

How to integrate with existing OHCRM workflows when making changes.

## Available Workflows

| Workflow | Slash Command | When to Use |
|----------|---------------|-------------|
| Main Router | `/main-workflow` | General routing based on intent |
| Migrations | `/manage-migrations` | Database schema changes |
| Modules | `/manage-modules` | New frontend module scaffolding |
| Verify | `/verify-system` | Lint, type check, test |
| Deploy | `/deploy-production` | Production deployment |
| OpenSpec | `/openspec-proposal` | Spec-driven feature development |

---

## Database Migrations

Use `/manage-migrations` when:
- Adding new tables
- Adding columns to existing tables
- Creating RPC functions
- Creating or modifying triggers
- Modifying RLS policies

The workflow runs:
```powershell
./scripts/workflow_migrations.ps1 -Mode Agent -Apply yes -GenTypes yes
```

---

## New Modules

Use `/manage-modules` when:
- Creating a new page/view
- Creating associated context
- Setting up new routes

The workflow runs:
```powershell
./scripts/workflow_modules.ps1 -Mode Agent -ModuleName "ModuleName"
```

---

## System Verification

Use `/verify-system` when:
- After making changes to verify correctness
- Debugging issues
- Before deployment

The workflow runs:
```powershell
./scripts/workflow_verify.ps1 -Mode Agent -AutoApprove
```

---

## OpenSpec (Spec-Driven Development)

For significant features, use OpenSpec to:
1. Create a proposal with requirements
2. Get approval before implementation
3. Track implementation progress
4. Archive after deployment

### When to Use OpenSpec

Use when:
- Adding new capabilities
- Making breaking changes
- Changing architecture
- Significant feature additions

Skip when:
- Bug fixes
- Typos/formatting
- Dependency updates
- Configuration changes

### OpenSpec Commands

```bash
# List existing specs
openspec list --specs

# List active changes
openspec list

# Validate a change
openspec validate [change-id] --strict

# Archive after deployment
openspec archive [change-id] --yes
```

### OpenSpec Directory Structure

```
openspec/
├── project.md          # Project conventions
├── specs/              # Current truth (what IS built)
│   └── [capability]/
│       └── spec.md
└── changes/            # Proposals (what SHOULD change)
    └── [change-name]/
        ├── proposal.md
        ├── tasks.md
        ├── design.md   # Optional
        └── specs/      # Delta changes
```

---

## Workflow Decision Tree

```
What are you doing?
│
├── Database change?
│   └── Use /manage-migrations
│
├── New frontend module?
│   └── Use /manage-modules
│
├── Bug fix or small change?
│   └── Direct implementation, then /verify-system
│
├── New feature or capability?
│   └── Use /openspec-proposal first
│
└── Deploying to production?
    └── Use /deploy-production
```

---

## OHCRM-Specific Workflow Notes

### Before Database Changes
Always check `references/business-logic-map.md` for trigger impacts.

### After Any Change
Run verification:
```bash
npm run lint
npm run type-check
```

### New Module Checklist
1. Create migration for any new tables
2. Create context provider
3. Create view component
4. Add route in App.tsx
5. Add to provider hierarchy if needed
6. Update types.ts with new interfaces

---

## Multi-Device Workflow

When working on OHCRM from multiple devices (laptop, desktop, etc.):

### Option 1: Use `/switch-device` Workflow
```bash
# Before leaving current device
/switch-device
```
This auto-commits and pushes all changes.

### Option 2: Manual Git Sync

**Before leaving Device A:**
```bash
git add .
git commit -m "WIP: switching devices"
git push
```

**On Device B:**
```bash
git pull
```

### What Syncs Automatically (via Git)

| What | Location | Syncs? |
|------|----------|--------|
| Skills | `.agent/skills/` | ✅ Yes |
| Workflows | `.agent/workflows/` | ✅ Yes |
| Code | `views/`, `contexts/`, etc. | ✅ Yes |
| Migrations | `supabase/migrations/` | ✅ Yes |

### What Does NOT Sync

| What | Location | Why |
|------|----------|-----|
| Conversation history | `C:\Users\...\antigravity\brain\` | Local to device |
| Node modules | `node_modules/` | Regenerated via `npm install` |

### Setup New Device
```bash
git clone https://github.com/YOUR_USERNAME/OHCRM.git
cd OHCRM
npm install
npx supabase db reset  # Apply migrations locally
```
