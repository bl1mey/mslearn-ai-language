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
- `git commit`: save a snapshot locally on the current branch.
- `git push`: send local commits to a remote repo such as GitHub.
- A commit does not automatically create a pull request.
- A pull request is created after pushing a branch and asking to merge it into another branch, often `main`.

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
