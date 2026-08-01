# Python Coaching Notes

Compact cheat sheet for concepts covered during coaching.

## Running The Lab

- `cd "G:\AI Engineer MS Learn\mslearn-ai-language\Labfiles\02-language-agent\Python\text-agent"`: move PowerShell into the folder containing `text-agent.py`, `.env`, and `requirements.txt`.
- `python text-agent.py`: run the script from the current folder.
- Use quotes around paths with spaces, such as `"AI Engineer MS Learn"`.
- In VS Code, **Open in Integrated Terminal** opens a terminal already located in the folder you clicked.
- Save before running: `Ctrl + S`. PowerShell runs the saved file on disk, not unsaved editor changes.
- `cls`: Windows terminal command to clear the visible terminal output.

## Imports And Environment

- `from dotenv import load_dotenv`: import the function that loads `.env` values.
- `import os`: import Python's operating system module.
- `load_dotenv()`: load variables from `.env` into the Python process.
- `os.getenv("FOUNDRY_ENDPOINT")`: read an environment variable; returns `None` if missing.
- `os.environ["AGENT_NAME"]`: read an environment variable; raises an error if missing.
- `.env` values are strings by default.
- `int(os.getenv("MAX_RETRIES"))`: convert a string env value to an integer.
- `float(os.getenv("TEMPERATURE"))`: convert a string env value to a decimal number.
- `os.getenv("DEBUG") == "true"`: convert a string env value to a Boolean-style check.

## Azure Foundry Endpoints And Auth

- Foundry can show more than one endpoint for the same deployed model.
- `https://<resource>.openai.azure.com`: Azure OpenAI-compatible runtime endpoint.
- `https://<resource>.services.ai.azure.com`: Azure AI Foundry / Azure AI Services runtime endpoint.
- A Foundry project endpoint may include `/api/projects/<project-name>`.
- Some SDK clients need the resource endpoint only, up to the `.com` domain.
- For `TextAnalyticsClient(...)`, use the Azure AI Services resource endpoint:

```env
FOUNDRY_ENDPOINT=https://<resource-name>.services.ai.azure.com
```

- Do not include the project suffix for `TextAnalyticsClient(...)`:

```text
/api/projects/<project-name>
```

- The Python client must match the endpoint style.
- `AzureOpenAI(...)` from the `openai` package expects the Azure OpenAI endpoint style.
- For `AzureOpenAI(...)`, use the root endpoint, not `/openai/v1`:

```env
MODEL_ENDPOINT=https://<resource>.openai.azure.com
MODEL_NAME=<deployment-name>
```

- `MODEL_NAME` should usually be the deployment name, not just the base model name.
- A token scope is the audience the login token is valid for. It does not need to look like the endpoint URL.
- For Azure OpenAI model inference with Microsoft Entra ID, use the Cognitive Services scope:

```python
token_provider = get_bearer_token_provider(
    DefaultAzureCredential(),
    "https://cognitiveservices.azure.com/.default"
)
```

- `https://ai.azure.com/.default` appears in newer Foundry Models examples that use the generic `OpenAI` client with a `/openai/v1/` base URL.
- `https://cognitiveservices.azure.com/.default` is commonly used by Azure OpenAI examples that use `AzureOpenAI(...)`.
- The Microsoft docs can look inconsistent here, so match all three pieces from the same sample: client class, endpoint shape, and token scope.
- A `401 PermissionDenied` can mean the signed-in identity lacks the right role, the endpoint/client pair is wrong, or the token was requested for the wrong audience.
- `Principal does not have access to API/Operation` means Azure knows who the user is, but that identity lacks permission for the requested API operation.
- For Language/Text Analytics calls with `DefaultAzureCredential()`, the signed-in identity usually needs the `Cognitive Services User` role on the Azure AI Services / Foundry resource.
- If the role was just added, run `az logout` and `az login` to refresh the local token, then retry after a few minutes if needed.
- If local file contents print before the error, file reading is working; the failure is probably at the first Azure API call, such as `ai_client.detect_language(...)`.
- Exact short phrases seen in Microsoft docs:
- "`Cognitive Services User role`"
- "`https://<resource>.openai.azure.com/openai/v1/`"
- "`https://ai.azure.com/.default`"
- "`https://cognitiveservices.azure.com`"
- Source checked: `https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/configure-entra-id`

