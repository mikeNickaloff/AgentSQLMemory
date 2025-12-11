# AGENT INSTRUCTIONS
- Read this entire document and follow it's process strictly. Details matter here.
- Do not re-invent the wheel.  Compare the descriptions of existing part of the project using ```./wheel.sh``` and compare the descriptions columns with what the user's prompt requirements are to find existing code paths and reusable codepaths instead of fully implementing code.
- Each section of this document is a critical part of the system and cannot be overlooked.
- DO NOT RE-INVENT THE WHEEL!!

# Agentic Data Storage and retrieval 
- provides accurate data retrieval and storage about a project
- decrease overhead by caching lots of data in a database
- replaces the need for massive context windows filled with file contents by using targeted sql statements to get relevant data

## when sqlite3 is available
- will use a local database to store project data 

### WHEEL.db
#### setup
- Use ```./wheel.sh``` to interact with the database.
- The database is in a sqlite3-backed file named `WHEEL.db` with the following tables.

> * `files` (id, relpath, description) // project files with detailed descriptions
> * `defs` (id, file_id, type, signature, parameters, description) // type is property, function, or signal (or any  other public construct accessible from other constructs, no private members). descriptions are paramount here and must be well-documented for every function and signal, and to lesser degree every property
> * `changes` (id, title,  context, status) // will be for change tracking
> * `todo` (id, change_id, def_id, file_id, change_id, description) // todo items to be done, one item for each definition in each definition of each file that is to be added or changed or removed. Must have all fields completed before starting work
>
> [OPTIONAL] `change_files` (id, change_id, file_id) // optional chage file tracking per change (for complex tasks with many changes across multiple files)
> [OPTIONAL] `change_defs` (id, change_id, file_id, def_id) // optional definition tracking per change (for complex tasks with many changes to many definitions)

- chmod +x wheel.sh 

- wheel.sh will verify and import the database from WHEEL.sql if WHEEL.db doesnt exist. 
- if WHEEL.sql is missing, then a blank database with the appropriate file structure will be created. 
- When a blank WHEEL.db is created from the condition where no WHEEL.db and no WHEEL.sql were present, scan the  project directory for source files and add their accessible definitions (functions, methods, exports) to the database with ```./wheel.sh defs``` (see ```./wheel.sh defs --help```)



### sqlite3 database tools for WHEEL.db 

#### wheel.sh
- You can choose to use this shortcut tool which provides access to WHEEL.db through a convenience shell script: 
> ``` wheel.sh ```

- Many common operations are already supported and relevant data can quickly be accessed.

- There are many more use cases for wheel.sh and prefer it over using sqlite3 direct statements or reading file contents into context. 
> ``` ./wheel.sh --help```

- How to query and merge multiple tables (INNER JOIN) -- use the --merge parameter  lke "ON" and multiple --table parameters with space-separated columns
> ```./wheel.sh query  files.relpath defs.signature defs.description --table files --table defs --merge defs.file_id,files.id```

- How to do a keyword search for rows containing all keywords in the same row (over all columns):
> ``` ./wheel.sh search term1 term2 term3 ... termX --table defs```

