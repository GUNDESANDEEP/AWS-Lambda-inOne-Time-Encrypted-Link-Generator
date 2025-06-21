# Simulate the Lambda function behavior in a Colab-like environment (without actual AWS integration).
# We'll mock DynamoDB using a dictionary to demonstrate storing, reading, and deleting a secure link.

import json
import uuid
import time

# Mock database (simulates DynamoDB)
mock_db = {}

# Simulate POST request to create secure link
def create_secure_link(message, type_="text", expires_in=10):
    link_id = str(uuid.uuid4())
    ttl = int(time.time()) + (expires_in * 60)

    mock_db[link_id] = {
        'message': message,
        'type': type_,
        'expiresAt': ttl
    }

    return {
        "statusCode": 200,
        "body": json.dumps({
            "url": f"https://mock-api.com/secure-link?id={link_id}"
        })
    }

# Simulate GET request to read and delete secure link
def read_secure_link(link_id):
    if link_id not in mock_db or mock_db[link_id]['expiresAt'] < time.time():
        return {
            "statusCode": 404,
            "body": json.dumps({"error": "Link expired or not found"})
        }

    data = mock_db[link_id]
    del mock_db[link_id]  # Self-destruct after reading

    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": data['message'],
            "type": data['type']
        })
    }

# Example Usage
# Step 1: Create a secure link
create_response = create_secure_link("This is a secret message", expires_in=1)
create_response_json = json.loads(create_response["body"])
secure_link_id = create_response_json["url"].split('=')[-1]

# Step 2: Read the secure link
read_response = read_secure_link(secure_link_id)

# Step 3: Try to read again (should fail as it's one-time)
read_again_response = read_secure_link(secure_link_id)

create_response, read_response, read_again_response
# AWS-Lambda-inOne-Time-Encrypted-Link-Generator 
