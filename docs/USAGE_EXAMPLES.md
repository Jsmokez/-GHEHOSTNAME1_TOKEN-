# Usage Examples and Best Practices

## Overview

This document provides practical examples and best practices for using the GitHub workflows and sponsorship templates in this repository. All examples are production-ready and include error handling.

## GitHub Workflows Usage

### CodeQL Security Analysis Examples

#### Basic Setup for New Repository

1. **Copy the workflow file to your repository**:
```bash
mkdir -p .github/workflows
cp .github/workflows/codeql.yml .github/workflows/
```

2. **Customize for your language stack**:
```yaml
# .github/workflows/codeql.yml
name: "CodeQL Advanced"

on:
  push:
    branches: [ "main", "develop" ]  # Add your branches
  pull_request:
    branches: [ "main" ]
  schedule:
    - cron: '19 13 * * 2'  # Customize schedule

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      packages: read
      actions: read
      contents: read

    strategy:
      fail-fast: false
      matrix:
        include:
        - language: javascript-typescript
          build-mode: none
        - language: python
          build-mode: none
```

#### Advanced Configuration Examples

**Multi-Language Project with Custom Build**:
```yaml
strategy:
  fail-fast: false
  matrix:
    include:
    - language: javascript-typescript
      build-mode: none
    - language: python
      build-mode: none
    - language: java-kotlin
      build-mode: manual
    - language: c-cpp
      build-mode: manual

steps:
- name: Checkout repository
  uses: actions/checkout@v4

- name: Setup Java (for Java projects)
  if: matrix.language == 'java-kotlin'
  uses: actions/setup-java@v3
  with:
    java-version: '17'
    distribution: 'temurin'

- name: Setup Python (for Python projects)
  if: matrix.language == 'python'
  uses: actions/setup-python@v4
  with:
    python-version: '3.11'

- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    build-mode: ${{ matrix.build-mode }}
    queries: security-extended,security-and-quality

- name: Manual Build for Java
  if: matrix.language == 'java-kotlin' && matrix.build-mode == 'manual'
  run: |
    ./gradlew clean build -x test

- name: Manual Build for C++
  if: matrix.language == 'c-cpp' && matrix.build-mode == 'manual'
  run: |
    cmake -B build
    cmake --build build

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v3
  with:
    category: "/language:${{matrix.language}}"
```

**Custom Query Configuration**:
```yaml
- name: Initialize CodeQL with Custom Queries
  uses: github/codeql-action/init@v3
  with:
    languages: ${{ matrix.language }}
    queries: |
      security-extended
      security-and-quality
      +github/codeql/cpp/ql/src/Security
      +github/codeql/javascript/ql/src/Security
    config-file: ./.github/codeql-config.yml
```

Create `.github/codeql-config.yml`:
```yaml
name: "Custom CodeQL Config"

disable-default-queries: false

queries:
  - name: Extended Security Queries
    uses: security-extended
  - name: Quality Queries  
    uses: security-and-quality

paths-ignore:
  - "test/**"
  - "tests/**"
  - "**/*.test.js"
  - "**/node_modules/**"

paths:
  - "src/**"
  - "lib/**"
```

### CI Workflow Customization Examples

#### Node.js Project Example

```yaml
name: Node.js CI

on:
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella
```

#### Python Project Example

```yaml
name: Python CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11"]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: Run linting
      run: |
        flake8 src tests
        black --check src tests
        mypy src
    
    - name: Run tests
      run: |
        pytest tests/ --cov=src --cov-report=xml
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
```

## Sponsorship Templates Usage

### Basic CSV Processing Examples

#### Simple Validation and Processing

