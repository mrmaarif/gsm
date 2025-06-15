# GitHub Secrets Manager (GSM)

GSM is a modern, production-ready Rust CLI tool for managing GitHub repository secrets securely and efficiently. It supports per-value AES-256-GCM encryption, multi-repo workflows, and seamless integration with the GitHub API.

## Features
- **AES-256-GCM encryption**: Each secret value is individually encrypted with a unique salt and nonce.
- **Multi-repo support**: Manage secrets for multiple repositories and organizations from a single config file.
- **Config file encryption**: Only the `env` values are encrypted, preserving the YAML structure.
- **Modern CLI**: Built with `clap` for a user-friendly command-line experience.
- **Bulk operations**: Encrypt/decrypt all secrets in a folder with a single command.
- **GitHub integration**: Push secrets directly to GitHub repositories using the REST API.
- **Docker support**: Easily containerize and run GSM in CI/CD pipelines.

## Available Commands
```
validate     Validate a configuration file
encrypt      Encrypt a raw config file
decrypt      Decrypt an encrypted config file
encrypt-all  Encrypt all raw config files
decrypt-all  Decrypt all encrypted config files
push         Push secrets to GitHub repositories
help         Print this message or the help of the given subcommand(s)
```

---

# Usage (for users)

## Environment Variables
- `GITHUB_TOKEN`: Your GitHub personal access token
- `ENCRYPTION_KEY`: The master key for encryption/decryption (recommended to set in `.env` file)

## ⚠️ IMPORTANT SECURITY NOTE

**ALWAYS encrypt your config files before committing to source control, especially public repositories!**

- **NEVER** commit raw YAML files containing secrets to GitHub
- **ALWAYS** use `encrypt` or `encrypt-all` commands before pushing to your repository
- Only commit files from the `encrypted/` folder to source control
- Keep raw files in `raw/` folder and add `raw/` to your `.gitignore`
- Use strong, unique `ENCRYPTION_KEY` and store it securely (environment variables, CI/CD secrets, etc.)

```bash
# ✅ GOOD: Encrypt first, then commit
gsm encrypt-all --input ./config
git add config/encrypted/
git commit -m "Add encrypted config files"

# ❌ BAD: Never do this!
git add config/raw/
git commit -m "Add raw config files"  # This exposes your secrets!
```

## Example Config File
```yaml
org: your-github-org
repositories:
  - repo1
  - repo2
env:
  SECRET_KEY: supersecret
  API_TOKEN: abc123
```

## Typical Workflow

### 1. Prepare Config Files
- Place unencrypted YAML files in `your-folder/raw/`.
- Encrypted files will be written to `your-folder/encrypted/`.

### 2. Encrypt All Secrets
```sh
./target/release/gsm encrypt-all --input your-folder
```

### 2a. Encrypt a Single File
```sh
./target/release/gsm encrypt --file your-folder/raw/config.yaml --output your-folder/encrypted/config.yaml
```

### 3. Decrypt All Secrets
```sh
./target/release/gsm decrypt-all --input your-folder
```

### 3a. Decrypt a Single File
```sh
./target/release/gsm decrypt --file your-folder/encrypted/config.yaml --output your-folder/raw/config.yaml
```

### 4. Validate a Config File
```sh
./target/release/gsm validate --file path/to/config.yaml
```

### 5. Push Secrets to GitHub
Set your GitHub token in the environment:
```sh
export GITHUB_TOKEN=ghp_xxx...
```
Then run:
```sh
./target/release/gsm push --file path/to/raw/config.yaml
```

---

# Development (for contributors)

## Prerequisites
- Rust (1.87.0)
- A GitHub personal access token with `repo` and `admin:repo_hook` permissions

## Build
```sh
cargo build --release
```

## Run
```sh
./target/release/gsm --help
```

## Project Structure
```
config/           # Example and template config files
src/              # Rust source code
  cli/            # CLI command implementations
  config.rs       # Config loading and validation
  crypto.rs       # Encryption/decryption logic
  github.rs       # GitHub API integration
  error.rs        # Error types
```

## License
MIT 