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
{"live_browser_url":"https://www.browserbase.com/devtools-fullscreen/inspector.html?wss=connect.browserbase.com/debug/365d0f97-e049-4b2c-b5f0-acdc13aac623/devtools/page/9339B1B48D7E43A1989E0C8EE0104DA9?debug=true","session_id":"365d0f97-e049-4b2c-b5f0-acdc13aac623","status":"started","storage_state_path":null}%    
```


set goal, and some initial steps
```
curl -X POST 'http://0.0.0.0:8000/run-agent-initial-steps-stream' \
-H 'Content-Type: application/json' \
-d '{
  "session_id": "365d0f97-e049-4b2c-b5f0-acdc13aac623",
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
  "session_id": "365d0f97-e049-4b2c-b5f0-acdc13aac623",
  "goal": "click on search button",
  "s3_path": "s3://loggia-tests/loggia-test/tests/2/flow.json",
  "storage_state_s3_path": null
}'
data: Agent setup completed

data: [{'tool_call_id': 'call_EMPttlrgylKzhYIll9amzzbC', 'role': 'tool', 'name': 'navigation', 'content': 'The action is: ```click(\'250\')``` - the result is: Based on the screenshot, the search has been performed successfully. Evidence of this includes:\n\n1. **Search Query**: The search bar at the top shows "dining table," indicating that a search has been executed.\n2. **Search Results**: Various products related to "dining table" are displayed below the search bar, which is a typical result of clicking a search button.\n3. **Contextual Filters**: Options like "Round," "Extendable," and "Black" suggest a filtering system often associated with search results.\n\nThese elements confirm that the goal of clicking the search button has been accomplished, as the application is now showing search results for the queried term.'}]

data: Final Response: ModelResponse(id='chatcmpl-AeU76pquQvI2HDoS2DlTrWfuFU8wN', choices=[Choices(finish_reason='stop', index=0, message=Message(content='The search button has been successfully clicked, and the search results for "dining table" are now displayed. Please provide further instructions if needed.', role='assistant', tool_calls=None, function_call=None))], created=1734212240, model='gpt-4o-mini-2024-07-18', object='chat.completion', system_fingerprint='fp_6fc10e10eb', usage=Usage(completion_tokens=30, prompt_tokens=588, total_tokens=618))
```

terminate the browserbase session, whole session completed
```
curl -X POST "http://0.0.0.0:8000/terminate-browserbase?session_id=365d0f97-e049-4b2c-b5f0-acdc13aac623"
{"status":"terminated"}%
```