## SDK Versions And Docs Filters

- A Python script depends on installed SDK packages, usually listed in `requirements.txt`.
- For example, `azure-ai-textanalytics==5.3.0` means the script needs the Azure AI Text Analytics Python SDK, version `5.3.0`.
- The SDK package controls which Python imports and client classes are available.
- Before copying Microsoft Learn code, check that the docs page matches the SDK/package version used by the script.
- In Microsoft Learn URLs, `view=azure-python` usually means stable Python SDK docs.
- In Microsoft Learn URLs, `view=azure-python-preview` means preview/beta Python SDK docs.
- The Microsoft Learn version dropdown can default to a latest/preview view, so do not rely only on the page title or search result.
- For `azure-ai-textanalytics` stable `5.3.x`, use `TextAnalyticsClient` examples such as `recognize_entities(...)`, `recognize_pii_entities(...)`, and `detect_language(...)`.
- For `azure-ai-textanalytics` preview `6.0.0b*`, examples may use `TextAnalysisClient` and `analyze_text(...)`.
- Service API versions, such as `2023-04-01`, are not the same thing as Python SDK versions, such as `5.3.0`.
- Match all three pieces before deciding an approach: `requirements.txt` package version, Microsoft Learn docs view/version, and the client class used in the sample.

## Service SDKs Vs Model Endpoints

- An Azure AI / Foundry endpoint is the remote service address.
- The Python client decides which API surface you are using.
- `TextAnalyticsClient(...)` uses built-in Azure AI Language operations.
- Methods such as `detect_language(...)`, `recognize_entities(...)`, and `recognize_pii_entities(...)` are service SDK methods, not separate resources shown individually in Foundry.
- These methods are predefined operations: you send text and get structured results back.
- `AzureOpenAI(...)` uses deployed model endpoints.
- Model calls are prompt- or file-driven: you send text/audio and the model generates text, speech, transcription, or another response.
- `AIProjectClient(...).get_openai_client()` can call Foundry project features such as agents.
- An agent call can use a model plus instructions, tools, and orchestration configured in Foundry.

Useful mental model:

```text
Service SDK method -> fixed API operation
Model endpoint     -> deployed model inference
Agent call         -> model plus configured instructions/tools
```

Example:

```python
ai_client.recognize_pii_entities([text])
```

- Calls a purpose-built Language API for PII detection.

```python
client.audio.transcriptions.create(model=model_deployment, file=audio_file)
```

- Calls a deployed model to transcribe audio.

## Functions And Script Entry

- `def main():`: define a function named `main`; empty `()` means it takes no direct inputs.
- `main()`: call/run the `main` function.
- Defining a function does not run it. Calling it runs it.
- `if __name__ == "__main__":`: true when the file is run directly.
- `if __name__ == "__main__":` should be at the far left, outside `main()`.
- The `main()` under it should be indented 4 spaces.

```python
def main():
    print("Inside main")


if __name__ == "__main__":
    main()
```

## Indentation

- Python uses indentation to show ownership.
- `0 spaces`: top level of the file.
- `4 spaces`: inside a function, `if`, `try`, etc.
- `8 spaces`: inside something that is already inside something else.
- `try:` and `except` line up because they are paired.

```python
try:
    risky_step()
except Exception as ex:
    print(ex)
```

## Terminal Clearing

- `os.system(...)`: run a terminal command from Python.
- `os.name == "nt"`: checks whether Python is running on Windows.
- `'cls' if os.name == 'nt' else 'clear'`: choose `cls` on Windows, otherwise `clear`.

