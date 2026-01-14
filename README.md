# Claude Plugins

A collection of plugins for Claude Code.

## Installation

Add the marketplace to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "claude-plugins": {
      "source": {
        "source": "github",
        "repo": "claude-contrib/claude-plugins"
      }
    }
  }
}
```

Then install plugins:

```text
/plugin install <plugin-name>@claude-plugins
```

## Available Plugins

| Plugin                                               | Description                                                  |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| [`agents-context`](plugins/agents-context/README.md) | Enable [AGENTS.md](https://agents.md/) rules for Claude Code |

## License

MIT
