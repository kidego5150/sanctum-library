  Claude Code CLI — Headless & Dangerous Flags                                                                                                                     
                                                                                                                                                                   
  Dangerous / Permission Bypass                                                                                                                                    
                                                                                                                                                                   
  ┌──────────────────────────────────────┬──────────────────────────────────────────────────────────────────────┐                                                        
  │              Flag                    │                             What it does                             │                                                        
  ├──────────────────────────────────────┼──────────────────────────────────────────────────────────────────────┤                                                        
  │ --allow-dangerously-skip-permissions │ Skips all permission prompts. Use only in sandboxed/CI environments. │
  └──────────────────────────────────────┴──────────────────────────────────────────────────────────────────────┘

  Headless (Non-Interactive) Mode                                                                                                                                  
   
  claude -p "your prompt"                          # basic headless                                                                                                
  claude --print "your prompt"                     # same, long form                                                                                               
   
  Output Format                                                                                                                                                    
                  
  claude -p "prompt" --output-format text          # default                                                                                                       
  claude -p "prompt" --output-format json          # structured JSON response
  claude -p "prompt" --output-format stream-json   # streaming JSON (line-delimited)                                                                               
   
  Tool Control                                                                                                                                                     
                  
  --allowedTools "Bash,Read,Write"                 # whitelist specific tools                                                                                      
  --disallowedTools "Bash"                         # blacklist specific tools
                                                                                                                                                                   
  Full Headless Agent Example
                                                                                                                                                                   
  claude -p "list all .ts files in src/" \
    --output-format json \                                                                                                                                         
    --allowedTools "Bash,Glob,Read" \
    --dangerously-skip-permissions \                                                                                                                               
    --max-turns 10                                                                                                                                                 
   
  JSON Output Shape (--output-format json)                                                                                                                         
                  
  {
    "type": "result",
    "subtype": "success",
    "result": "...",                                                                                                                                               
    "session_id": "...",
    "total_cost_usd": 0.001,                                                                                                                                       
    "num_turns": 1                                                                                                                                                 
  }
                                                                                                                                                                   
  stream-json Shape (one object per line)                                                                                                                          
   
  Emits assistant messages, tool calls, and a final result object — suitable for | jq piping.                                                                      
                  
  Other Useful Flags                                                                                                                                               
                  
  ┌─────────────────────────┬──────────────────────────┐                                                                                                           
  │          Flag           │         Purpose          │
  ├─────────────────────────┼──────────────────────────┤
  │ --tmux="classic"        │ Opens tmux session**     │
  ├─────────────────────────┼──────────────────────────┤ 
  │ **req.   -w <name>      │ Worktree mode            │
  ├─────────────────────────┼──────────────────────────┤ 
  │ --bare                  │ Bare naked mode          │
  ├─────────────────────────┼──────────────────────────┤ 
  │ --model claude-opus-4-7 │ Override model           │
  ├─────────────────────────┼──────────────────────────┤
  │ --system-prompt "..."   │ Custom system prompt     │
  ├─────────────────────────┼──────────────────────────┤                                                                                                           
  │ --add-dir /path         │ Add directory to context │
  ├─────────────────────────┼──────────────────────────┤                                                                                                           
  │ --max-turns N           │ Cap agent turns          │
  ├─────────────────────────┼──────────────────────────┤
  │ --no-interactive        │ Alias for headless mode  │
  └─────────────────────────┴──────────────────────────┘                                                                                                           
   
  ---                                                                                                                                                              
  The canonical pattern for a JSON-returning CLI agent is -p "prompt" --output-format json --dangerously-skip-permissions with an explicit --allowedTools
  whitelist.                                          