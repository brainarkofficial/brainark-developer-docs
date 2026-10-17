# brainark-developer-docs

Developer guides and API reference for BrainArk

## Never commit secrets
Private keys, seed/recovery phrases, `.env` files, keystores and API secrets are blocked by
`.gitignore`, a pre-commit hook and a secret scan on every push. Enable the hook once per clone:

```
git config core.hooksPath .githooks
```

All production keys live only in the encrypted server vault.
