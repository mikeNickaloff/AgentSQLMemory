In order to get AgentSQLMemory working  for a brand new project, copy all files from this repo into an empty directory and then run the agent (for example codex) on the command line interface or via an agentic implementation that can execute shell scripts

If you want to use this system on an existing project, copy all files into the root of the project and then run `codex`. 
The first prompt should be something like `scan my project and setup the database according to AGENTS.md`  unless you want it to create purely brand new functions with no integration with existing codebase.

If you don't scan the project, it will not be able to integrate as cleanly with your existing code and will reinvent the wheel more as it wont have any knowledge of the existing code and what it does. 
However, it will still track any new code it adds, and it will also update the database whenever it reads a file, so eventually it will populate part of the database, but will still not be as well-integrated. 

* Note * Large code bases will take a while to fully scan, and may even require multiple prompts / follow-up steps which the system will guide you through.  The good news is, even if you use up 100% of your context window, you can just start a new session with a blank context and it will be able to access all the memory from the last session.

* Note * Once database is scanned, writing code will happen in less than 2 minutes even for extremely complex integrations, this system handles them with ease in just a few minutes and uses almost no CPU time since its offloading the context into a local DB.