- How to keyword search and return specific columns but search the entire row:
> ```` ./wheel.sh search term1 term2 ... termX --table files --columns file.relpath,file.description```

- Manipulate todo lists  -- see ```./wheel.sh --help``` for more info
> ``` ./wheel.sh todo [list/add/del/search]``` 

- Also available shortcuts are:
> ```./wheel.sh files [list/add/del/search]```
> ``` ./wheel.sh defs [list/add/del/search] ```
> ``` ./wheel.sh changes [list/add/del/search] ```

- Many shortcuts have additional effort saving implementations such as 
> ```./wheel changes search --file ".cpp" ```  // finds all changes to files containing .cpp in their name

- You can chain together multiple calls to wheel.sh to aggregate information too. For example to see what functions were modified by the latest change record:
> ``` ./wheel.sh changes list  | cut -f1 -d  |  tail -n +3 | egrep "[0-9]+" | \
      tail --lines=1 | xargs -I{} ./wheel.sh query change_defs --where "change_id={}" --columns "def_id" | \
      tail -n +3 | xargs -I%% ./wheel.sh query defs --where "defs.id=%%" ```  

#### Uses

- Use wheel.sh to quickly find functions and their purpose in project files without reading though every single file or doing complex queries.
- Every function/method/routine/member that can be accessed or executed should be added to WHEEL.db via` wheel.sh` with the signature and parameters whenever new code is added.
- Always check with `wheel.sh` output to see what other things have already been invented in this project before creating implementation steps or writing code.
- Always reference `wheel.sh` output when directly when writing code and utilize existing types or helpers when possible instead of creating new ones. Try to use base classes to decrease the amount of overall code paths in the application by reusing existing ones when possible.
- If working with complex chages spanning multiple files, always before making any changes, create a new change entry using `wheel.sh` with specific details. For each definition that will be changed, add a new row into change_defs with the file_id and def_id or by using ./wheel.sh change_defs add --file "relpath" --def "signature" or --def_id 24 --file_id 14
- Once finished with the change table, then use ```./wheel.sh todo add```  command to add specific todo step needing to be changed and create todo items based on that output. This is not always necessary when the all of the changes can be made with minimal effort. 
- When requesting approval for changes, make sure to use output from wheel.sh for providing data. do not rely on contex window memory for reporting, reviewing, or notifying. 
- Do not read  WHEEL.sql into context window. It is automatically generated by ./wheel.sh and contains the latest database changes in text forma so that the database can be distributed in a non-binary format.
- If sqlite3 not available on system, track all of the data listed into a file called WHEEL.md which will contain each file, definitions (and descriptions off them)  but no change tracking will be put into WHEEL.md (to prevent context overload)
- Do not read or consider `WHEEL.sql` into the internal agent context.


# GENERAL CODING SUGGESTIONS
* Break down large problems into multiple simple specfic steps when creating Implementation steps.
* Dont reinvent the wheel or modify existing code unless absolutely necessary.
* Look for a way to implement changes by using existing code first, then if not possible, create new code paths.
* Work slowly and go step-by-step to make compact, requirement fulfilling, working, elegant code.

# Decomposition Engine Specification (DEngine)

## 1. Overview

The Decomposition Engine (DEngine) provides structured functional decomposition,
rule generation, dependency validation, primitive extraction, and system
recomposition capabilities for AGENTOS.

DEngine enables agents to transform any high-level concept, task, subsystem,
algorithm, or architecture into a validated hierarchical structure suitable for
reuse, recombination, and synthesis.

It works alongside the AgentSQLMemory database, the semantic interface exposed
via `wheel.sh`, and the LLM-generation layer to ensure that agents produce
architecturally coherent and dependency-consistent designs.

---

## 2. System Responsibilities

DEngine performs the following duties:

1. **Recursive Functional Decomposition**
   - Break any concept into subsystems, components, and primitives.
   - Maintain hierarchical parent→child relationships.

2. **Rule Generation**
   - Produce domain-appropriate constraints such as:
     - include/build requirements (e.g., Qt `#include <QStateMachine>`)
     - ordering constraints
     - conceptual prerequisites
     - runtime dependencies
     - safety constraints
   - Encode all rules explicitly in AgentSQLMemory.

3. **Rule Validation**
   - Detect contradictions, missing prerequisites, or dependency loops.
   - Identify unresolved or low-confidence rules.

4. **Primitive Extraction**
   - Identify terminal actions, functions, or capabilities that form reusable
     building blocks across domains.

5. **Pattern-Based Recomposition**
   - Rebuild systems using known decomposition trees, primitives, and patterns.
   - Guarantee structural integrity before synthesis.

6. **Persistent Knowledge Storage**
   - Store all nodes, rules, primitives, and patterns in SQLite for long-term
     reuse across projects.

---

## 3. Data Model Specification

### 3.1 Decomposition Units

Represents any concept, subsystem, component, or primitive.

| Field | Type | Description |
|------|------|-------------|
| id | INTEGER | Primary key |
| name | TEXT | Unit name |
| type | TEXT | subsystem/component/primitive/rule |
| description | TEXT | Generated description |
| parent_id | INTEGER | Parent decomposition unit |
| context | TEXT | Domain or environment (Qt, robotics, ML, etc.) |
| confidence | REAL | LLM confidence (0–1) |

---

### 3.2 Dependency Rules

Represents constraints required for a decomposition unit.

| Field | Type | Description |
|-------|--------|-------------|
| id | INTEGER | Primary key |
| unit_id | INTEGER | FK → decomposition_units.id |
| requirement | TEXT | Dependency (e.g., "requires `<QStateMachine>`") |
| rule_type | TEXT | include, build, runtime, ordering, safety, conceptual |
| reason | TEXT | Why this rule exists |
| generated_by | TEXT | Model or agent responsible |