```python
os.system('cls' if os.name == 'nt' else 'clear')
```

## Input And Print

- `print("Hello")`: show text on screen.
- `input("User prompt: ")`: show a prompt, wait for user text, and return what they typed.
- `prompt = input("User prompt: ")`: store the user's typed text in `prompt`.
- Debug `print(...)` lines can show which step a script has reached.

```python
color = input("What is your favorite colour? ")
print(f"Your favorite colour is {color}")
```

## Strings

- `"hello"` and `'hello'`: normal strings.
- `f"Hello, {name}"`: f-string; inserts variable values.
- `r"C:\Users\Asha\new"`: raw string; useful for Windows paths and regex.
- `"""multi-line text"""`: triple-quoted string; can span lines.
- `\n`: newline character.

```python
print(f"{agent_name}: {response.output_text}")
```

- Without the `f`, `{agent_name}` would print as literal text.

## Debugging Slow Steps

- A script that looks stuck may just be waiting on a slow step.
- API calls can take time because Python sends data to a remote service and waits for a response.
- Audio transcription can be slow because the file must be read, sent, processed, and returned as text.
- Add checkpoint prints before and after important actions to find where the wait happens.
- Temporarily comment out terminal clearing while debugging so earlier messages are not erased.

```python
print("Sending transcription request...")
with open(file_path, "rb") as audio_file:
    transcription = client.audio.transcriptions.create(
        model=model_deployment,
        file=audio_file,
        response_format="text"
    )

print("Request finished")
print(transcription)
```

## Brackets

- `()` parentheses: call functions/methods, group expressions, or create tuples.
- `[]` square brackets: Python list; like a JSON array.
- `{}` curly braces: Python dictionary; like a JSON object.

```text
JSON [] array  -> Python [] list
JSON {} object -> Python {} dictionary
```

```python
input=[{"role": "user", "content": prompt}]
```

- Outer `[]`: a list.
- Inner `{}`: a dictionary.
- `"role"` and `"content"`: dictionary keys.
- `"user"` and `prompt`: dictionary values.

## Dictionaries

- Dictionaries store key/value pairs.

```python
message = {"role": "user", "content": "hello"}
print(message["content"])
```

- `message["content"]` gets the value stored under the `"content"` key.

## Objects And Dot Access

- An object is a bundle of related data and actions.
- Dot access means "look inside this object."
- No parentheses: read a value/property.
- Parentheses: call/run a method.

```python
response.output_text
```

- Gets the `output_text` property from the `response` object.

```python
response.model_dump_json(indent=2)
```

- Calls the `model_dump_json()` method on the `response` object.

## Methods Vs Functions

- Function: called directly, such as `print()`, `input()`, `load_dotenv()`.
- Method: called on an object with a dot, such as `project_client.get_openai_client()`.
- A method is a kind of function that belongs to an object.

```python
openai_client = project_client.get_openai_client()
```

- `project_client`: object.
- `get_openai_client()`: method.
- `openai_client`: result returned by the method.

## Azure Clients

- A host/server is the remote service you connect to.
- An endpoint is the service address.
- A client is the local Python object that talks to the service.
- A credential proves who you are.

```python
project_client = AIProjectClient(
    endpoint=foundry_endpoint,
    credential=DefaultAzureCredential(),
)
```

- `endpoint=foundry_endpoint`: where to connect.
- `credential=DefaultAzureCredential()`: how to authenticate.
- `DefaultAzureCredential()` can use Azure CLI login, VS Code login, managed identity, or environment credentials.

## Foundry Agent Calls

```python
response = openai_client.responses.create(
    input=[{"role": "user", "content": prompt}],
    extra_body={"agent_reference": {"name": agent_name, "type": "agent_reference"}},
)
```

