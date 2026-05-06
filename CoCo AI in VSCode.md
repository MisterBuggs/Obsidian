---
modified:
  - 2026-05-05T10:23:53+02:00
  - 2026-05-06T12:34:15+02:00
created: 2026-05-05T10:09:54+02:00
---
#Science_Computer #Learn #Statistic 
relates to #BayBE , #Sequencing_RNA 

# Continue Plugin 
[CoCo AI :: Documentation for HPC](https://docs.hpc.gwdg.de/services/ai-services/coco/index.html)
## keyboard shortcuts
- ```Ctrl+L```: Open chat with current selection as content.
- ```Ctrl+I```: Enters prompt window to request in-code edits. *Cave*: No progress bar while processing the prompt; wait a moment before assuming an error.
- When encountering a bug / error during compilation / interpretation
	```Ctrl+Shift+R```: Open chat with error as context.
- ```Tab```: Fill in current suggested autocomplete. May need to disable tab spacing, see [[CoCo AI in VSCode#Continue Plugin]]. 
- ```Ctrl+RightArrow```: Like ```Tab``` above but word for word. 

## Arcana
It would be real neat if GWDG enabled models called through SAIA to use [Arcana](https://chat-ai.academiccloud.de/arcanas/) / RAG. 
## Set up MCP Server in Continue (?)
I don't yet fully grasp what I get from MCP servers that I don't get in Continue base functionality. To me, it sounds like I only get customizable Agent mode, which is likely not worth the hassle to set up a local MCP server and translate the settings from the ZED documentation to the continue documentation. 
### json for ZED
from https://docs.hpc.gwdg.de/services/ai-services/coco/index.html
```
{
  "context_servers": {
    "tool-server": {
      "command": {
        "path": "~/dev/tool-server/tool-server",
        "args": [
          "--transport",
          "stdio"
        ],
        "env": null
      },
      "settings": {}
    }
  }
}
```
### Continue config yaml example
from https://docs.continue.dev/reference 
```
name: My Config
version: 1.0.0
schema: v1
mcpServers:
  - name: My MCP Server
    command: uvx
    args:
      - mcp-server-sqlite
      - --db-path
      - ./test.db
    cwd: /Users/NAME/project
    env:
      NODE_ENV: production
```
### Notes
This seems to require me to run a local MCP server.