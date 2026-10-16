# Example for manual token authorization
Context: Discord developer reference: Example for manual token authorization
[Source](../entry-points/entry-points-and-linking-index.md) / [Owning endpoint, event, object or procedure conditions](overview.md) / [Previous source section](handle-token-response-example-response.md) / [Following source section](working-with-tokens.md)
<a id="source-example-for-manual-token-authorization"></a>

### Example for Manual Token Authorization

 The following Python example demonstrates a manual authorization flow.

```
import requests
import json
import time

API_ENDPOINT = 'https://discord.com/api/v10'
CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
SCOPE = 'sdk.social_layer'

def authorize_device():
  data = {
    'scope': SCOPE
  }
  headers = {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
  r = requests.post('%s/oauth2/device/authorize' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  return r.json()

def exchange_device_code(device_code):
  print("Attempting to exchange device code for access token...")
  data = {
      "grant_type": "urn:ietf:params:oauth:grant-type:device_code",
      "device_code": device_code
    }
  headers = {"Content-Type": "application/x-www-form-urlencoded"}

  r = requests.post('%s/oauth2/token' % API_ENDPOINT, data=data, headers=headers, auth=(CLIENT_ID, CLIENT_SECRET))
  r.raise_for_status()
  response = r.json()
  return r.json()

def poll_token_exchange(device_code, interval, expires_in):
  start_time = time.time()

  while time.time() - start_time < expires_in:
    try:
      print("Checking for authorization...")
      result = exchange_device_code(device_code)
      return result
    except requests.exceptions.HTTPError as e:
      if e.response.status_code == 400:
        # Authorization is pending, wait and try again
        print(f"Waiting {interval} seconds for user to authorize...")
        time.sleep(interval)
      else:
        # Some other error occurred
        raise e

  raise Exception(f"Authorization timed out after {expires_in} seconds")

# Step 1: Get device authorization details
device_auth = authorize_device()

# Step 2: Display authorization details to user
print("\nPlease visit:", device_auth["verification_uri_complete"])

# Step 3: Poll for token exchange
try:
    access_token = poll_token_exchange(
        device_auth["device_code"],
        device_auth["interval"],
        device_auth["expires_in"]
    )
    print("\nSuccess! Access Token:", access_token["access_token"])
except Exception as e:
    print("\nError:", str(e))

# Step 4: Use the access token with discordpp::Client::UpdateToken and discordpp::Client::Connect

```

---