- `response`: stores the agent's reply object.
- `openai_client.responses.create(...)`: ask the client to create a response.
- `input=[...]`: list of messages to send.
- `"role": "user"`: says the message is from the user.
- `"content": prompt`: sends what the user typed.
- `extra_body={...}`: Azure-specific extra data.
- `agent_reference`: tells Foundry which agent to use.
- The script calls the agent named in `.env`, such as `AGENT_NAME=Text-Analysis-Agent`.
- The script does not automatically call "latest"; it calls the exact agent name you configure.

## Response Output

```python
print(f"{agent_name}: {response.output_text}")
```

- Prints only the readable answer text from the response.

```python
print(f"\nResponse Details: {response.model_dump_json(indent=2)}")
```

- Does not create a variable.
- Prints a blank line, the label `Response Details:`, and the full response object as formatted JSON.
- `indent=2`: format JSON with 2 spaces per indentation level.
- Useful for seeing tool calls and response metadata.

## Inspecting Objects

- `print(type(response))`: show what kind of object `response` is.
- `print(dir(response))`: list many available properties and methods.
- `print(response.model_dump_json(indent=2))`: inspect the full response data.
- Use docs, VS Code autocomplete, and response dumps to discover available fields.

## Exceptions And Errors

```python
try:
    result = 10 / 0
except Exception as ex:
    print(ex)
```

- `try:` runs code that might fail.
- `except Exception as ex:` catches the error and stores it in `ex`.
- `print(ex)` prints the error message.
- `403 Forbidden`: Azure understood the request, but the signed-in identity lacks permission.
- `400 tool_user_error`: the agent/tool call failed, often because the agent selected a tool that errored or the request did not match the tool setup.

## Requirements

- `requirements.txt`: lists external packages needed by the script.
- `pip list`: shows packages installed in the active environment.
- `(.venv)` in the terminal prompt means the virtual environment is active.
- A virtual environment is an isolated Python workspace for one project.
- This repo's virtual environment is at:

```text
G:\AI Engineer MS Learn\mslearn-ai-language\.venv
```

- Turn on the virtual environment from the repo root:

```powershell
.\.venv\Scripts\Activate.ps1
```

- Turn on the virtual environment from anywhere:

```powershell
& "G:\AI Engineer MS Learn\mslearn-ai-language\.venv\Scripts\Activate.ps1"
```

- Turn off the virtual environment:

```powershell
deactivate
```

- Closing the terminal also turns it off for that terminal session.
- When `(.venv)` is active, `pip install -r requirements.txt` installs packages into `.venv`, not into the current lab folder.
- The current folder only decides where PowerShell looks for `requirements.txt`.
- Installed packages usually go under:

```text
G:\AI Engineer MS Learn\mslearn-ai-language\.venv\Lib\site-packages
```

- Check where a package is installed:

```powershell
python -m pip show azure-ai-textanalytics
```

- Look for the `Location:` line.

```text
python-dotenv
azure-identity
azure-ai-projects==2.0.0b4
```

## Git And GitHub

- Git tracks file changes in your local repository.
- GitHub stores a remote copy of a repository online.
- `git status`: show the current branch and changed files.
- `git status --short`: show the same status in compact form.
- `git ls-files`: list files Git is already tracking.
- `git branch --show-current`: show the current local branch.
- `git remote -v`: show which remote repo `fetch` and `push` use.
- `git add file`: stage the current saved version of a file for the next commit.
- `git diff --cached`: show exactly what is staged for the next commit.
- `git commit`: save a snapshot locally on the current branch.
- `git commit -m "message"`: commit staged changes using the quoted text as the commit message.
- `git push`: send local commits to a remote repo such as GitHub.
- A commit does not automatically create a pull request.
- A pull request is created after pushing a branch and asking to merge it into another branch, often `main`.

Git workflow:

```text
Working folder -> staging area -> commit -> push
```

