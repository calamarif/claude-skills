---
name: prophecy_custom_gem
description: >
  Use this skill whenever the user wants to create a Prophecy custom SQL gem (also called a
  custom gem, macro gem, or plugin gem). Trigger on phrases like "build a gem", "create a custom gem",
  "write a Prophecy gem", "new SQL gem", "gem for [some transformation]", or any request to extend
  Prophecy with a reusable component. Also trigger when the user shares a gem requirement, paste of
  an existing gem, or asks to modify/debug a gem. Do NOT skip this skill just because the request
  seems simple — even one-input gems need the full Python + SQL pair.
---

# Prophecy Custom SQL Gem Builder

This skill guides Claude through producing a complete, working Prophecy custom SQL gem — a matched
pair of a **Python interface file** (`.py`) and a **SQL macro file** (`.sql`). Both files are
required; a gem with only one of them will not work.

---

## Quick orientation

Every Prophecy SQL gem consists of exactly two files:

| File | Purpose |
|------|---------|
| `<GemName>.py` | Defines the gem's UI dialog, validation, and how it encodes its properties into a macro call |
| `<GemName>.sql` | The dbt Jinja2 macro that executes the actual SQL transformation |

The Python file calls the SQL macro via `{{ projectName.GemName(...) }}`. The macro receives
all inputs as strings and must reconstruct them (e.g. parse JSON, normalise lists).

**Common failure modes to avoid:**
- Using a nested custom `@dataclass` as a property type → always use `List[dict]` instead
- Forgetting to add `dependsOnUpstreamSchema: bool = True` when the gem reads input columns
- Inconsistent encoding between `apply()`, `loadProperties()`, and `unloadProperties()`
- Missing dialect variants (`snowflake__`, `duckdb__`) in the SQL macro
- Not normalising `relation_name` from a Python list-string to a Jinja2 list in the macro

---

## Step 1 — Gather requirements

Ask the user for all of the following before writing any code. Do not proceed until you have
every answer.

1. **What SQL transformation** should the gem perform? (brief plain-English description)
2. **Gem name** — must be PascalCase, e.g. `MyTransform`
3. **Project name** — the `projectName` value used throughout the code, e.g. `prophecy_basics`.
   This is the exact string from the Prophecy project settings; prompt the user to confirm it.
4. **Number of input ports** — usually 1; occasionally 2 for join-style gems
5. **User-facing inputs** — what parameters does the gem need, and how should they appear in
   the UI? Examples: column selectors, free-text fields, checkboxes, dropdowns, rule tables.

---

## Step 2 — Study the official examples BEFORE writing any code

Fetch and read the following raw files from the Prophecy examples repo. Do this before
writing a single line of code — the examples are the ground truth for correct patterns.

**Python interface examples:**
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/gems/DataCleansing.py`
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/gems/MultiColumnEdit.py`

**SQL macro examples:**
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/macros/DataCleansing.sql`
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/macros/MultiColumnEdit.sql`

If the user's gem needs a UI pattern not covered by the two examples above, also fetch the
relevant gem from this list:

> CountRecords, DataEncoderDecoder, DataMasking, DynamicSelect, FindDuplicates, FuzzyMatch,
> GenerateRows, JSONParse, MultiColumnRename, RecordID, Regex, Sample, TableOperations,
> TextToColumns, ToDo, Transpose, UnionByName, XMLParse

URL pattern:
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/gems/<Name>.py`
- `https://raw.githubusercontent.com/prophecy-io/prophecy-basics/main/macros/<Name>.sql`

---

## Step 3 — Generate the Python interface file (`<GemName>.py`)

### Canonical class structure

```python
import dataclasses
import json
from dataclasses import dataclass, field
from typing import List

from prophecy.cb.sql.MacroBuilderBase import *
from prophecy.cb.ui.uispec import *


class MyGem(MacroSpec):
    name: str = "MyGem"
    projectName: str = "prophecy_basics"   # use the value confirmed with user
    category: str = "Transform"            # "Transform" | "Prepare" | "Join" | "Aggregate"
    minNumOfInputPorts: int = 1
    supportedProviderTypes: list[ProviderTypeEnum] = [
        ProviderTypeEnum.Databricks,
        ProviderTypeEnum.Snowflake,
        ProviderTypeEnum.BigQuery,
        ProviderTypeEnum.ProphecyManaged,
    ]
    dependsOnUpstreamSchema: bool = True   # required whenever the gem reads column names
```

### Properties dataclass

```python
    @dataclass(frozen=True)
    class MyGemProperties(MacroProperties):
        schema: str = ""                              # always include — stores upstream column info
        relation_name: List[str] = field(default_factory=list)  # always include — upstream node label(s)

        # --- Add your gem-specific properties below ---
        # List[dict]  → table-of-rows (NEVER use a nested @dataclass here)
        # List[str]   → multi-column selector
        # bool        → checkbox
        # str         → text input, select box, or single-column dropdown
        # int / float → number input
```

> **Why `List[dict]` for table rows?** Prophecy's serialisation layer cannot round-trip nested
> frozen dataclasses. Always use `List[dict]` for repeating row structures and bind them to
> `BasicTable`.

