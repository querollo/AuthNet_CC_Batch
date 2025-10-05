# Authorize.Net Expired Card Scanner

A Python script that scans all customer profiles in your Authorize.Net account to identify expired credit cards and exports them to a CSV file.

## Features

- Scans all customer payment profiles for expired credit cards
- Fast concurrent processing (10 parallel workers)
- Incremental progress saving - resume from interruptions
- Exports results to CSV with full customer details
- Secure credential management via environment variables

## Requirements

- Python 3.6 or higher
- Authorize.Net SDK for Python
- Valid Authorize.Net merchant account credentials

## Installation

1. **Install the Authorize.Net SDK:**
   ```bash
   pip install authorizenet
   ```

2. **Download the script** to your desired location

3. **Set up environment variables** (see Configuration below)

## Configuration

### Required Environment Variables

The script requires two credentials from your Authorize.Net account:

**On Windows:**
```cmd
set AUTHNET_LOGIN_ID=your_api_login_id
set AUTHNET_TRANSACTION_KEY=your_transaction_key
```

**On Mac/Linux:**
```bash
export AUTHNET_LOGIN_ID=your_api_login_id
export AUTHNET_TRANSACTION_KEY=your_transaction_key
```

**To make them permanent:**
- Windows: Add them to System Environment Variables
- Mac/Linux: Add the export commands to your `~/.bashrc` or `~/.zshrc`

### Finding Your Credentials

1. Log into your Authorize.Net merchant account
2. Navigate to **Account** > **Settings** > **API Credentials & Keys**
3. Copy your **API Login ID** and **Transaction Key**

### Script Settings

You can modify these constants in the script:

```python
ENV = constants.PRODUCTION          # Use SANDBOX for testing
MAX_WORKERS = 10                     # Number of concurrent threads
BATCH_SIZE = 5000                    # Save progress every N profiles
```

## Usage

### Basic Usage

Simply run the script:

```bash
python expired_card_scanner.py
```

### What Happens

1. The script fetches all customer profile IDs from your account
2. Processes profiles in batches of 5,000
3. For each profile, checks all payment methods for expired cards
4. Saves results incrementally to `expired_cards_output.csv`
5. Tracks progress in `scan_progress.txt`

### Sample Output

```
Starting expired credit card scan with INCREMENTAL SAVE...
Progress will be saved every 5000 profiles.
You can stop and restart this script at any time!

STARTING NEW SCAN of 15000 profiles

Processing batch: 0 to 5000 (5000 profiles)...
  Batch progress: 500/5000... (12 expired found in batch)
  Batch progress: 1000/5000... (27 expired found in batch)
  ...
✓ Saved 45 customers with expired cards from this batch
✓ Progress saved: 5000/15000 profiles completed
✓ Total expired cards found so far: 45
```

### Resuming After Interruption

If the script stops (intentionally or due to an error), simply run it again:

```bash
python expired_card_scanner.py
```

It will automatically resume from the last saved checkpoint.

## Output

### CSV File Structure

The script creates `expired_cards_output.csv` with the following columns:

| Column | Description |
|--------|-------------|
| customerProfileId | Authorize.Net customer profile ID |
| merchantCustomerId | Your custom customer ID (if set) |
| email | Customer email address |
| description | Customer description/name |
| paymentProfileId | Payment method ID |
| cardNumber | Masked card number (last 4 digits) |
| expirationDate | Card expiration date |

### Example CSV Output

```csv
customerProfileId,merchantCustomerId,email,description,paymentProfileId,cardNumber,expirationDate
12345678,CUST001,john@example.com,John Doe,87654321,XXXX1234,12/2024
12345679,CUST002,jane@example.com,Jane Smith,87654322,XXXX5678,11/2024
```

## How It Works

1. **Fetches Profile IDs** - Retrieves all customer profile IDs from your account
2. **Concurrent Processing** - Uses 10 parallel workers to fetch profile details quickly
3. **Expiration Check** - Parses various date formats and compares to current date
4. **Incremental Save** - Saves results every 5,000 profiles to prevent data loss
5. **Progress Tracking** - Maintains a checkpoint file for resumable operation

## Performance

- **Speed**: Processes ~500 profiles per minute (varies by API response time)
- **For 10,000 profiles**: Approximately 20 minutes
- **For 50,000 profiles**: Approximately 100 minutes

## Troubleshooting

### "Missing credentials" Error

**Problem**: Script exits immediately with credential error

**Solution**: Ensure environment variables are set correctly:
```bash
echo $AUTHNET_LOGIN_ID        # Mac/Linux
echo %AUTHNET_LOGIN_ID%       # Windows
```

### API Errors

**Problem**: "Error fetching profile IDs" or failed requests

**Solutions**:
- Verify credentials are correct
- Check you're using the right environment (PRODUCTION vs SANDBOX)
- Ensure your API credentials have proper permissions
- Check Authorize.Net service status

### Slow Performance

**Problem**: Script running slower than expected

**Solutions**:
- Reduce `MAX_WORKERS` to 5 if hitting rate limits
- Check your internet connection
- Verify Authorize.Net API is responding normally