- Working folder: where files are edited.
- Staging area: the selected changes for the next commit.
- Commit: a local saved snapshot of the staged changes.
- Push: sends local commits to a remote repo.
- If a file is edited after staging, run `git add file` again to stage the latest saved version.

Short status examples:

```text
 M .gitignore
D  Labfiles/02-language-agent/Python/text-agent/.env
?? AGENTS.md
```

- ` M file`: modified in the working folder, not staged.
- `M  file`: modified and staged for commit.
- `D  file`: deleted or removed from Git tracking and staged.
- `?? file`: untracked; Git sees it but is not tracking it yet.
- The left status column is the staging area.
- The right status column is the working folder.

Useful distinction:

```text
git ls-files
```

lists files Git is tracking.

```powershell
dir
```

lists files on disk.

## Gitignore And Secrets

- `.gitignore` tells Git which untracked files to ignore.
- `.gitignore` does not stop Git tracking a file that is already tracked.
- `.venv/` ignores the local Python virtual environment folder.
- `.env` ignores files named exactly `.env`.
- `*.env` ignores files ending in `.env`, such as `dev.env` or `local.env`.

Recommended ignore rules for local Python secrets and environments:

```gitignore
.venv/
.env
*.env
```

If a tracked `.env` file now contains real secrets, do not commit those secret values.

To stop tracking a `.env` file but keep it on disk:

```powershell
git rm --cached Labfiles/02-language-agent/Python/text-agent/.env
```

- `git rm`: remove from Git.
- `--cached`: remove only from Git's tracking/index.
- The local file should remain on disk.

Why not commit `.env` if commits are local?

- A local commit can be pushed later by mistake.
- Secrets committed once remain in Git history, even if removed in a later commit.
- Cleaning secrets from Git history is harder than avoiding the commit.
- Local repos may be backed up, copied, shared, zipped, or inspected by tools.
- A safe pattern is to keep real secrets in `.env` and commit a fake template such as `.env.example`.

Example `.env.example`:

```env
FOUNDRY_ENDPOINT=your_project_endpoint
AGENT_NAME=your_agent_name
```

Example local commit made during this repo setup:

```powershell
git commit -m "Ignore local environment files, add coaching notes and coaching agent files"
```

Result:

```text
[main c3999c7] Ignore local environment files, add coaching notes and coaching agent files
12 files changed
```

- `main`: the branch the commit was created on.
- `c3999c7`: the short commit ID.
- The commit removed tracked lab `.env` files from Git, added ignore rules, and added local notes.
- The `.env` files can still exist locally on disk while being ignored by Git.

## Paths In Python

```python
from pathlib import Path
```

- Imports Python's modern path helper.

```python
Path(__file__).parent
```

- Gets the folder containing the current Python file.

```python
Path(__file__).parent / ".env"
```

- Builds a path to a `.env` file beside the current Python file.

## File Encoding

- Text files are stored as bytes, and encoding tells Python how to turn those bytes into readable characters.
- `encoding="utf8"` tells Python to read the file as UTF-8 text.
- Most modern text files use UTF-8, so it is usually the first encoding to try.
- If Python uses the wrong encoding, you may see strange characters or get a `UnicodeDecodeError`.
- VS Code shows the current file encoding in the bottom-right status bar, such as `UTF-8`, `UTF-8 with BOM`, or `Windows 1252`.
- Clicking the encoding label in VS Code gives options like `Reopen with Encoding` and `Save with Encoding`.
- If VS Code shows `Windows 1252`, try `encoding="cp1252"` in Python.
- Use text mode such as `"r"` for readable files like `.txt`, `.csv`, `.json`, `.md`, `.html`, `.py`, and `.xml`.
- Use binary mode such as `"rb"` for files that must be read as raw bytes, such as `.mp3`, `.png`, `.jpg`, `.pdf`, `.zip`, `.xls`, and `.xlsx`.

```python
text = open(file_path, encoding="utf8").read()
```
