# 🧠 Antigravity Developer Skill

> **A template skill for AI coding assistants that teaches cross-module awareness and business logic dependencies.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Compatible](https://img.shields.io/badge/Claude-Compatible-8A2BE2)](https://claude.ai)
[![Antigravity Skill](https://img.shields.io/badge/Antigravity-Skill-orange)](https://github.com/anthropics)

---

## 🎯 What Is This?

This is an **Antigravity Skill** - a knowledge package that enhances AI coding assistants with:

- **Cross-module dependency awareness** - AI understands how changes ripple across your codebase
- **Pre-flight impact checklists** - Validates changes before implementation
- **Code pattern templates** - Generates code matching your project's style
- **Business logic documentation** - References specifications for requirements

## 🚀 Quick Start

### Installation

1. Copy this skill to your project's `.agent/skills/` directory:

```bash
# Clone this repo
git clone https://github.com/garyyap13/AG-developer-skill.git

# Copy to your project
mkdir -p /path/to/your-project/.agent/skills/
cp -r AG-developer-skill /path/to/your-project/.agent/skills/developer-skill/
```

2. The skill automatically activates when relevant.

## 📁 Skill Structure

```
developer-skill/
├── SKILL.md                      # 🧠 Main AI instructions
├── README.md                     # 📖 This file
└── references/
    ├── business-logic-map.md     # 🔗 Cross-module dependencies
    ├── architecture.md           # 🏗️ Project structure
    ├── frontend-patterns.md      # ⚛️ React/TypeScript patterns
    ├── database-patterns.md      # 🗃️ PostgreSQL/Supabase patterns
    ├── types-reference.md        # 📝 TypeScript types
    └── workflow-integration.md   # 🔄 Development workflows
```

## ✨ Key Features

| Feature | What It Does |
|---------|--------------|
| **Business Logic Map** | Documents which modules affect others & what triggers fire |
| **Impact Checklists** | Pre-change validation to avoid breaking related systems |
| **Code Templates** | Patterns for contexts, views, RPCs, and triggers |
| **Workflow Routing** | Guides AI to appropriate development workflow |

## 🔧 Adapting for Your Project

This skill is a **template** - customize it for your codebase:

### Files to Customize

| File | What to Change |
|------|----------------|
| `business-logic-map.md` | Map YOUR cross-module dependencies |
| `architecture.md` | Document YOUR folder structure |
| `types-reference.md` | Add YOUR TypeScript interfaces |
| `frontend-patterns.md` | Update with YOUR component patterns |
| `database-patterns.md` | Add YOUR database conventions |

### Tech Stack

This template is optimized for:
- **Frontend**: React + TypeScript
- **Backend**: Supabase (PostgreSQL)
- **State**: React Context API

Adapt the patterns for your stack!

## 📚 How It Works

When you ask the AI to make a change, it will:

1. **Check dependencies** - Read `business-logic-map.md` to understand impacts
2. **Run pre-flight checks** - Verify which modules/triggers are affected
3. **Follow patterns** - Generate code matching your existing style
4. **Reference specs** - Link to requirements documentation

### Example

> "Add a discount field to orders"

The AI will:
- ✅ Check what modules depend on orders
- ✅ Identify database triggers that will fire
- ✅ Suggest migration + type + context changes
- ✅ Follow your code patterns

## 🤝 Contributing

1. Fork this repository
2. Customize for your use case
3. Share improvements via pull request

## 📄 License

MIT License - Adapt freely for your projects!

---

<div align="center">
  <sub>Built for use with Antigravity's AI skill system</sub>
</div>