### Helper — resolving upstream node names

Always include this helper unchanged. It populates `relation_name`.

```python
    def get_relation_names(self, component: Component, context: SqlContext):
        all_upstream_nodes = []
        for inputPort in component.ports.inputs:
            upstreamNode = None
            for connection in context.graph.connections:
                if connection.targetPort == inputPort.id:
                    upstreamNode = context.graph.nodes.get(connection.source)
            all_upstream_nodes.append(upstreamNode)
        relation_name = []
        for upstream_node in all_upstream_nodes:
            if upstream_node is None or upstream_node.label is None:
                relation_name.append("")
            else:
                relation_name.append(upstream_node.label)
        return relation_name
```

### `dialog()` — building the UI

See the **UI component cheat-sheet** below. Key layout rules:
- Top-level container is always `ColumnsLayout` with `Ports()` as the **leftmost** column
- Group related controls inside `StepContainer > Step > StackLayout`
- Use `SchemaColumnsDropdown` (never `TextBox`) wherever the user picks a column name
- Use `Condition()` to show/hide fields that only apply when a checkbox is ticked
- Use `BasicTable` for any repeating row structure

### `validate()`

```python
    def validate(self, context: SqlContext, component: Component) -> List[Diagnostic]:
        diagnostics = super(MyGem, self).validate(context, component)
        # Add a Diagnostic for each invalid state, e.g.:
        # if not component.properties.myRequiredField:
        #     diagnostics.append(Diagnostic("properties.myRequiredField",
        #                                   "Field is required",
        #                                   SeverityLevelEnum.Error))
        return diagnostics
```

### `onChange()`

Refresh `schema` and `relation_name` whenever the component is edited.

```python
    def onChange(self, context: SqlContext, oldState: Component, newState: Component) -> Component:
        schema = json.loads(str(newState.ports.inputs[0].schema).replace("'", '"'))
        fields_array = [{"name": f["name"], "dataType": f["dataType"]["type"]} for f in schema["fields"]]
        relation_name = self.get_relation_names(newState, context)
        newProperties = dataclasses.replace(
            newState.properties,
            schema=json.dumps(fields_array),
            relation_name=relation_name
        )
        return newState.bindProperties(newProperties)
```

### `apply()` — building the macro call

```python
    def apply(self, props: MyGemProperties) -> str:
        resolved_macro_name = f"{self.projectName}.{self.name}"
        # Encode arguments to match how the SQL macro will receive them:
        #   relation_name  → str(props.relation_name)        arrives as a Python list-string
        #   schema         → props.schema                    arrives as a raw JSON string
        #   str value      → "'" + props.myStr + "'"         wrap in single quotes
        #   bool value     → str(props.myBool).lower()       produces "true"/"false"
        #   List[str]      → str(props.myList)               arrives as a Python list-string
        #   List[dict]     → json.dumps(props.myList)        arrives as a JSON string
        arguments = [str(props.relation_name), props.schema]
        params = ",".join(arguments)
        return f"{{{{ {resolved_macro_name}({params}) }}}}"
```

### `loadProperties()` and `unloadProperties()`

These two methods must be **perfectly symmetric** with each other and with `apply()`.
A mismatch is the most common source of gem bugs — double-check every property.

```python
    def loadProperties(self, properties: MacroProperties) -> PropertiesType:
        parametersMap = self.convertToParameterMap(properties.parameters)
        return MyGem.MyGemProperties(
            relation_name=json.loads(parametersMap.get("relation_name", "[]").replace("'", '"')),
            schema=parametersMap.get("schema", ""),
            # Decode each property using the inverse of how apply() encoded it:
            #   str value   → parametersMap.get("myStr", "")
            #   bool value  → parametersMap.get("myBool", "false") == "true"
            #   List[str]   → json.loads(parametersMap.get("myList", "[]").replace("'", '"'))
            #   List[dict]  → json.loads(parametersMap.get("myList", "[]"))
        )

    def unloadProperties(self, properties: PropertiesType) -> MacroProperties:
        return BasicMacroProperties(
            macroName=self.name,
            projectName=self.projectName,
            parameters=[
                MacroParameter("relation_name", json.dumps(properties.relation_name)),
                MacroParameter("schema", str(properties.schema)),
                # One MacroParameter per property — encoding must match apply():
                #   str value   → MacroParameter("myStr", properties.myStr)
                #   bool value  → MacroParameter("myBool", str(properties.myBool).lower())
                #   List[str]   → MacroParameter("myList", str(properties.myList))
                #   List[dict]  → MacroParameter("myList", json.dumps(properties.myList))
            ],
        )
```

### `updateInputPortSlug()`

Include this unchanged — it keeps column info fresh when the port is reconnected.

```python
    def updateInputPortSlug(self, component: Component, context: SqlContext):
        schema = json.loads(str(component.ports.inputs[0].schema).replace("'", '"'))
        fields_array = [{"name": f["name"], "dataType": f["dataType"]["type"]} for f in schema["fields"]]
        relation_name = self.get_relation_names(component, context)
        newProperties = dataclasses.replace(
            component.properties,
            schema=json.dumps(fields_array),
            relation_name=relation_name
        )
        return component.bindProperties(newProperties)
```

