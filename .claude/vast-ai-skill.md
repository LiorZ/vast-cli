# Vast.ai CLI Skill for AI Agents

## Overview

This skill enables AI agents to interact with the Vast.ai GPU cloud marketplace platform through its command-line interface. Vast.ai provides access to affordable GPU compute resources for machine learning, AI training, and inference workloads.

## Installation

### Option 1: Install via pip (Recommended)
```bash
pip install vastai
```

### Option 2: Direct download
```bash
wget https://raw.githubusercontent.com/vast-ai/vast-python/master/vast.py
chmod +x vast.py
```

### Option 3: Using Poetry
```bash
curl -sSL https://install.python-poetry.org | python3 -
poetry install
```

## Authentication

Before using the CLI, you must set your API key:

```bash
# Set API key (obtain from https://vast.ai/console/cli/)
vastai set api-key YOUR_API_KEY_HERE
```

The API key is stored in `~/.config/vastai/vast_api_key` (or `~/.vast_api_key` for legacy).

You can also set the API key via environment variable:
```bash
export VAST_API_KEY=your_api_key_here
```

## Global Options

All commands support these global options:
- `--url URL` - Server REST API URL (default: https://console.vast.ai)
- `--retry N` - Retry limit for API calls (default: 3)
- `--explain` - Output verbose explanation of CLI to API mapping
- `--raw` - Output machine-readable JSON
- `--api-key KEY` - Use specific API key instead of stored one

## Core Commands

### Searching for GPU Instances

#### Search for available offers
```bash
# Basic search
vastai search offers

# Search with filters
vastai search offers 'gpu_ram >= 24 num_gpus = 1 reliability > 0.95'

# Search with ordering (ascending with +, descending with -)
vastai search offers 'gpu_name = RTX_4090' -o 'dph_total+'

# Limit results
vastai search offers --limit 10
```

#### Search Query Syntax

Filters use the format: `field operator value`

**Operators:**
- `=` or `==` - equals
- `!=` - not equals  
- `>`, `>=`, `<`, `<=` - comparisons
- `in` - value in list
- `notin` or `nin` - value not in list

**Common Filter Fields:**
| Field | Description |
|-------|-------------|
| `num_gpus` | Number of GPUs |
| `gpu_name` | GPU model name (use underscores for spaces, e.g., `RTX_4090`) |
| `gpu_ram` | GPU memory in GB |
| `cpu_ram` | System RAM in GB |
| `cpu_cores` | Number of CPU cores |
| `disk_space` | Available disk space in GB |
| `dph_total` | Total cost per hour in USD |
| `reliability` | Machine reliability score (0-1) |
| `inet_up` | Upload speed in Mbps |
| `inet_down` | Download speed in Mbps |
| `cuda_max_good` | CUDA version |
| `driver_version` | NVIDIA driver version |
| `geolocation` | Country code |
| `verification` | Verification status |
| `direct_port_count` | Number of direct ports available |
| `pcie_bw` | PCIe bandwidth |
| `dlperf` | Deep learning performance score |

**Example Searches:**
```bash
# Find RTX 4090 with at least 24GB VRAM, high reliability
vastai search offers 'gpu_name=RTX_4090 gpu_ram>=24 reliability>0.95'

# Find multi-GPU instances under $2/hour
vastai search offers 'num_gpus>=4 dph_total<2.0'

# Find instances in specific countries
vastai search offers 'geolocation in [US,CA,DE]'

# Search for interruptible (spot) instances
vastai search offers --type bid
```

### Instance Management

#### Create an instance
```bash
# Create from an offer ID (obtained from search)
vastai create instance OFFER_ID --image pytorch/pytorch:latest --disk 50

# Create with Jupyter notebook
vastai create instance OFFER_ID --image pytorch/pytorch:latest --jupyter --direct

# Create with SSH access
vastai create instance OFFER_ID --image ubuntu:22.04 --ssh --direct

# Create with environment variables and port mappings
vastai create instance OFFER_ID --image myimage --env '-e MY_VAR=value -p 8080:8080'

# Create with onstart script
vastai create instance OFFER_ID --image myimage --onstart-cmd 'pip install -r requirements.txt && python train.py'

# Create interruptible (spot) instance with bid price
vastai create instance OFFER_ID --image myimage --bid_price 0.50
```

#### Launch from template
```bash
vastai launch instance OFFER_ID --template_hash HASH
```

#### View instances
```bash
# List all your instances
vastai show instances

# Show specific instance details
vastai show instance INSTANCE_ID

# Get instance SSH URL
vastai ssh-url INSTANCE_ID
```

#### Control instances
```bash
# Start a stopped instance
vastai start instance INSTANCE_ID

# Stop a running instance
vastai stop instance INSTANCE_ID

# Reboot an instance
vastai reboot instance INSTANCE_ID

# Destroy an instance permanently
vastai destroy instance INSTANCE_ID

# Start/stop multiple instances
vastai start instances ID1 ID2 ID3
vastai stop instances ID1 ID2 ID3
vastai destroy instances ID1 ID2 ID3
```

#### Label instances
```bash
vastai label instance INSTANCE_ID 'my-training-job'
```

#### Update instance
```bash
vastai update instance INSTANCE_ID --min_bid 0.30
```

### Data Transfer

#### Copy files to/from instances
```bash
# Copy local file to instance
vastai copy ./local_file.txt INSTANCE_ID:/workspace/

# Copy from instance to local
vastai copy INSTANCE_ID:/workspace/results.tar.gz ./

# Copy between instances
vastai copy SOURCE_INSTANCE:/path DEST_INSTANCE:/path

# Use specific SSH key
vastai copy ./data INSTANCE_ID:/workspace/ -i ~/.ssh/my_key
```

#### Cloud sync (with cloud storage providers)
```bash
# Sync to cloud storage
vastai cloud copy --src /workspace/data --dst s3://mybucket/data \
  --instance INSTANCE_ID --connection CONNECTION_ID --transfer "Instance to Cloud"

# Sync from cloud storage  
vastai cloud copy --src s3://mybucket/data --dst /workspace/data \
  --instance INSTANCE_ID --connection CONNECTION_ID --transfer "Cloud to Instance"
```

#### Cancel ongoing transfers
```bash
vastai cancel copy INSTANCE_ID
vastai cancel sync INSTANCE_ID
```

### Logs and Monitoring

#### View instance logs
```bash
vastai logs INSTANCE_ID

# Tail logs
vastai logs INSTANCE_ID --tail

# Get specific log type
vastai logs INSTANCE_ID --type postrun
```

#### Execute commands on instance
```bash
vastai execute INSTANCE_ID 'nvidia-smi'
vastai execute INSTANCE_ID 'python train.py'
```

### Volumes (Persistent Storage)

#### Search for volume offers
```bash
vastai search volumes
vastai search volumes 'disk_space >= 100 reliability > 0.95'
```

#### Create and manage volumes
```bash
# Create a volume
vastai create volume OFFER_ID --size 100 --name my-data-volume

# Show your volumes
vastai show volumes

# Link volume to instance during creation
vastai create instance OFFER_ID --image myimage --link-volume VOLUME_ID --mount-path /data

# Delete volume
vastai delete volume VOLUME_ID
```

### Network Volumes

```bash
# Search network volume offers
vastai search network-volumes

# Create network volume
vastai create network volume OFFER_ID --size 500 --name shared-storage

# Show network disks
vastai show network-disks
```

### Billing and Account

#### View account info
```bash
vastai show user
```

#### View invoices
```bash
vastai show invoices

# Search invoices with date range
vastai search invoices --start_date "01/01/2024" --end_date "12/31/2024"
```

#### Add credit
```bash
vastai show deposit
```

#### Transfer credit
```bash
vastai transfer credit --amount 10.00 --recipient user@email.com
```

### SSH Keys

```bash
# Create/add SSH key
vastai create ssh-key "$(cat ~/.ssh/id_rsa.pub)"

# Or generate new key pair
vastai create ssh-key -y

# List SSH keys
vastai show ssh-keys

# Delete SSH key
vastai delete ssh-key KEY_ID

# Attach SSH key to running instance
vastai attach ssh INSTANCE_ID "ssh-rsa AAAA..."

# Detach SSH key from instance
vastai detach ssh INSTANCE_ID KEY_ID
```

### API Keys

```bash
# Create restricted API key
vastai create api-key --name "limited-key" --permission_file permissions.json

# Show API keys
vastai show api-keys

# Delete API key
vastai delete api-key KEY_ID

# Reset API key
vastai reset api-key
```

### Templates

```bash
# Create template
vastai create template --name "my-template" --image pytorch/pytorch:latest \
  --jupyter --direct --env "-e MY_VAR=value"

# Search templates
vastai search templates

# Delete template
vastai delete template TEMPLATE_ID
```

### Teams and Subaccounts

```bash
# Create team
vastai create-team --team_name "ML Team"

# Invite member
vastai invite member --email user@example.com --role_id ROLE_ID

# Show team members
vastai show members

# Create team role
vastai create team-role --name "Developer" --permissions permissions.json

# Show roles
vastai show team-roles
```

### Environment Variables

```bash
# Create environment variable (available to all instances)
vastai create env-var MY_SECRET_KEY "secret_value"

# List environment variables
vastai show env-vars

# Update environment variable
vastai update env-var VAR_ID --value "new_value"

# Delete environment variable
vastai delete env-var VAR_ID
```

### Autoscaling and Endpoints

#### Create autoscaling worker groups
```bash
vastai create workergroup --template_hash HASH --search_params "gpu_ram>=24 num_gpus=1" \
  --endpoint_name my-endpoint --min_load 10 --target_util 0.9

# Show worker groups
vastai show workergroups

# Get worker group logs
vastai get wrkgrp-logs WORKERGROUP_ID

# Update worker group
vastai update workergroup WORKERGROUP_ID --cold_workers 2

# Delete worker group
vastai delete workergroup WORKERGROUP_ID
```

#### Create endpoints
```bash
vastai create endpoint --endpoint_name my-api --min_load 5 --max_workers 10

# Show endpoints
vastai show endpoints

# Get endpoint logs
vastai get endpt-logs ENDPOINT_ID

# Delete endpoint
vastai delete endpoint ENDPOINT_ID
```

## Host Commands (For Machine Providers)

### List machines for rent
```bash
# List a machine
vastai list machine MACHINE_ID --price_gpu 0.50 --price_inetu 0.01 --price_inetd 0.01

# Show your machines
vastai show machines

# Unlist a machine
vastai unlist machine MACHINE_ID
```

### Machine maintenance
```bash
# Schedule maintenance
vastai schedule maint MACHINE_ID --sdate UNIX_TIMESTAMP --duration 2.0

# Cancel maintenance
vastai cancel maint MACHINE_ID

# Show maintenance windows
vastai show maints -ids "MACHINE_ID1,MACHINE_ID2"
```

### Self-test machine
```bash
vastai self-test machine MACHINE_ID
vastai self-test machine MACHINE_ID --ignore-requirements
```

### Pricing
```bash
# Set minimum bid price
vastai set min-bid MACHINE_ID --price 0.25

# Set default job
vastai set defjob MACHINE_ID --price_gpu 0.50 --image ubuntu:22.04
```

## Clusters (Multi-node)

```bash
# Create cluster
vastai create cluster "10.0.0.0/24" MANAGER_MACHINE_ID

# Show clusters
vastai show clusters

# Join machine to cluster
vastai join cluster MACHINE_ID CLUSTER_ID

# Delete cluster
vastai delete cluster CLUSTER_ID
```

## Overlays (Virtual Networks)

```bash
# Create overlay network
vastai create overlay CLUSTER_ID "my-overlay"

# Show overlays
vastai show overlays

# Join instance to overlay
vastai join overlay INSTANCE_ID OVERLAY_ID

# Delete overlay
vastai delete overlay OVERLAY_ID
```

## Snapshots

```bash
# Take snapshot of running container
vastai take snapshot INSTANCE_ID --repo myuser/myimage \
  --docker_login_user myuser --docker_login_pass mytoken
```

## Cloud Connections

```bash
# Show configured cloud storage connections
vastai show connections
```

## Benchmarks

```bash
# Search GPU benchmarks
vastai search benchmarks
```

## Scheduled Jobs

```bash
# Show scheduled jobs
vastai show scheduled-jobs

# Delete scheduled job
vastai delete scheduled-job JOB_ID
```

## Common Workflows

### 1. Spin up a PyTorch training instance

```bash
# Search for suitable GPU
vastai search offers 'gpu_ram>=24 num_gpus>=2 reliability>0.95 inet_down>500' -o 'dph_total+'

# Create instance with the best offer
vastai create instance OFFER_ID \
  --image pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel \
  --disk 100 \
  --jupyter --direct \
  --onstart-cmd 'pip install wandb transformers datasets'

# Monitor instance
vastai show instance INSTANCE_ID

# Get Jupyter URL
vastai ssh-url INSTANCE_ID
```

### 2. Run a batch training job

```bash
# Create instance with onstart script
vastai create instance OFFER_ID \
  --image pytorch/pytorch:latest \
  --disk 50 \
  --ssh --direct \
  --onstart-cmd 'git clone https://github.com/myrepo/training.git && cd training && pip install -r requirements.txt && python train.py'

# Check logs
vastai logs INSTANCE_ID --tail
```

### 3. Set up persistent storage workflow

```bash
# Find and create a volume
vastai search volumes 'disk_space>=100'
vastai create volume OFFER_ID --size 100 --name training-data

# Create instance with volume attached
vastai create instance GPU_OFFER_ID \
  --image pytorch/pytorch:latest \
  --link-volume VOLUME_ID \
  --mount-path /data

# Your data persists even after instance destruction
```

### 4. Cost-effective spot instance workflow

```bash
# Search for interruptible offers
vastai search offers --type bid 'gpu_name=RTX_4090'

# Create spot instance with max bid
vastai create instance OFFER_ID \
  --image myimage \
  --bid_price 0.30

# Implement checkpointing in your code to handle interruptions
```

## Best Practices

1. **Always use `--raw` for scripting** - Returns JSON output that's easy to parse
2. **Check reliability scores** - Filter for `reliability > 0.95` for production workloads
3. **Use volumes for data persistence** - Instance local storage is ephemeral
4. **Implement checkpointing** - Especially for spot/interruptible instances
5. **Use onstart scripts** - For reproducible environment setup
6. **Monitor costs** - Use `vastai show invoices` regularly
7. **Use templates** - For repeatable instance configurations
8. **Direct connections** - Use `--direct` for faster SSH/Jupyter access

## Error Handling

Common errors and solutions:

| Error | Solution |
|-------|----------|
| "API key not found" | Run `vastai set api-key YOUR_KEY` |
| "Insufficient balance" | Add credits at https://vast.ai/console/billing |
| "Offer no longer available" | Re-run search and use a different offer ID |
| "Instance failed to start" | Check `vastai logs INSTANCE_ID` for details |
| "Port not available" | Try different direct_port_count filter |

## JSON Output Format

When using `--raw`, output is JSON. Example for `vastai show instances --raw`:

```json
[
  {
    "id": 12345,
    "machine_id": 67890,
    "actual_status": "running",
    "num_gpus": 2,
    "gpu_name": "RTX 4090",
    "dph_total": 0.75,
    "ssh_host": "ssh5.vast.ai",
    "ssh_port": 22345,
    "image_uuid": "pytorch/pytorch:latest"
  }
]
```

## Related Resources

- **Documentation**: https://vast.ai/docs
- **API Reference**: https://docs.vast.ai/api
- **Console**: https://console.vast.ai
- **Python SDK**: https://github.com/vast-ai/vast-sdk
- **Discord Support**: #api channel

## Version

This skill is compatible with vastai CLI version 0.x and above. Run `vastai --version` to check your version.
