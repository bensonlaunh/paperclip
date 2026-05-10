# `@paperclipai/plugin-exe-dev`

Published exe.dev sandbox provider plugin for Paperclip.

This package lives in the Paperclip monorepo, but it is intentionally excluded from the root `pnpm` workspace and shaped to publish and install like a standalone npm package. That lets operators install it from the Plugins page by package name without introducing root lockfile churn.

## Install

From a Paperclip instance, install:

```text
@paperclipai/plugin-exe-dev
```

## Configuration

Configure exe.dev from `Company Settings -> Environments`, not from the plugin's instance settings page.

- Put the exe.dev API token on the sandbox environment itself.
- When you save an environment, Paperclip stores pasted API keys as company secrets.
- `EXE_API_KEY` remains an optional host-level fallback when an environment omits the key.
- The current implementation provisions VMs through exe.dev's HTTPS API and runs commands through direct SSH to the created VM.

Operational notes:

- The API token must allow the lifecycle commands the provider uses: `new`, `ls`, and `rm`. `restart` is only needed if you extend the provider to restart retained VMs.
- The Paperclip host must have SSH access to the resulting `*.exe.xyz` VMs. You can rely on the host's normal SSH config/agent or set `sshIdentityFile` in the environment config.
- Reusable leases keep the VM alive between runs. exe.dev does not expose a documented "stop and later resume" command in the public CLI docs, so `reuseLease: true` means "retain the VM" rather than "suspend it."
- The provisioning path uses `https://exe.dev/exec`, which exe.dev documents as a command-style HTTPS API with a 30-second request timeout. Typical `new` calls are expected to fit inside that limit; command execution does not use `/exec`.

## Local development

```bash
cd packages/plugins/sandbox-providers/exe-dev
pnpm install --ignore-workspace --no-lockfile
pnpm build
pnpm test
pnpm typecheck
```

These commands assume the repo root has already been installed once so the local `@paperclipai/plugin-sdk` workspace package is available to the compiler during development.

## Package layout

- `src/manifest.ts` declares the sandbox-provider driver metadata
- `src/plugin.ts` implements the environment lifecycle hooks
- `paperclipPlugin.manifest` and `paperclipPlugin.worker` point the host at the built plugin entrypoints in `dist/`
