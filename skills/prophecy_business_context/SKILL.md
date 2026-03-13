---
name: prophecy_business_context
description: "Essential background on Prophecy (prophecy.ai / prophecy.io) — the AI-native data transformation and pipeline platform. Use this skill whenever the user asks anything related to Prophecy: explaining the product to customers or prospects, writing docs, blog posts, one-pagers, sales decks or emails about Prophecy, comparing Prophecy to competitors, understanding its architecture, features, or terminology, preparing for customer conversations, or crafting GTM messaging. Trigger even for short or casual questions like \"how does Prophecy handle X?\" or \"how do I position Prophecy vs dbt?\""
---

# Prophecy.io — Company & Product Context
 
The user works in **Sales / GTM** at Prophecy. They frequently ask Claude to help write
customer-facing content, explain the product, compare it to competitors, and craft
messaging. Use this context to give accurate, well-positioned answers without requiring
the user to re-explain what Prophecy is.
 
---
 
## What is Prophecy?
 
Prophecy (prophecy.ai) is an **AI-native data transformation and pipeline platform**
that lets data teams — from business analysts to senior data engineers — build, deploy,
and monitor production-grade data pipelines using a visual, low-code interface that
generates clean, open-source code (Apache Spark or SQL/dbt).
 
**Core value proposition:** The speed and ease of a visual/AI interface *without*
sacrificing code quality, openness, or engineering best practices. Users are never
locked in — all code lives in Git and runs anywhere.
 
**One-line pitch:** "Prophecy makes every data user 10× more productive by combining
AI, visual development, and open-source code in one platform."
 
---
 
## Key Differentiators
 
| Differentiator | What it means |
|---|---|
| **Visual ↔ Code parity** | Every visual pipeline compiles to high-quality Spark or SQL code. Users can switch between canvas and code editor at any time — both stay in sync. No proprietary lock-in. |
| **AI-powered development** | AI generates pipelines, auto-completes transformations, documents code, and recommends fixes when errors occur. |
| **Git-native** | All pipelines are stored as code in Git (GitHub, GitLab, Bitbucket, Azure DevOps). Built-in CI/CD, branching, merging, and code review workflows. |
| **Automated testing** | Auto-generates unit tests for data quality and integration. Tests run in CI/CD before promotion to production. |
| **Observability** | Integrated monitoring and a visual break-fix-redeploy workflow. Errors are highlighted on the canvas with AI-driven resolution suggestions. |
| **Extensibility via Gems/Plugins** | Data engineers write reusable custom components ("Gems") in Spark or SQL. These are shared via Plugin Hub and become available to all users, including business teams, visually. |
| **Multi-persona** | Serves data engineers (Spark/Python), analytics engineers (SQL/dbt), and business data users (no-code data prep) in a single unified platform. |
 
---
 
## Architecture & Technical Concepts
 
### Gems
Gems are the visual building blocks of pipelines — drag-and-drop components representing
sources, targets, and transformations. They come in two flavors:
- **Built-in Gems:** Standard operations (joins, filters, aggregations, etc.)
- **Custom Gems / Plugins:** Written by data engineers in Spark or SQL, versioned on Git,
  and distributed via Plugin Hub for reuse across teams.
 
### Fabrics
A Fabric is a configured connection to a compute environment (e.g., a Databricks workspace,
Snowflake account, or BigQuery project). Users attach pipelines to a Fabric to execute them.
 
### Pipelines
The core unit of work. A pipeline is a DAG (directed acyclic graph) of Gems that reads
data, transforms it, and writes it to a target. Pipelines compile to Spark or SQL code.
 
### Projects
Collections of related pipelines, datasets, and jobs organized in Git repositories.
 
### Jobs / Orchestration
Prophecy supports native orchestration via Databricks Workflows and Apache Airflow. Jobs
can be triggered on schedules, data events, or other conditions.
 
### Supported Execution Engines
- **Apache Spark** (primary): Databricks, EMR, HDInsight, on-prem Spark clusters
- **SQL / dbt Core**: Snowflake, BigQuery, Redshift, Databricks SQL
 
### Deployment Options
- **SaaS**: Managed by Prophecy, connects to customer's Git and cloud data platform
- **On-Premises / Private Cloud**: For enterprises with strict security/data sovereignty
  requirements
 
