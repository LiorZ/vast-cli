# Claude Code Skills

This repository includes AI agent skills for interacting with the Vast.ai CLI.

## Available Skills

### Vast.ai CLI Skill

**Location**: [.claude/vast-ai-skill.md](.claude/vast-ai-skill.md)

A comprehensive skill that enables AI agents to:

- Search for available GPU instances on the Vast.ai marketplace
- Create, manage, and destroy compute instances
- Handle persistent storage with volumes
- Transfer data between instances and cloud storage
- Manage SSH keys, API keys, and authentication
- Set up autoscaling worker groups and endpoints
- Work with teams and subaccounts
- Host machines on the marketplace (for providers)

## Quick Reference

### Installation
```bash
pip install vastai
```

### Authentication
```bash
vastai set api-key YOUR_API_KEY
```

### Essential Commands
```bash
# Search for GPU instances
vastai search offers 'gpu_ram>=24 reliability>0.95'

# Create an instance
vastai create instance OFFER_ID --image pytorch/pytorch:latest --jupyter --direct

# List your instances
vastai show instances

# Destroy an instance
vastai destroy instance INSTANCE_ID
```

### Search Query Syntax
```
field operator value

Operators: = != > >= < <= in notin
Fields: num_gpus, gpu_name, gpu_ram, cpu_ram, dph_total, reliability, etc.

Example: 'gpu_name=RTX_4090 num_gpus>=2 dph_total<1.0'
```

## Using Skills with Claude

When interacting with Claude or other AI agents, reference the skill documentation for:

1. Complete command syntax and options
2. Available filter fields for searches
3. Common workflows and best practices
4. Error handling guidance

The skill provides structured information that helps AI agents:
- Understand the CLI's capabilities
- Generate correct commands
- Handle edge cases appropriately
- Follow best practices

## Contributing

To add or improve skills:

1. Edit files in the `.claude/` directory
2. Follow the existing format and structure
3. Include practical examples
4. Document error cases and solutions
