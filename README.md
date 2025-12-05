# **A Language-Agnostic Code Intelligence Memory System**

*wheel.sh* is a lightweight, persistent code-intelligence layer designed to extend the working memory of AI coding assistants by storing structured metadata, references, and descriptions in a local SQLite database.  
This enables AI models to recall details about any codebase instantly, without repeatedly scanning the source tree or exceeding context window limits.

*wheel.sh* is fully language-agnostic and works even with user-defined DSLs and novel syntaxes (e.g., q-html, q-script). It achieves this by recording information *as code is generated*, leveraging the fact that the AI has maximum understanding at creation time.

---

# **Key Features**

### **✓ Language-Agnostic**
Works with:
- C, C++, JavaScript, Python  
- QML  
- HTML/CSS/XML  
- Domain-specific languages  
- Custom syntaxes and experimental DSLs  
- Mixed-language projects  

If it’s text, *wheel.sh* can store metadata about it.

---

### **✓ Persistent Code Memory**
*wheel.sh* stores:
- function descriptions  
- symbol definitions  
- references and usages  
- alias relationships  
- high-level summaries  
- component relationships  
- behavior and purpose descriptions  

This ensures the AI never needs to re-read entire files to find important logic.

---

### **✓ Extremely Fast Search**
```
./wheel.sh search heal HP
```

Results are instant because they query a **local SQLite database**, not an LLM.

---

### **✓ Zero Maintenance**
- No parser rules  
- No AST requirements  
- No language-specific indexing  
- No grammar maintenance  
- No re-index passes  
- No upgrades needed when DSLs evolve  

*wheel.sh* grows as the AI writes code, not as you patch the tool.

---

# **Why *wheel.sh* Exists**

AI code assistants struggle with:

- limited context windows  
- expensive project-wide scanning  
- CPU timeouts  
- difficulty locating renamed or indirectly-referenced functions  
- handling custom or experimental languages  
- ambiguous naming  
- multi-file, multi-language architectures  

*wheel.sh* eliminates these issues by storing structured metadata and references as the AI generates code. This allows future queries to be answered instantly without full project re-analysis.

---

# **How It Works (Architecture Overview)**

1. **AI generates or edits code.**
2. The AI also emits:
   - symbol definitions  
   - reference entries  
   - brief summaries  
   - relationships  
   - alias mappings  
3. *wheel.sh* inserts these into a local SQLite database:
4. AI agents query *wheel.sh* instead of re-reading code, eliminating context overflow and wasted compute.

---

# **Requirements**

- **Linux**, **macOS**, or **WSL**  
  - Fully supported  
- **Native Windows**  
  - Currently partially supported  
  - Works best under Git Bash or WSL  
  - Minor path handling improvements needed for full Windows-native compatibility  
- `bash`
- `sqlite3`
- Any AI Coding Agent that can run commands, understand the results, and read AGENTS.md 

**No compilers, SDKs, or language servers required.**

---

# **Installation**

Clone the repository:

```bash
git clone https://github.com/mikeNickaloff/AgentSQLMemory
cd AgentSQLMemory
chmod +x *wheel.sh
```

Copy wheel.sh and Agents.MD to any project:
```bash
cp * /path/to/my_project/
```

Run Agentic Coding Tool (Like OpenAI codex or claude) from the project directory
```bash
codex
```

Prompt the Agent to `read Agents.MD and scan the project's files for entry into the database.`

After some time scanning (only takes a long time on initial scan, afterwards no scanning required)...

Prompt agent to make changes to your code as you normally would and witness the magic.. the AI finally can code right!


---

# **Basic Commands (Examples)**

- *wheel.sh* is for AI-powered agents to use, not humans.
- Humans can use, but it is just an implementation of the existing human system called "memory" which most humans are equipped with out of the box already. 

# **Design Philosophy**

*wheel.sh* is built on one principle:

> *“Let the AI store what it knows when it knows it — not after.”*

Static analysis tools must understand every language.  
*wheel.sh* does not.  
It relies on the model to create metadata incrementally, using SQLite as permanent memory.

This removes the need for:

- parsers  
- ASTs  
- compilation  
- per-language plugins  
- fragile static analysis logic  

*wheel.sh* works with **any** language that exists today or might be invented tomorrow.

---

# **Roadmap**

- Windows-native improvements  
- Graph visualization of symbol relationships  
- API for multi-agent integration  
- Dependency and lifecycle mapping  
- Project-wide summarization tools  

---

# **License**

TBD — pending evaluation of open-source or source-available licensing.

---

# **Contributions**

Contributions, suggestions, and extensions are welcome once the repository is public.  
For now, *wheel.sh* is under active development and used experimentally in AI-assisted workflows.


---

# ORIGINAL README.md


# AGENTIC-CODING-FRAMEWORK
The latest in Agent-powered AI coding AGENTS.md evolution. This AGENTS.md and *wheel.sh* pair is enough to empower Agentic large language coding models to effectively utilize, build, and undersand highly compelx code bases with many files and defintions that far exceed the agent's context window by leveraging a sqlite3 database to offload much of the work that agents would normally do into quick, targetted shortcuts. 

- Decreases the overall time to process a user prompt by significant amount. Also decreases overall CPU requirements, and increases the accuracy, and adds the ability to tightly integrate to any agentic coding model that supports AGENTS.md.

## Testing environment
- A window with a component that can be used to instantiate an animated red square which will automatically move to a random place when a instance is created (similar to a class in other languages)
  
### Main.qml 
```
import QtQuick
Window {
    width: 800
    height: 800
    title: "hello world"

   Component {
      id: squareComponent
      Rectangle {
        id: squareRoot
        function moveTo(x, y) {
           squareRoot.x = x
           squareRoot.y = y;
        }
        Behavior on x { NumberAnimation { duration: 300 } }
        Behavior on y { NumberAnimation { duration: 300 } }
        Component.onCompleted: {
              var maxX = 800
              var maxY = 800
              var rx = Math.random() * maxX
              var ry = Math.random() * maxY
              moveTo(rx, ry)
        }
     }
   }

  Item {
      id: root
      anchors.fill: parent
  }

}
```



## What does this fix?
- Agentic code generators tend to not take into consideration existing code bases and opt to reinvent the *wheel.sh* when given the option.


### prompt
``` Create 10 red squares that have the same dimensions and each one starts at (0,0) and  moves to a different position when created ```

  ### Output
       Rectangle {
            id: square1
            x: 0
            y: 0
            width: 50
            height: 50
            color: "red"

            NumberAnimation on x { from: 0; to: 300; duration: 300000; running: true }
            NumberAnimation on y { from: 0; to: 100; duration: 300000; running: true }
       }
         Rectangle {
            id: square2
            x: 0
            y: 0
            width: 50
            height: 50
            color: "red"

            NumberAnimation on x { from: 0; to: 300; duration: 300000; running: true }
            NumberAnimation on y { from: 0; to: 300; duration: 300000; running: true }
        }
      ...

## What this file does when added to projects
- Grants code generating agents the ability to reference and use other parts of the project quickly and effectively without additional prompting.

### Prompt
``` Create 10 red squares that have the same dimensions and each one starts at (0,0) and  moves to a different position when created ```

### output
```
 Repeater {
    model: 10
    delegate: squareComponent 
}
```
