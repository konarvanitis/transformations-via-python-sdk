# transformations-via-python-sdk
The materials for the course CDF Transformations via Python SDK.

It contains a hands-on notebook (`OIDC_Transformations_via_Python_SDK.ipynb`) showcasing Cognite Data Fusion Transformations with the Cognite Python SDK, and the sample weather data it uses (`data/`).

https://cognite-docs.readthedocs-hosted.com/projects/cognite-sdk-python/en/latest/

## Getting Started

### Prerequisites

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

### 1. Clone the repository

```bash
git clone https://github.com/cognitedata/transformations-via-python-sdk
```

### 2. Install dependencies

We recommend using uv to manage your Python virtual environment:

```bash
uv sync
```

This installs the dependencies defined in `pyproject.toml` and creates a virtual environment in the project folder.

### 3. Run the notebook

Open the repo in your IDE (e.g., VS Code) and open `OIDC_Transformations_via_Python_SDK.ipynb`.

> **Note:** You may need to select the uv virtual environment (`.venv`) as your kernel.

The notebook logs in interactively through your browser. You also need the client secret from the course lesson, which Transformations use to run with their own OIDC credentials. Create a `.env` file in the repository root (it is git-ignored, so it never gets committed) and add it there:

```
CLIENT_SECRET=<the secret from the course lesson>
```

The notebook loads it with `python-dotenv`.

#### Note: MSAL is no longer installed explicitly

Earlier versions of the notebook ran `%pip install msal` and used [MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python) (Microsoft Authentication Library for Python) directly: it ran the device-code login (enter a code at https://microsoft.com/devicelogin) and the resulting access token was passed to the Cognite client as a fixed `Token`, which expired after about an hour.

The notebook now uses `OAuthInteractive.default_for_entra_id` from `cognite-sdk` instead. MSAL is still used under the hood, since `cognite-sdk` depends on it and `uv sync` installs it transitively. That is why it is not listed in `pyproject.toml` and not imported in the notebook. What changes for you:

- **Login:** a browser window opens and redirects back to `http://localhost:53000`, instead of a device code. The Azure app registration needs this URL registered as a redirect URI of type "Mobile and desktop applications".
- **Token refresh:** the SDK refreshes the token automatically.

If you need the old device-code flow (for example, the redirect URI is not registered), add `msal` as a dependency again (`uv add msal`) and restore the `PublicClientApplication` login from the git history of the notebook.

### 4. Set up clean notebook diffs (one-time, per clone)

Jupyter stamps your local kernel name and Python version into each notebook's metadata every time you run it, which shows up as noisy, unrelated diffs in `git status`/`git diff`. Run this once after cloning to strip that noise before it ever reaches git:

```bash
uv run nbstripout --install --attributes .gitattributes
git config filter.nbstripout.extrakeys "metadata.kernelspec metadata.language_info.version metadata.vscode"
```

This registers a git filter that strips outputs, execution counts, and the kernel/version metadata from notebooks whenever git reads or diffs them — your local `.ipynb` files on disk are untouched, so notebooks still run and show outputs normally in your editor.

## Alternative: pip installation

If you prefer not to use uv, you can install the dependencies directly with pip:

```bash
pip install "cognite-sdk[pandas]" notebook
```

## Troubleshooting

### WSL: interactive login fails with `gio: ... Operation not supported`

On WSL, Python's interactive OAuth login (the authentication cell at the top of the notebook) tries to open your default browser and can fail with an error like:

```
gio: https://login.microsoftonline.com/...: Operation not supported
```

This happens because WSL has no browser handler registered for the login URL. Copy the URL from the error message and paste it directly into your Windows browser — the login redirect to `localhost` will reach the notebook correctly thanks to WSL2's automatic localhost port forwarding.