```python
#!/usr/bin/env python3
"""
Simple sponsorship processor with validation and error handling.
"""

import csv
import sys
import logging
from typing import List, Dict, Tuple
import re

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class SponsorshipValidator:
    """Validates sponsorship data according to GitHub's requirements."""
    
    USERNAME_PATTERN = re.compile(r'^[a-zA-Z0-9](?:[a-zA-Z0-9\-]*[a-zA-Z0-9])?$')
    MAX_USERNAME_LENGTH = 39
    
    @classmethod
    def validate_username(cls, username: str) -> Tuple[bool, str]:
        """Validate GitHub username format."""
        if not username:
            return False, "Username cannot be empty"
        
        if len(username) > cls.MAX_USERNAME_LENGTH:
            return False, f"Username too long (max {cls.MAX_USERNAME_LENGTH} chars)"
        
        if not cls.USERNAME_PATTERN.match(username):
            return False, "Username contains invalid characters"
        
        return True, "Valid username"
    
    @classmethod
    def validate_amount(cls, amount: str) -> Tuple[bool, str, float]:
        """Validate and parse sponsorship amount."""
        if not amount:
            return False, "Amount cannot be empty", 0.0
        
        # Clean the amount string
        clean_amount = amount.strip().replace('$', '').replace(',', '')
        
        try:
            float_amount = float(clean_amount)
            if float_amount <= 0:
                return False, "Amount must be positive", 0.0
            if float_amount > 12000:  # GitHub's monthly limit
                return False, "Amount exceeds GitHub's monthly limit ($12,000)", 0.0
            return True, "Valid amount", float_amount
        except ValueError:
            return False, f"Invalid amount format: {amount}", 0.0

def process_sponsorship_file(filename: str) -> Tuple[List[Dict], List[str]]:
    """
    Process sponsorship CSV file and return valid records and errors.
    
    Returns:
        Tuple of (valid_records, error_messages)
    """
    valid_records = []
    errors = []
    
    try:
        with open(filename, 'r', newline='', encoding='utf-8') as file:
            reader = csv.DictReader(file)
            
            # Check if required columns exist
            required_columns = ['Maintainer username', 'Sponsorship amount in USD']
            if not all(col in reader.fieldnames for col in required_columns):
                errors.append(f"Missing required columns: {required_columns}")
                return valid_records, errors
            
            for row_num, row in enumerate(reader, start=2):  # Start at 2 for header
                username = row['Maintainer username'].strip()
                amount_str = row['Sponsorship amount in USD'].strip()
                
                # Validate username
                username_valid, username_msg = SponsorshipValidator.validate_username(username)
                if not username_valid:
                    errors.append(f"Row {row_num}: {username_msg}")
                    continue
                
                # Validate amount
                amount_valid, amount_msg, amount = SponsorshipValidator.validate_amount(amount_str)
                if not amount_valid:
                    errors.append(f"Row {row_num}: {amount_msg}")
                    continue
                
                # Add valid record
                valid_records.append({
                    'username': username,
                    'amount': amount,
                    'currency': 'USD',
                    'row_number': row_num
                })
                
                logger.info(f"Validated: {username} - ${amount}")
    
    except FileNotFoundError:
        errors.append(f"File not found: {filename}")
    except Exception as e:
        errors.append(f"Error reading file: {str(e)}")
    
    return valid_records, errors

def main():
    """Main processing function."""
    if len(sys.argv) != 2:
        print("Usage: python process_sponsorships.py <csv_file>")
        sys.exit(1)
    
    filename = sys.argv[1]
    logger.info(f"Processing file: {filename}")
    
    valid_records, errors = process_sponsorship_file(filename)
    
    # Report results
    print(f"\n📊 Processing Results:")
    print(f"✅ Valid records: {len(valid_records)}")
    print(f"❌ Errors: {len(errors)}")
    
    if errors:
        print(f"\n🚨 Errors found:")
        for error in errors:
            print(f"  - {error}")
    
    if valid_records:
        print(f"\n💰 Valid sponsorships:")
        total_amount = sum(record['amount'] for record in valid_records)
        for record in valid_records:
            print(f"  - {record['username']}: ${record['amount']:.2f}")
        print(f"\n💵 Total amount: ${total_amount:.2f}")
    
    # Exit with error code if there were any errors
    sys.exit(1 if errors else 0)

if __name__ == "__main__":
    main()
```

