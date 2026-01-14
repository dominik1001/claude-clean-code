# Claude Clean Code

A Claude Code marketplace with opinionated plugins to write Clean Code

## Available Skills
- *Supabase*: Write clean code using supabase-js

## Available Commands
- `/audit`: Audit your code for potential issues
- `/cleanup`: Analyze and remove dead code

## Installation

```
/plugin marketplace add dominik1001/claude-clean-code
```

### Require for your project/team

```
"extraKnownMarketplaces": {
  "claude-clean-code": {
    "source": {
      "source": "github",
      "repo": "dominik1001/claude-clean-code"
    }
  }
},
"enabledPlugins": {
  "clean-code@claude-clean-code": true
}
```
