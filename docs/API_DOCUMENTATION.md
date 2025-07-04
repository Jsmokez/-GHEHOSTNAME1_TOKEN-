# API Documentation

## Overview

This document provides comprehensive API documentation for the GitHub workflows and templates in this repository. It includes detailed information about inputs, outputs, configuration options, and integration patterns.

## GitHub Actions Workflows API

### CodeQL Advanced Security Analysis Workflow

**File**: `.github/workflows/codeql.yml`

#### Workflow Configuration

```yaml
name: "CodeQL Advanced"
```

#### Triggers

| Trigger Type | Configuration | Description |
|--------------|---------------|-------------|
| `push` | `branches: ["main"]` | Automatic scan on main branch commits |
| `pull_request` | `branches: ["main"]` | Scan all PRs targeting main branch |
| `schedule` | `cron: '19 13 * * 2'` | Weekly scan every Tuesday at 1:19 PM UTC |

#### Matrix Strategy

The workflow uses a dynamic matrix strategy that auto-detects languages:

```yaml
strategy:
  fail-fast: false
  matrix:
    include:
    # Languages are auto-detected based on repository content
    # Supported: c-cpp, csharp, go, java-kotlin, javascript-typescript, python, ruby, swift
```

#### Inputs

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `languages` | string | No | Auto-detected | Comma-separated list of languages to analyze |
| `build-mode` | string | No | `autobuild` | Build mode: `autobuild`, `manual`, or `none` |
| `queries` | string | No | Default queries | Custom CodeQL query packs to run |

#### Outputs

| Output | Type | Description |
|--------|------|-------------|
| Security alerts | SARIF | Security vulnerabilities uploaded to GitHub Security tab |
| Analysis results | JSON | Detailed analysis results in CodeQL format |
| Build logs | Text | Compilation and analysis logs |

#### Configuration Examples

**Basic Language Override**:
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript-typescript,python
```

**Custom Queries**:
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    queries: security-extended,security-and-quality
```

**Manual Build Mode**:
```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    build-mode: manual

- name: Manual Build
  run: |
    make clean
    make build
    make test
```

#### Error Handling

Common error scenarios and solutions:

| Error | Cause | Solution |
|-------|-------|----------|
| `No source code was seen during the build` | Autobuild failed | Switch to manual build mode |
| `Language not supported` | Invalid language specified | Use supported language identifiers |
| `Out of memory` | Large codebase | Use larger runners or reduce scope |

### Basic CI Workflow

**File**: `.github/workflows/GHCR_TOKEN`

#### Workflow Configuration

```yaml
name: CI
```

#### Triggers

| Trigger Type | Configuration | Description |
|--------------|---------------|-------------|
| `push` | `branches: ["main"]` | Run on main branch commits |
| `pull_request` | `branches: ["main"]` | Run on PRs to main |
| `workflow_dispatch` | Manual trigger | Allow manual execution |

#### Job Configuration

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

#### Steps API

| Step | Action | Purpose |
|------|--------|---------|
| Checkout | `actions/checkout@v4` | Clone repository code |
| One-line script | `run: echo Hello, world!` | Template for simple commands |
| Multi-line script | `run: \| ...` | Template for complex operations |

#### Customization Examples

**Adding Environment Variables**:
```yaml
- name: Run with environment
  env:
    NODE_ENV: production
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    echo "Environment: $NODE_ENV"
    npm run build
```

**Matrix Strategy**:
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest, macos-latest]
```

## CSV Template API

### Data Structure

#### Schema Definition

```typescript
interface SponsorshipRecord {
  maintainerUsername: string;  // GitHub username
  sponsorshipAmount: string;   // USD amount (numeric or $-prefixed)
}
```

#### Field Specifications

| Field | Type | Format | Validation Rules |
|-------|------|--------|------------------|
| `Maintainer username` | String | GitHub username | 1-39 characters, alphanumeric + hyphens |
| `Sponsorship amount in USD` | String | `100` or `$100` | Positive numeric value |

#### CSV Format

```csv
Maintainer username,Sponsorship amount in USD
username1,100
username2,$50
username3,25
```

### Processing API

#### Python Implementation

```python
import csv
from typing import List, Dict
import re

class SponsorshipProcessor:
    def __init__(self, csv_file: str):
        self.csv_file = csv_file
        self.records: List[Dict[str, str]] = []
    
    def load_data(self) -> None:
        """Load sponsorship data from CSV file."""
        with open(self.csv_file, 'r', newline='') as file:
            reader = csv.DictReader(file)
            self.records = list(reader)
    
    def validate_record(self, record: Dict[str, str]) -> bool:
        """Validate a single sponsorship record."""
        username = record.get('Maintainer username', '').strip()
        amount = record.get('Sponsorship amount in USD', '').strip()
        
        # Validate username format
        username_pattern = r'^[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?$'
        if not re.match(username_pattern, username) or len(username) > 39:
            return False
        
        # Validate amount format
        amount_clean = amount.replace('$', '').replace(',', '')
        try:
            float_amount = float(amount_clean)
            return float_amount > 0
        except ValueError:
            return False
    
    def process_sponsorships(self) -> List[Dict[str, any]]:
        """Process all valid sponsorship records."""
        valid_records = []
        
        for record in self.records:
            if self.validate_record(record):
                username = record['Maintainer username'].strip()
                amount_str = record['Sponsorship amount in USD'].strip()
                amount = float(amount_str.replace('$', '').replace(',', ''))
                
                valid_records.append({
                    'username': username,
                    'amount': amount,
                    'currency': 'USD'
                })
        
        return valid_records