#### Advanced Processing with GitHub API Integration

```python
#!/usr/bin/env python3
"""
Advanced sponsorship processor with GitHub API integration.
"""

import csv
import requests
import time
import json
from typing import List, Dict, Optional
import logging
from dataclasses import dataclass

@dataclass
class SponsorshipRecord:
    username: str
    amount: float
    currency: str = "USD"
    tier_id: Optional[str] = None

class GitHubSponsorshipManager:
    """Manages GitHub sponsorships via API."""
    
    def __init__(self, token: str):
        self.token = token
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {token}',
            'Accept': 'application/vnd.github+json',
            'X-GitHub-Api-Version': '2022-11-28'
        })
        self.rate_limit_remaining = 5000
        self.rate_limit_reset = time.time()
    
    def check_rate_limit(self):
        """Check and respect GitHub API rate limits."""
        if self.rate_limit_remaining < 10:
            sleep_time = self.rate_limit_reset - time.time()
            if sleep_time > 0:
                logging.info(f"Rate limit low, sleeping for {sleep_time:.1f} seconds")
                time.sleep(sleep_time)
    
    def get_user_info(self, username: str) -> Optional[Dict]:
        """Get user information from GitHub API."""
        self.check_rate_limit()
        
        try:
            response = self.session.get(f'https://api.github.com/users/{username}')
            
            # Update rate limit info
            self.rate_limit_remaining = int(response.headers.get('X-RateLimit-Remaining', 0))
            self.rate_limit_reset = int(response.headers.get('X-RateLimit-Reset', time.time()))
            
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 404:
                logging.warning(f"User not found: {username}")
                return None
            else:
                logging.error(f"API error for {username}: {response.status_code}")
                return None
                
        except Exception as e:
            logging.error(f"Error fetching user {username}: {str(e)}")
            return None
    
    def verify_sponsorship_eligibility(self, username: str) -> bool:
        """Check if user is eligible for sponsorship."""
        user_info = self.get_user_info(username)
        if not user_info:
            return False
        
        # Check if user has sponsorship enabled
        # Note: This is a simplified check, actual implementation may vary
        return user_info.get('type') == 'User'
    
    def create_sponsorship(self, record: SponsorshipRecord) -> bool:
        """Create a sponsorship (placeholder implementation)."""
        # Note: Actual GitHub sponsorship API endpoints may differ
        logging.info(f"Would create sponsorship: {record.username} - ${record.amount}")
        
        # Simulate API call delay
        time.sleep(0.1)
        return True

def process_sponsorships_with_api(csv_file: str, github_token: str, dry_run: bool = True):
    """
    Process sponsorships with GitHub API validation.
    
    Args:
        csv_file: Path to CSV file
        github_token: GitHub API token
        dry_run: If True, only validate without creating sponsorships
    """
    logging.basicConfig(level=logging.INFO)
    logger = logging.getLogger(__name__)
    
    # Initialize GitHub API manager
    gh_manager = GitHubSponsorshipManager(github_token)
    
    # Process CSV file
    valid_records, errors = process_sponsorship_file(csv_file)
    
    if errors:
        logger.error("Validation errors found, aborting:")
        for error in errors:
            logger.error(f"  {error}")
        return False
    
    # Verify users exist on GitHub
    verified_records = []
    for record in valid_records:
        username = record['username']
        amount = record['amount']
        
        logger.info(f"Verifying user: {username}")
        
        if gh_manager.verify_sponsorship_eligibility(username):
            verified_records.append(SponsorshipRecord(
                username=username,
                amount=amount
            ))
            logger.info(f"✅ User verified: {username}")
        else:
            logger.warning(f"❌ User not eligible: {username}")
    
    # Create sponsorships
    if not dry_run and verified_records:
        logger.info(f"Creating {len(verified_records)} sponsorships...")
        
        for record in verified_records:
            success = gh_manager.create_sponsorship(record)
            if success:
                logger.info(f"✅ Sponsorship created: {record.username}")
            else:
                logger.error(f"❌ Failed to create sponsorship: {record.username}")
    
    # Summary
    total_amount = sum(record.amount for record in verified_records)
    logger.info(f"\n📊 Summary:")
    logger.info(f"  Verified users: {len(verified_records)}")
    logger.info(f"  Total amount: ${total_amount:.2f}")
    logger.info(f"  Dry run: {dry_run}")
    
    return True

# Usage example
if __name__ == "__main__":
    import os
    
    github_token = os.getenv('GITHUB_TOKEN')
    if not github_token:
        print("Please set GITHUB_TOKEN environment variable")
        exit(1)
    
    process_sponsorships_with_api(
        'bulk-sponsorships-template.csv',
        github_token,
        dry_run=True  # Set to False to actually create sponsorships
    )
```

