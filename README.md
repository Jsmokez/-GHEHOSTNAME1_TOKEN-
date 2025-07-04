# Repository Documentation

## Overview

This repository contains GitHub Actions workflows and templates for managing code security scanning and bulk sponsorship operations. It provides automated security analysis through CodeQL and standardized templates for processing sponsorship data.

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── codeql.yml                    # Advanced CodeQL security analysis workflow
│       ├── `GHCR_TOKEN`                  # Basic CI workflow template
│       └── `codeql-analysis.yml`         # CodeQL analysis configuration
├── bulk-sponsorships-template.csv        # Primary sponsorship template
├── bulk-sponsorships-template (2).csv    # Secondary sponsorship template
└── README.md                             # This documentation file
```

## GitHub Workflows

### 1. CodeQL Advanced Security Analysis (`codeql.yml`)

**Purpose**: Automated security vulnerability scanning using GitHub's CodeQL engine.

**Triggers**:
- Push to `main` branch
- Pull requests to `main` branch  
- Scheduled runs (Tuesdays at 13:19 UTC)

**Features**:
- Multi-language support detection
- Configurable build modes (automatic/manual)
- Security and quality analysis
- Custom query support

**Usage**:
```yaml
# The workflow automatically detects languages in your repository
# Supported languages: c-cpp, csharp, go, java-kotlin, javascript-typescript, python, ruby, swift
```

**Configuration Options**:
- **Custom Queries**: Uncomment and modify the `queries` parameter to add custom security rules
- **Build Mode**: Set to "manual" for compiled languages requiring custom build steps
- **Languages**: The matrix automatically includes detected languages

**Example Customization**:
```yaml
# To add custom queries, modify this section in codeql.yml:
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    queries: security-extended,security-and-quality
```

### 2. Basic CI Workflow (`GHCR_TOKEN`)

**Purpose**: Template workflow for basic continuous integration tasks.

**Triggers**:
- Push to `main` branch
- Pull requests to `main` branch
- Manual workflow dispatch

**Current Actions**:
- Repository checkout
- Basic shell command execution
- Multi-line script support

**Usage Example**:
```yaml
# Customize the workflow by modifying the run commands:
- name: Run a one-line script
  run: echo Hello, world!

- name: Run a multi-line script
  run: |
    echo Add other actions to build,
    echo test, and deploy your project.
```

## Sponsorship Templates

### CSV Template Structure

Both sponsorship template files follow the same structure for processing bulk sponsorship data.

**Columns**:
- `Maintainer username`: GitHub username of the project maintainer
- `Sponsorship amount in USD`: Monetary amount (accepts formats: `100` or `$100`)

### Primary Template (`bulk-sponsorships-template.csv`)

```csv
Maintainer username,Sponsorship amount in USD
maintainerUsername1,100 or $100
maintainerUsername2,100 or $100
```

### Usage Instructions

1. **Download the template**:
   ```bash
   curl -O https://raw.githubusercontent.com/[your-org]/[your-repo]/main/bulk-sponsorships-template.csv
   ```

2. **Fill in sponsorship data**:
   - Replace `maintainerUsername1`, `maintainerUsername2` with actual GitHub usernames
   - Replace `100 or $100` with actual sponsorship amounts
   - Add additional rows as needed

3. **Amount Format Options**:
   - Numeric only: `100`
   - With currency symbol: `$100`
   - Both formats are accepted

**Example Populated Template**:
```csv
Maintainer username,Sponsorship amount in USD
octocat,50
defunkt,$100
mojombo,25
pjhyett,$75
```

### API Integration

These templates are designed to work with GitHub's Sponsors API. Here's how to process them:

**Example Processing Script** (Python):
```python
import csv
import requests

def process_sponsorships(csv_file, github_token):
    with open(csv_file, 'r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            username = row['Maintainer username']
            amount = row['Sponsorship amount in USD'].replace('$', '')
            
            # Process sponsorship via GitHub API
            # (Implementation depends on specific API endpoints)
            print(f"Processing ${amount} sponsorship for {username}")

# Usage
process_sponsorships('bulk-sponsorships-template.csv', 'your_github_token')
```

## Security Considerations

### CodeQL Configuration

- **Permissions**: The CodeQL workflow uses minimal required permissions
- **Secrets**: No additional secrets required for basic operation
- **Custom Queries**: Review custom queries for security implications before adding

### Token Management

- Store GitHub tokens in repository secrets, not in workflow files
- Use the principle of least privilege for API tokens
- Regularly rotate access tokens

## Contributing

### Adding New Workflows

1. Create new workflow files in `.github/workflows/`
2. Follow GitHub Actions best practices
3. Test workflows in a fork before merging
4. Update this documentation

### Modifying Templates

1. Maintain CSV header structure for compatibility
2. Validate data formats before committing
3. Test with small datasets first
4. Document any format changes

## Troubleshooting

### CodeQL Issues

**Problem**: "We were unable to automatically build your code"
**Solution**: Set `build-mode: manual` and add custom build steps

**Problem**: Analysis taking too long
**Solution**: Consider using larger runners for resource-intensive projects

### CSV Template Issues

**Problem**: Invalid amount formats
**Solution**: Ensure amounts are numeric or include only `$` prefix

**Problem**: Missing usernames
**Solution**: Verify all GitHub usernames exist and are spelled correctly

## API References

- [GitHub CodeQL API Documentation](https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/using-code-scanning-api)
- [GitHub Sponsors API](https://docs.github.com/en/rest/sponsors)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

## License

Please refer to the repository's license file for usage terms and conditions.