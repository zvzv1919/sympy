# Catalog Initialization Strategies

## Common Requirements
Target Repository: `sympy/`
Explore target repository, perform these 3 tasks:
1. Write a catalog.md file under target repository root, explaining what this project and each submodule (directory) does
2. Spawn fleets of agent, each go to a sub-dir (ignore non-repo directories like __pycache__), write a catalog.md file under each submodule, requirements are stated below.
3. Modify the root .md file to more accurately list the sub directories and summarize what they do. Also make sure python files directly under root also have their file paths and summaries in the root .md file.

### Note
- Record progress as subagents finish to allow continue from failure. e.g. use a progress.md file which tracks which agent(s) succeeded
- Ignore all test files in the catalog, even if they're python files.

### Structure
- **Top-level sections** group `.py` files by role/functionality (e.g. "Core", "Utilities", "Parsing").
- Each `.py` file gets its own **sub-header** within the appropriate group.
- An optional **Glossary** section at the top and **Appendix** (caveats) section at the bottom may be included.

### Per-file sub-header content
Each `.py` file's section contains:
1. A one-line summary of what the file does.
2. Functions/classes, organized into logical sub-sections with **adaptive verbosity**:
   - Trivial/obvious functions — omit entirely.
   - Simple helpers/utils — list name or declaration only.
   - Moderate complexity — name + short summary.
   - High complexity — bullet point section listing key functionalities.
3. (Optional) A brief "Caveats" note for anything surprising or easy to misuse.

### Root catalog.md
Follows the same format, but each **module** plays the role of a `.py` file: one sub-header per module with a summary and link to its module-level catalog.