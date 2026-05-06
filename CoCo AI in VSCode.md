---
modified:
  - 2026-05-05T10:23:53+02:00
  - 2026-05-06T10:35:40+02:00
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

## Set up MCP Server in Continue
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