---
 
## Target Personas & Use Cases
 
| Persona | How they use Prophecy |
|---|---|
| **Data Engineer** | Build and maintain complex Spark pipelines visually; write custom Gems; manage CI/CD and testing |
| **Analytics Engineer / SQL Developer** | Build dbt-style SQL transformations visually; generate clean SQL code; version control models |
| **Business / Data Analyst** | Self-serve data prep without writing code; use business-friendly no-code interface |
| **Data Executive / Platform Lead** | Modernize legacy ETL (Informatica, AbInitio, SSIS); standardize tooling; enforce governance |
 
**Primary enterprise pain points Prophecy solves:**
- Legacy ETL modernization (migrating from Informatica PowerCenter, AbInitio, SSIS)
- Bridging the gap between coders and non-coders on data teams
- Slow, error-prone manual pipeline development
- Lack of software best practices (testing, CI/CD, documentation) in data engineering
- Vendor lock-in from proprietary ETL tools
 
---
 
## Competitive Positioning
 
### Prophecy vs. Informatica / AbInitio / Legacy ETL
- Legacy tools = proprietary, expensive, slow, no Git, no modern cloud-native workflow
- Prophecy = open-source code output, Git-native, AI-assisted, cloud-native, faster to build
 
### Prophecy vs. dbt
- dbt is SQL/code-only, requires technical users, no visual interface, no Spark support
- Prophecy wraps dbt Core with a visual layer and AI, extending it to more users while generating the same dbt SQL code
 
### Prophecy vs. Matillion
- Matillion = proprietary visual tool with its own code format, limited Git-native workflow
- Prophecy = generates 100% open-source Spark/SQL code, true Git-native, better for complex Spark use cases
 
### Prophecy vs. Azure Data Factory / AWS Glue
- Cloud-vendor-specific, lock-in to one cloud, limited collaboration & testing
- Prophecy = cloud-agnostic, multi-cloud, richer development experience with AI
 
### Key win themes in sales:
1. **No lock-in** — open-source code, runs anywhere
2. **All personas on one platform** — replaces multiple point solutions
3. **AI + visual = faster time to value**
4. **Enterprise-grade** — Git, CI/CD, testing, observability baked in (not bolted on)
5. **Legacy modernization path** — transpilers to migrate from Informatica/AbInitio
 
---
 
## Company Facts
 
- **Founded:** ~2019, Palo Alto, CA
- **Founders:** Raj Bains (CEO) and team from Microsoft, NVIDIA, Hortonworks
- **Investors:** Databricks, Insight Partners, SignalFire (Series B)
- **Customers:** Fortune 50 enterprises including HSBC, JP Morgan, Microsoft, Amgen,
  SAP, Ralph Lauren, Deutsche Telekom, Toyota; hundreds of engineers running thousands
  of ETL workloads daily
- **Website:** prophecy.ai (product) / prophecy.io (legacy, redirects)
- **Docs:** docs.prophecy.ai
 
---
 
## Messaging & Tone Guidance (for GTM / Sales content)
 
- Lead with **business outcomes**, not features: faster pipelines, less downtime, more
  self-service, reduced engineering backlog
- Emphasize **openness and no lock-in** — resonates strongly with data leaders burned
  by Informatica
- **"10× more productive"** is a validated claim used in marketing
- Avoid over-emphasizing the visual interface alone — the key insight is that the visual
  *generates real code*, which is what separates Prophecy from toyish no-code tools
- For executive audiences: focus on modernization ROI, governance, and talent leverage
- For technical audiences: Git-native, open-source output, testing, extensibility
 
---
 
## Notes for Claude
 
- The user is in **Sales/GTM**, so assume they want polished, customer-ready language
  unless otherwise specified
- When writing competitive content, be accurate but favor Prophecy's strengths
- The product is called **Prophecy** (not "Prophecy.io") in current branding
- If asked about pricing, note that Prophecy has a free tier and enterprise pricing —
  direct the user to prophecy.ai/pricing or suggest they confirm with their team for
  current details
- The user may need help writing emails, one-pagers, battle cards, blog posts, slide
  narratives, or discovery call prep — tailor output accordingly
