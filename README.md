# LiteWebAgent backend

## 1. QuickStart
```
python3.11 -m venv venv
. venv/bin/activate
pip3.11 install -r requirements.txt
```
Then, a required step is to setup playwright by running
```
playwright install chromium
```
test your playwright installation
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)  # Set headless=True if you prefer no UI
    page = browser.new_page()
    page.goto('https://example.com')
    print(page.title())
    browser.close()

```

Then please create a .env file, and update your API keys:

```bash
cp .env.example .env
```

You are ready to go! Try FunctionCallingAgent on google.com
```
python3.11 api/google_test.py
```

start local fastapi server

```
python3.11 -m api.main
```

## 2. unauthenticated
```
curl -X POST 'http://0.0.0.0:8000/start-browserbase' \
-H 'Content-Type: application/json' \
-d '{"storage_state_s3_path": null}'
{"live_browser_url":"https://www.browserbase.com/devtools-fullscreen/inspector.html?wss=connect.browserbase.com/debug/c2375f88-ac4f-4407-a441-00cbb6a37f6e/devtools/page/00180B32E9F5DB9D3ED33F2376D45860?debug=true","session_id":"c2375f88-ac4f-4407-a441-00cbb6a37f6e","status":"started","storage_state_path":null}%   



curl -X POST 'http://0.0.0.0:8000/connect-browserbase' \
-H 'Content-Type: application/json' \
-d '{
    "session_id": "c2375f88-ac4f-4407-a441-00cbb6a37f6e",
    "storage_state_s3_path": null
}'


curl -X POST 'http://0.0.0.0:8000/run-agent-initial-steps-stream' \
-H 'Content-Type: application/json' \
-d '{
  "session_id": "c2375f88-ac4f-4407-a441-00cbb6a37f6e",
  "starting_url": "https://www.google.com",
  "goal": "type dining table in text box",
  "s3_path": "s3://loggia-tests/loggia-test/tests/2/flow.json",
  "storage_state_s3_path": null
}'

curl -X POST 'http://0.0.0.0:8000/run-agent-followup-steps-stream' \
-H 'Content-Type: application/json' \
-d '{
  "session_id": "c2375f88-ac4f-4407-a441-00cbb6a37f6e",
  "goal": "click on search button",
  "s3_path": "s3://loggia-tests/loggia-test/tests/2/flow.json",
  "storage_state_s3_path": null
}'

curl -X POST "http://0.0.0.0:8000/terminate-browserbase?session_id=c2375f88-ac4f-4407-a441-00cbb6a37f6e"
```