# Prophecy Claude Skills

A collection of Claude skills that provide context and capabilities for working with [Prophecy](https://prophecy.ai) — the AI-native data transformation and pipeline platform.

These skills are designed to be loaded into Claude to enable accurate, well-positioned responses about Prophecy without needing to re-explain the product each time.

---

## Skills

### 1. `prophecy_business_context` — Business Context & GTM
**File:** `prophecy_business_context/SKILL.md`

Provides Claude with essential background on Prophecy as a company and product. Use this skill for:

- Writing customer-facing content (emails, one-pagers, battle cards, blog posts)
- Comparing Prophecy to competitors (Informatica, dbt, Matillion, ADF, AWS Glue)
- Explaining Prophecy's architecture, features, and terminology
- Preparing for discovery calls or customer conversations
- Crafting GTM and sales messaging

Includes company facts, key differentiators, competitive positioning, target personas, and tone guidance tailored for Sales/GTM users.

---

### 2. `prophecy_custom_gem` — Custom SQL Gem Creator
**File:** `prophecy_custom_gem/SKILL.md`

Enables Claude to build complete, working Prophecy custom SQL gems from a description. Use this skill for:

- Creating new custom gems (single or multi-input)
- Writing the Python wrapper (`__init__.py`) and SQL macro (`.sql`) pair
- Modifying or debugging existing gems
- Understanding gem property types, validation, and code generation patterns

Always produces both required files — Python class and Jinja SQL macro — with correct structure and naming conventions.

---

## Folder Structure

```
prophecy-claude-skills/
├── README.md
├── prophecy-io/
│   └── SKILL.md          # Business context & GTM skill
└── prophecy-gem/
    └── SKILL.md          # Custom SQL gem creator skill
```

---

## How to Use

1. In Claude, open **Settings → Skills**
2. Add a new skill and paste in the contents of the relevant `SKILL.md` file
3. Claude will automatically apply the skill context when relevant prompts are detected

Both skills can be loaded simultaneously — they cover distinct use cases and do not conflict.

---

## Contributing

To suggest updates or improvements, open a pull request with your changes to the relevant `SKILL.md` file. Please include a brief description of what changed and why.