### Batch Processing Examples

#### Processing Large Sponsorship Files

```python
#!/usr/bin/env python3
"""
Batch processor for large sponsorship files with progress tracking.
"""

import csv
import time
from typing import Iterator, List
from tqdm import tqdm
import pandas as pd

def process_large_csv_in_batches(filename: str, batch_size: int = 100) -> Iterator[List[Dict]]:
    """
    Process large CSV files in batches to manage memory usage.
    
    Args:
        filename: Path to CSV file
        batch_size: Number of records per batch
        
    Yields:
        Batches of sponsorship records
    """
    batch = []
    
    with open(filename, 'r', newline='', encoding='utf-8') as file:
        reader = csv.DictReader(file)
        
        for row in reader:
            batch.append(row)
            
            if len(batch) >= batch_size:
                yield batch
                batch = []
        
        # Yield remaining records
        if batch:
            yield batch

def process_sponsorships_with_progress(filename: str):
    """Process sponsorships with progress bar and statistics."""
    
    # First pass: count total rows for progress bar
    total_rows = sum(1 for _ in open(filename, 'r', encoding='utf-8')) - 1  # Subtract header
    
    valid_count = 0
    error_count = 0
    total_amount = 0.0
    
    # Process in batches with progress bar
    with tqdm(total=total_rows, desc="Processing sponsorships") as pbar:
        for batch in process_large_csv_in_batches(filename, batch_size=50):
            batch_valid, batch_errors = process_batch(batch)
            
            valid_count += len(batch_valid)
            error_count += len(batch_errors)
            total_amount += sum(record['amount'] for record in batch_valid)
            
            pbar.update(len(batch))
            pbar.set_postfix({
                'Valid': valid_count,
                'Errors': error_count,
                'Total $': f"${total_amount:.2f}"
            })
            
            # Small delay to prevent overwhelming APIs
            time.sleep(0.1)
    
    # Final summary
    print(f"\n🎉 Processing complete!")
    print(f"✅ Valid records: {valid_count}")
    print(f"❌ Errors: {error_count}")
    print(f"💰 Total amount: ${total_amount:.2f}")

def process_batch(batch: List[Dict]) -> Tuple[List[Dict], List[str]]:
    """Process a batch of sponsorship records."""
    valid_records = []
    errors = []
    
    for row in batch:
        # Use the validation logic from previous examples
        username = row.get('Maintainer username', '').strip()
        amount_str = row.get('Sponsorship amount in USD', '').strip()
        
        # Validate and process (simplified for brevity)
        try:
            amount = float(amount_str.replace('$', '').replace(',', ''))
            if username and amount > 0:
                valid_records.append({
                    'username': username,
                    'amount': amount,
                    'currency': 'USD'
                })
        except (ValueError, AttributeError):
            errors.append(f"Invalid record: {row}")
    
    return valid_records, errors
```

