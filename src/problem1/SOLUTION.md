# Notion format for this task
+ https://defiant-august-ce6.notion.site/Problem-1-Too-Many-Things-To-Do-1a285b7592a68143b3aedafa20e9aca7

# Problem 1: Too Many Things To Do
<aside>
⏰ Duration: You should not spend more than **2 hours** on this problem.
*Time estimation is for internship roles, if you are a software professional you should spend significantly less time.*

</aside>

# Task

Given the following file, write a CLI command to complete the objective:

Objective: submit a HTTP GET request to `https://example.com/api/:order_id` with Order IDs that are selling `TSLA`, writing to output to `./output.txt`

File: `./transaction-log.txt`

```
{"order_id": "12345", "symbol": "AAPL", "quantity": 100, "price": 142.50, "side": "buy", "timestamp": "2025-02-18T09:15:30Z"}
{"order_id": "12346", "symbol": "TSLA", "quantity": 50, "price": 890.15, "side": "sell", "timestamp": "2025-02-18T09:16:10Z"}
{"order_id": "12347", "symbol": "GOOG", "quantity": 10, "price": 2850.60, "side": "buy", "timestamp": "2025-02-18T09:17:20Z"}
{"order_id": "12348", "symbol": "AMZN", "quantity": 200, "price": 3201.45, "side": "sell", "timestamp": "2025-02-18T09:18:05Z"}
{"order_id": "12349", "symbol": "SPY", "quantity": 300, "price": 411.20, "side": "buy", "timestamp": "2025-02-18T09:19:15Z"}
{"order_id": "12350", "symbol": "MSFT", "quantity": 150, "price": 305.75, "side": "sell", "timestamp": "2025-02-18T09:20:00Z"}
{"order_id": "12351", "symbol": "NVDA", "quantity": 60, "price": 215.40, "side": "buy", "timestamp": "2025-02-18T09:20:45Z"}
{"order_id": "12352", "symbol": "NFLX", "quantity": 25, "price": 650.30, "side": "sell", "timestamp": "2025-02-18T09:21:25Z"}
{"order_id": "12353", "symbol": "BTCUSD", "quantity": 0.5, "price": 27000.00, "side": "buy", "timestamp": "2025-02-18T09:22:10Z"}
{"order_id": "12354", "symbol": "ETHUSD", "quantity": 2, "price": 1900.30, "side": "sell", "timestamp": "2025-02-18T09:23:00Z"}
{"order_id": "12355", "symbol": "XRPUSD", "quantity": 1000, "price": 0.85, "side": "buy", "timestamp": "2025-02-18T09:23:40Z"}
{"order_id": "12356", "symbol": "BABA", "quantity": 120, "price": 145.75, "side": "sell", "timestamp": "2025-02-18T09:24:25Z"}
{"order_id": "12357", "symbol": "SPY", "quantity": 50, "price": 413.50, "side": "buy", "timestamp": "2025-02-18T09:25:15Z"}
{"order_id": "12358", "symbol": "AAPL", "quantity": 30, "price": 141.10, "side": "sell", "timestamp": "2025-02-18T09:26:00Z"}
{"order_id": "12359", "symbol": "GOOG", "quantity": 75, "price": 2840.40, "side": "buy", "timestamp": "2025-02-18T09:26:45Z"}
{"order_id": "12360", "symbol": "AMZN", "quantity": 50, "price": 3215.60, "side": "sell", "timestamp": "2025-02-18T09:27:30Z"}
{"order_id": "12361", "symbol": "MSFT", "quantity": 180, "price": 307.10, "side": "buy", "timestamp": "2025-02-18T09:28:05Z"}
{"order_id": "12362", "symbol": "TSLA", "quantity": 60, "price": 885.10, "side": "sell", "timestamp": "2025-02-18T09:28:50Z"}
{"order_id": "12363", "symbol": "NVDA", "quantity": 120, "price": 220.80, "side": "buy", "timestamp": "2025-02-18T09:29:30Z"}
{"order_id": "12364", "symbol": "NFLX", "quantity": 40, "price": 648.50, "side": "sell", "timestamp": "2025-02-18T09:30:00Z"}
```

### Considerations

1. This should be done in a single CLI command.
2. The machine is a x86 running Ubuntu 24.04.

# Solution

## Requirements

- awk and jq have already installed

## Solution 1: One CLI command

```bash
jq -r 'select(.symbol=="TSLA") | .order_id' ./transaction-log.txt | xargs -I {} sh -c 'echo "Fetching data for Order ID: {}" && curl -s "https://example.com/api/{}" && echo ""' > ./output.txt
```

## Extend bash script - Solution 1: Using awk

```bash
#!/bin/sh

# Define input and output files
INPUT_FILE="./transaction-log.txt"
OUTPUT_FILE="./output.txt"

# Extract order IDs where the symbol is "TSLA" and save to a temporary file
awk -F ' ' '{if ($4 ~ /TSLA/) print $2}' "$INPUT_FILE" | tr -d '",' > order_id.txt

# Clear output file before writing new results
> "$OUTPUT_FILE"

# Loop through each extracted order ID and send an HTTP GET request
while read -r fn; do
    # Fetch data from the API for the given order ID and append the response to output.txt
    curl -X GET "http://example.com/api/:$fn" >> "$OUTPUT_FILE"
done < order_id.txt

echo "Data fetching complete. Results saved to $OUTPUT_FILE"
```

## Extend bash script - Solution 2: Using jq command

```bash
#!/bin/bash

# Define input and output files
INPUT_FILE="./transaction-log.txt"
OUTPUT_FILE="./output.txt"

# Clear output file if it exists
> "$OUTPUT_FILE"

# Extract order IDs for TSLA sell orders and fetch data from API
grep '"symbol": "TSLA"' "$INPUT_FILE" | jq -r '.order_id' | while read -r order_id; do
    echo "Fetching data for Order ID: $order_id"
    curl -s "https://example.com/api/$order_id" >> "$OUTPUT_FILE"
    echo "" >> "$OUTPUT_FILE"  # Add newline for readability
done

echo "Data fetching complete. Results saved to $OUTPUT_FILE"
```

## Output

```html
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>

```