---

### 3.3 Primitive Library

Terminal building blocks used in system design.

| Field | Type |
|--------|-------|
| id | INTEGER |
| name | TEXT |
| description | TEXT |
| input_signature | TEXT |
| output_signature | TEXT |

---

### 3.4 Recomposition Patterns

Reusable construction templates for rebuilding systems.

| Field | Type |
|--------|--------|
| id | INTEGER |
| pattern_name | TEXT |
| description | TEXT |
| structure | TEXT (JSON) |
| domain | TEXT |

---

## 4. Decomposition Algorithm

### 4.1 High-Level Procedure

1. Receive `(concept, context)` from agent.
2. Check if decomposition exists in memory.
3. If not, create a new decomposition root.
4. Request subsystem list from LLM.
5. Insert subsystem units into memory.
6. For each subsystem:
   - Request dependency rules.
   - Validate rules.
   - Store rules.
   - If the subsystem is non-terminal, recursively decompose.
7. Return complete decomposition tree.

---

## 5. Rule Generation Protocol

Whenever a subsystem is created, the following LLM prompt must be issued:

```
Generate all domain-specific dependencies for subsystem X in context Y.
Include include/build requirements, ordering constraints,
conceptual prerequisites, runtime needs, and safety considerations.
Explain why each rule is necessary.
```

Each generated rule must be:

1. Stored in `dependency_rules`.
2. Validated by the Rule Validator.
3. Tagged with `generated_by`.

---

## 6. Dependency Validation Protocol

Validations include:

- **Cycle detection**  
  Ensure no rule introduces circular dependencies.

- **Missing prerequisites**  
  If a rule references a unit not in memory, the engine must:
  - create a placeholder unit
  - force decomposition of that unit

- **Conflict resolution**  
  If the LLM generates contradictory rules, the validator:
  - flags them
  - assigns a low confidence score
  - requests clarification or regeneration

---

## 7. Primitive Extraction Rules

A decomposition unit is considered a primitive if:

- It represents an indivisible action or capability.
- It has no valid subsystem decomposition.
- It has a clear input/output behavioral signature.

All primitives must be registered in `primitives`.

---

## 8. Recomposition Engine

### 8.1 Inputs

- decomposition tree  
- dependency rules  
- primitives  
- recomposition patterns  

### 8.2 Outputs

- validated architecture  
- generated boilerplate  
- dependency graph  
- ordered execution plan  
- domain-specific system template  

### 8.3 Process

1. Load all decomposition units for concept.
2. Collect all dependency rules.
3. Map units to primitives.
4. Match structure against recomposition patterns.
5. Ensure all constraints are satisfied.
6. Synthesize final system.

---

## 9. Integration With wheel.sh

### New Commands

```
wheel decomposition expand <concept>
wheel decomposition tree <concept>
wheel decomposition rules <unit>
wheel decomposition validate <unit>
wheel primitives list
wheel primitives find <pattern>
wheel synthesize <concept>
```

These commands must call the Decomposition Engine and retrieve SQLite-backed data.

---

## 10. Agent Behavior Requirements

Agents MUST:

- Use DEngine for all non-trivial tasks.
- Never output architectural designs without validated decomposition.
- Store every decomposition result in memory for reuse.
- Always retrieve prior decomposition trees before generating new ones.
- Use decomposition trees when writing code, documentation, or designs.
- Ensure rule consistency when generating domain-specific boilerplate.

---

## 11. Output Guarantees

DEngine guarantees:

- Every concept produces a complete decomposition tree.
- All subsystems and rules are explicitly stored.
- Primitive library grows over time.
- Synthesized systems maintain structural integrity.
- Multiple concepts can be recomposed into new hybrids.

---

## 12. Failure Handling

If decomposition yields:

- Missing rules  
- Contradictory rules  
- Circular dependencies  
- Low-confidence primitives  

The engine must regenerate the affected section until:

- A consistent rule set exists  
- All dependencies are satisfied  
- All primitives are defined  

---

## 13. Versioning

Version the decomposition schema using semantic versioning under:

```
/agentos/decomposition/version.json
```

All agents must check compatibility before loading old decomposition trees.

---

# END OF DECOMPOSITION ENGINE SPECIFICATION