# Usage example
processor = SponsorshipProcessor('bulk-sponsorships-template.csv')
processor.load_data()
sponsorships = processor.process_sponsorships()

for sponsorship in sponsorships:
    print(f"Sponsor {sponsorship['username']}: ${sponsorship['amount']}")
```

#### JavaScript Implementation

```javascript
const fs = require('fs');
const csv = require('csv-parser');

class SponsorshipProcessor {
    constructor(csvFile) {
        this.csvFile = csvFile;
        this.records = [];
    }

    async loadData() {
        return new Promise((resolve, reject) => {
            const results = [];
            fs.createReadStream(this.csvFile)
                .pipe(csv())
                .on('data', (data) => results.push(data))
                .on('end', () => {
                    this.records = results;
                    resolve();
                })
                .on('error', reject);
        });
    }

    validateRecord(record) {
        const username = record['Maintainer username']?.trim() || '';
        const amount = record['Sponsorship amount in USD']?.trim() || '';

        // Validate username
        const usernamePattern = /^[a-zA-Z0-9]([a-zA-Z0-9\-]*[a-zA-Z0-9])?$/;
        if (!usernamePattern.test(username) || username.length > 39) {
            return false;
        }

        // Validate amount
        const amountClean = amount.replace(/[$,]/g, '');
        const numericAmount = parseFloat(amountClean);
        return !isNaN(numericAmount) && numericAmount > 0;
    }

    processSponsorships() {
        const validRecords = [];

        for (const record of this.records) {
            if (this.validateRecord(record)) {
                const username = record['Maintainer username'].trim();
                const amountStr = record['Sponsorship amount in USD'].trim();
                const amount = parseFloat(amountStr.replace(/[$,]/g, ''));

                validRecords.push({
                    username,
                    amount,
                    currency: 'USD'
                });
            }
        }

        return validRecords;
    }
}

// Usage example
async function processSponsorshipFile() {
    const processor = new SponsorshipProcessor('bulk-sponsorships-template.csv');
    await processor.loadData();
    const sponsorships = processor.processSponsorships();
    
    sponsorships.forEach(sponsorship => {
        console.log(`Sponsor ${sponsorship.username}: $${sponsorship.amount}`);
    });
}
```

## Integration Patterns

### GitHub API Integration

#### Authentication

```bash
# Using GitHub CLI
gh auth login

# Using personal access token
export GITHUB_TOKEN="your_token_here"
```

#### Sponsorship API Example

```python
import requests

def create_sponsorship(username: str, amount: float, token: str):
    """Create a sponsorship using GitHub API."""
    headers = {
        'Authorization': f'Bearer {token}',
        'Accept': 'application/vnd.github+json',
        'X-GitHub-Api-Version': '2022-11-28'
    }
    
    # Note: Actual sponsorship API endpoints may vary
    url = f'https://api.github.com/user/sponsorships'
    data = {
        'maintainer': username,
        'amount_cents': int(amount * 100),
        'currency': 'USD'
    }
    
    response = requests.post(url, headers=headers, json=data)
    return response.json()
```

### Workflow Integration

#### Triggering Workflows

```bash
# Trigger workflow manually
gh workflow run "CodeQL Advanced" --ref main

# Check workflow status
gh run list --workflow="CodeQL Advanced"
```

#### Accessing Results

```python
import requests

def get_codeql_results(repo: str, token: str):
    """Fetch CodeQL analysis results."""
    headers = {
        'Authorization': f'Bearer {token}',
        'Accept': 'application/vnd.github+json'
    }
    
    url = f'https://api.github.com/repos/{repo}/code-scanning/alerts'
    response = requests.get(url, headers=headers)
    return response.json()
```

## Error Codes and Troubleshooting

### Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `CSV_PARSE_ERROR` | Invalid CSV format | Check file encoding and structure |
| `VALIDATION_FAILED` | Record validation failed | Verify username and amount formats |
| `API_RATE_LIMIT` | GitHub API rate limit exceeded | Implement request throttling |
| `WORKFLOW_FAILED` | GitHub Actions workflow failed | Check workflow logs and permissions |

### Debugging Tools

#### CSV Validation Script

```python
def debug_csv_file(filename: str):
    """Debug CSV file structure and content."""
    import csv
    
    try:
        with open(filename, 'r') as file:
            # Check encoding
            content = file.read()
            print(f"File size: {len(content)} characters")
            
        with open(filename, 'r', newline='') as file:
            # Check CSV structure
            sniffer = csv.Sniffer()
            dialect = sniffer.sniff(file.read(1024))
            file.seek(0)
            
            reader = csv.DictReader(file, dialect=dialect)
            headers = reader.fieldnames
            print(f"Headers: {headers}")
            
            # Validate each row
            for i, row in enumerate(reader, 1):
                print(f"Row {i}: {row}")
                
    except Exception as e:
        print(f"Error reading CSV: {e}")

# Usage
debug_csv_file('bulk-sponsorships-template.csv')
```

## Version Compatibility

### GitHub Actions Versions

| Component | Current Version | Minimum Version | Notes |
|-----------|----------------|-----------------|-------|
| `actions/checkout` | v4 | v3 | Repository checkout |
| `github/codeql-action/init` | v3 | v2 | CodeQL initialization |
| `github/codeql-action/analyze` | v3 | v2 | CodeQL analysis |

### API Versions

| API | Version | Compatibility |
|-----|---------|---------------|
| GitHub REST API | 2022-11-28 | Current stable |
| GitHub GraphQL API | v4 | Current stable |
| CodeQL CLI | Latest | Auto-updated |

This documentation provides comprehensive coverage of all APIs and integration patterns available in this repository.