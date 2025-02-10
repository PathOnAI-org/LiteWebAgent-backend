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
start browserbase session, retrieve live_browser_url for the frontend
```
curl -X POST 'http://0.0.0.0:8000/start-browserbase' \
-H 'Content-Type: application/json' \
-d '{"storage_state_s3_path": null}'
{"live_browser_url":"https://www.browserbase.com/devtools-fullscreen/inspector.html?wss=connect.browserbase.com/debug/f525ba67-c88e-4485-b207-dd9bf188729f/devtools/page/1467F73862C1EB9B7C68C41A4C654BD6?debug=true","session_id":"f525ba67-c88e-4485-b207-dd9bf188729f","status":"started","storage_state_path":null}%   
```


set goal, and some initial steps
```
curl -X POST 'http://0.0.0.0:8000/run-agent-initial-steps-stream' \
-H 'Content-Type: application/json' \
-d '{
  "session_id": "f525ba67-c88e-4485-b207-dd9bf188729f",
  "starting_url": "https://www.google.com",
  "goal": "type dining table in text box",
  "s3_path": "s3://loggia-tests/loggia-test/tests/2/flow.json",
  "storage_state_s3_path": null
}'
data: Connected to browser. Live URL: https://www.browserbase.com/devtools-fullscreen/inspector.html?wss=connect.browserbase.com/debug/365d0f97-e049-4b2c-b5f0-acdc13aac623/devtools/page/9339B1B48D7E43A1989E0C8EE0104DA9?debug=true

data: Agent setup completed

data: [{'tool_call_id': 'call_hY93qWEb3cPEHM74XNaHslWX', 'role': 'tool', 'name': 'navigation', 'content': 'The action is: ```fill(\'113\', \'dining table\')``` - the result is: Yes, the goal is finished. The text "dining table" is correctly typed into the search box, as shown in the screenshot. This matches the original goal of typing "dining table" into the text box.'}]

data: Final Response: ModelResponse(id='chatcmpl-AeU5KgCTUnnbQYIpP3NLJmfjWCeCV', choices=[Choices(finish_reason='stop', index=0, message=Message(content='The task of typing "dining table" into the text box has been completed successfully. Please provide further instructions if needed.', role='assistant', tool_calls=None, function_call=None))], created=1734212130, model='gpt-4o-mini-2024-07-18', object='chat.completion', system_fingerprint='fp_6fc10e10eb', usage=Usage(completion_tokens=26, prompt_tokens=506, total_tokens=532))
```


add some follow-up steps
```
curl -X POST 'http://0.0.0.0:8000/run-agent-followup-steps-stream' \
-H 'Content-Type: application/json' \
-d '{
  "session_id": "f525ba67-c88e-4485-b207-dd9bf188729f",
  "goal": "click on search button",
  "s3_path": "s3://loggia-tests/loggia-test/tests/2/flow.json",
  "storage_state_s3_path": null
}'

data: {"type": "status", "message": "Starting setup..."}

data: {"type": "status", "message": "Storage state loaded"}

data: {"type": "browser", "message": "Browser connected at https://www.browserbase.com/devtools-fullscreen/inspector.html?wss=connect.browserbase.com/debug/f525ba67-c88e-4485-b207-dd9bf188729f/devtools/page/1467F73862C1EB9B7C68C41A4C654BD6?debug=true"}

data: {"type": "status", "message": "Agent setup complete"}

data: {"type": "thinking", "message": "Processing next step..."}

data: {"type": "tool_calls", "message": "Executing 1 actions..."}

data: {"type": "tool_execution", "message": "Executing: navigation"}

data: {"type": "tool_result", "message": {"tool_call_id": "call_UAJ8YtGfkMmIKE3QCZ5XgFnn", "role": "tool", "name": "navigation", "content": "The action is: ```fill('90', 'dining table')``` - the result is: Yes, the goal is finished. The screenshot shows that the text \"dining table\" has been typed into the Google search box, as evidenced by its presence in the text input field. This aligns with the original goal of typing \"dining table\" in the text box."}}

data: {"type": "thinking", "message": "The task of typing \"dining table\" in the text box has been completed. Please provide further instructions if needed."}

data: {"type": "action", "message": "No actions needed"}

data: {"type": "complete", "message": "Task completed", "response": [{"finish_reason": "stop", "index": 0, "message": {"content": "The task of typing \"dining table\" in the text box has been completed. Please provide further instructions if needed.", "role": "assistant", "tool_calls": null, "function_call": null}}]}
```

terminate the browserbase session, whole session completed
```
curl -X POST "http://0.0.0.0:8000/terminate-browserbase?session_id=f525ba67-c88e-4485-b207-dd9bf188729f"
{"status":"terminated"}%
```