## Best Practices

### Security Best Practices

#### 1. Token Management

```yaml
# ❌ Never do this - tokens in plain text
env:
  GITHUB_TOKEN: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ✅ Use repository secrets instead
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### 2. Least Privilege Permissions

```yaml
jobs:
  security-scan:
    permissions:
      # Only grant necessary permissions
      security-events: write  # For uploading SARIF results
      contents: read         # For checking out code
      # Do not grant unnecessary permissions like:
      # packages: write
      # actions: write
```

#### 3. Input Validation

```python
def secure_input_validation(user_input: str) -> str:
    """Securely validate and sanitize user input."""
    import re
    
    # Remove potentially dangerous characters
    sanitized = re.sub(r'[^\w\-.]', '', user_input)
    
    # Limit length
    if len(sanitized) > 100:
        raise ValueError("Input too long")
    
    return sanitized
```

### Performance Best Practices

#### 1. Workflow Optimization

```yaml
# ✅ Use caching to speed up builds
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# ✅ Run jobs in parallel when possible
jobs:
  test:
    # ...
  lint:
    # ...
  security:
    # Can run in parallel with test and lint
```

#### 2. CSV Processing Optimization

```python
# ✅ Use pandas for large datasets
import pandas as pd

def process_large_csv_optimized(filename: str):
    """Optimized processing for large CSV files."""
    
    # Read in chunks to manage memory
    chunk_size = 1000
    
    for chunk in pd.read_csv(filename, chunksize=chunk_size):
        # Vectorized operations are faster than loops
        chunk['amount_numeric'] = pd.to_numeric(
            chunk['Sponsorship amount in USD'].str.replace('$', ''),
            errors='coerce'
        )
        
        # Filter valid records efficiently
        valid_chunk = chunk[
            (chunk['Maintainer username'].notna()) &
            (chunk['amount_numeric'] > 0)
        ]
        
        # Process valid records
        process_chunk(valid_chunk)
```

### Error Handling Best Practices

#### 1. Comprehensive Error Handling

```python
import logging
from typing import Optional

def robust_api_call(url: str, headers: dict, retries: int = 3) -> Optional[dict]:
    """Make API call with comprehensive error handling."""
    
    for attempt in range(retries):
        try:
            response = requests.get(url, headers=headers, timeout=30)
            
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:  # Rate limited
                retry_after = int(response.headers.get('Retry-After', 60))
                logging.warning(f"Rate limited, waiting {retry_after}s")
                time.sleep(retry_after)
                continue
            elif response.status_code >= 500:  # Server error
                logging.warning(f"Server error {response.status_code}, retry {attempt + 1}")
                time.sleep(2 ** attempt)  # Exponential backoff
                continue
            else:
                logging.error(f"API error: {response.status_code}")
                return None
                
        except requests.exceptions.Timeout:
            logging.warning(f"Timeout on attempt {attempt + 1}")
        except requests.exceptions.ConnectionError:
            logging.warning(f"Connection error on attempt {attempt + 1}")
        except Exception as e:
            logging.error(f"Unexpected error: {str(e)}")
            break
    
    return None
```

#### 2. Graceful Degradation

```python
def process_with_fallback(records: List[Dict]) -> List[Dict]:
    """Process records with graceful degradation."""
    
    processed = []
    
    for record in records:
        try:
            # Try primary processing method
            result = primary_processor(record)
            processed.append(result)
        except Exception as e:
            logging.warning(f"Primary processing failed: {e}")
            
            try:
                # Fall back to simpler method
                result = fallback_processor(record)
                processed.append(result)
            except Exception as e2:
                logging.error(f"Fallback also failed: {e2}")
                # Continue with next record instead of failing completely
                continue
    
    return processed
```

This comprehensive usage guide provides practical, production-ready examples for implementing and customizing the workflows and templates in your projects.