---

### UI component cheat-sheet

| Need | Component |
|------|-----------|
| Select one column from input | `SchemaColumnsDropdown("").bindSchema("component.ports.inputs[0].schema").bindProperty("col")` |
| Select multiple columns | `SchemaColumnsDropdown("", appearance="minimal").withMultipleSelection().bindSchema("component.ports.inputs[0].schema").bindProperty("cols")` |
| Free-text entry | `TextBox("Label", placeholder="hint").bindProperty("myStr")` |
| Number entry | `NumberBox("Label", placeholder="0").withMin(0).bindProperty("myNum")` |
| True/false toggle | `Checkbox("Label").bindProperty("myBool")` |
| Pick one from a fixed list | `SelectBox("").addOption("Display label", "value").bindProperty("mySelect")` |
| Show element conditionally | `Condition().ifEqual(PropExpr("component.properties.myBool"), BooleanExpr(True)).then(<element>)` |
| Table of rows (add/remove) | `BasicTable("Title", columns=[Column("Header", "key", <widget>)]).bindProperty("myList")` |
| Column inside a BasicTable | `Column("Header", "dict_key", SchemaColumnsDropdown("").bindSchema(...))` |
| Horizontal split | `ColumnsLayout(gap="1rem", height="100%").addColumn(..., "5fr")` |
| Vertical stack | `StackLayout(height="100%").addElement(...)` |
| Ports panel (always leftmost) | `Ports()` — place via `.addColumn(Ports(), "content")` |
| Visual grouping | `StepContainer().addElement(Step().addElement(...))` |
| Section title | `TitleElement("Title")` |
| Static descriptive text | `NativeText("...")` |

---

## Step 4 — Generate the SQL macro file (`<GemName>.sql`)

SQL macros use **dbt adapter dispatch** so the gem works across Databricks, Snowflake, and
DuckDB. Always include all three dialect variants.

### Canonical macro structure

```sql
{% macro MyGem(relation_name, schema, my_param='default') -%}
    {{ return(adapter.dispatch('MyGem', 'prophecy_basics')(relation_name, schema, my_param)) }}
{% endmacro %}

{%- macro default__MyGem(relation_name, schema, my_param='default') -%}
    {# Step 1: Normalise relation_name from Python list-string to Jinja2 list #}
    {% set rel_list = relation_name
        if (relation_name is iterable and relation_name is not string)
        else [relation_name] %}

    {# Step 2: Parse schema JSON array → list of {name, dataType} dicts #}
    {% set schema_fields = schema | fromjson %}

    {# Step 3: Build the SELECT column list #}
    {% set cols = [] %}
    {% for field in schema_fields %}
        {% do cols.append(field.name) %}
    {% endfor %}

    select
        {{ cols | join(',\n    ') }}
    from {{ rel_list | join(', ') }}
{%- endmacro -%}

{%- macro snowflake__MyGem(relation_name, schema, my_param='default') -%}
    {# Snowflake variant: use double-quoted identifiers ("col"), :: casting #}
    {% set rel_list = relation_name
        if (relation_name is iterable and relation_name is not string)
        else [relation_name] %}
    {% set schema_fields = schema | fromjson %}
    ...
{%- endmacro -%}

{%- macro duckdb__MyGem(relation_name, schema, my_param='default') -%}
    {# DuckDB variant: use prophecy_basics.quote_identifier() for column names #}
    {% set rel_list = relation_name
        if (relation_name is iterable and relation_name is not string)
        else [relation_name] %}
    {% set schema_fields = schema | fromjson %}
    ...
{%- endmacro -%}
```

### SQL macro rules

- **Always include** `default__`, `snowflake__`, and `duckdb__` variants
- Must be valid **dbt-core Jinja2** — no non-standard filters
- `relation_name` arrives as a Python list-string (e.g. `"['upstream_node']"`) — always
  normalise to a proper list before use
- `schema` arrives as a JSON array of `{name, dataType}` objects — parse with `| fromjson`
- Build column lists with `{% do cols.append(...) %}` then `{{ cols | join(',\n    ') }}`
- Study the fetched examples for the exact quoting and casting idioms per dialect

---

## Step 5 — Output and handoff

1. **Confirm** the final gem name and `projectName` with the user before writing files
2. **Write** `<GemName>.py` 
3. **Write** `<GemName>.sql`
4. **Remind the user** of the testing workflow:
   - In Prophecy, open your project and navigate to the **Gem Builder** UI
   - Paste the `.py` content into the Python editor and the `.sql` content into the SQL editor
   - Save and add the gem to a test pipeline
   - Run the pipeline on a small dataset and verify the output
   - If the gem dialog doesn't render or macro fails, check the browser console / pipeline logs
     and iterate — the most common issues are encoding mismatches in `apply()` /
     `loadProperties()` and missing dialect variants in the SQL macro