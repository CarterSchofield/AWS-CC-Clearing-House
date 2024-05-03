# Documentation for `lambda_function.py`

## Overview

`lambda_function.py` is a Python script designed to be deployed as an AWS Lambda function. It handles the processing of financial transactions, interacting with DynamoDB to retrieve and update data about merchants, banks, and transactions. The function is heavily integrated with AWS services, utilizing the `boto3` library to communicate with DynamoDB. It also includes logic to simulate potential failures in banking operations and to manage credit or debit card transactions based on available balances or credit limits.

## Index

1. [Imports](#imports)
2. [DynamoDB Initialization](#dynamodb-initialization)
3. [Lambda Handler Function](#lambda-handler-function)
   - [Transaction Details Setup](#transaction-details-setup)
   - [Bank API Simulation](#bank-api-simulation)
   - [Processing Transaction](#processing-transaction)
   - [Logging and Response](#logging-and-response)
4. [Error Handling](#error-handling)
5. [Dependencies](#dependencies)
6. [Potential Improvements](#potential-improvements)

## Imports

The script starts by importing necessary Python libraries:

```python
import json
import boto3
from decimal import Decimal
import datetime
import random
```

- `json`: to handle JSON data.
- `boto3`: AWS SDK for Python.
- `Decimal`: for precise decimal arithmetic.
- `datetime`: to work with dates and times.
- `random`: to simulate random bank API failures.

## DynamoDB Initialization

The script initializes DynamoDB tables using the `boto3` library:

```python
dynamodb = boto3.resource('dynamodb')
merchant_table = dynamodb.Table('Merchants')
bank_table = dynamodb.Table('banks')
transaction_table = dynamodb.Table('Transactions')
```

Three tables are used:

- **Merchants**: Stores merchant data.
- **banks**: Maintains bank accounts and their details.
- **Transactions**: Logs all transaction attempts and their results.

## Lambda Handler Function

### Transaction Details Setup

A dictionary named `transaction_details` is initialized with default values including a transaction ID from `context.aws_request_id`, the current timestamp, and an initial status of "Error":

```python
transaction_details = {
    'TransactionID': str(context.aws_request_id),
    'Timestamp': datetime.datetime.now().isoformat(),
    'MerchantName': None,
    'MerchantID': None,
    'LastFourCC': None,
    'Amount': None,
    'Status': "Error"
}
```

### Bank API Simulation

The function simulates a bank API failure 10% of the time using the `random` module:

```python
if random.randint(1, 10) == 1:
    transaction_status = "Bank Not Available"
    transaction_details['Status'] = transaction_status
    transaction_table.put_item(Item=transaction_details)
    return {
        "statusCode": 503,
        "body": json.dumps({"message": transaction_status})
    }
```

### Processing Transaction

The main transaction processing occurs within several nested `if` statements that verify the presence of required data and conditions:

- Checks the presence of merchant data and validates it against the `Merchants` table.
- Verifies card details and checks against the `banks` table.
- Depending on the card type (credit or debit), it checks if the transaction can be approved based on the available credit or balance.

### Logging and Response

Every transaction, successful or not, is logged into the `Transactions` table. The function then returns an HTTP status code and a message indicating the result of the transaction:

```python
transaction_details['Status'] = transaction_status
transaction_table.put_item(Item=transaction_details)
return {
    "statusCode": 200 if transaction_status == "Approved" else 403 if "Declined" in transaction_status else 400,
    "body": json.dumps({"message": transaction_status})
}
```

## Error Handling

The script includes checks at each step of processing to ensure that necessary data is available and valid. Errors result in updating the transaction status and returning appropriate HTTP status codes.

## Dependencies

This script depends on:

- AWS Lambda (runtime environment)
- AWS DynamoDB (database)
- Python libraries: `boto3`, `json`, `decimal`, `datetime`, `random`

## Potential Improvements

- **Enhanced Logging**: Incorporate more detailed logging for debugging and auditing purposes.
- **Configuration Management**: Externalize configuration settings (e.g., table names) to make the code more adaptable.
- **Security Enhancements**: Implement more robust security checks and validations on incoming data.

This detailed documentation provides a comprehensive overview of the `lambda_function.py`, outlining its functionalities, structure, and the AWS services it interacts with.