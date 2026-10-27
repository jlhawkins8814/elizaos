# Crypto Queen local runtime

This repository includes a schema-valid `character.json` for Crypto Queen and a safe local runtime template at `config/crypto-queen.local.example.json`.

## What this profile enables

- Local elizaOS runtime
- Crypto Queen as the root sandbox character
- The canonical `@elizaos/plugin-wallet` surface explicitly disabled
- Automatic wallet-plugin selection and per-agent wallet bootstrap disabled
- No wallet private key, seed phrase, API token, or RPC credential stored in Git

This is a character/runtime starter, not an analytics-only wallet mode. Wallet analytics and transaction tools are unavailable until the wallet plugin is deliberately enabled. `ELIZA_AGENT_WALLET_AUTO_ENABLE=0` suppresses automatic plugin selection; it does not restrict an explicitly enabled plugin. `ELIZA_DISABLE_AGENT_WALLET_BOOTSTRAP=1` also suppresses the runtime's separate per-agent wallet creation path.

Enabling `plugins.entries.wallet.enabled=true` loads `EVMService`, whose wallet initialization can generate and persist an `EVM_PRIVATE_KEY` if none exists. Do not enable the full plugin expecting a signing-key-free first launch. The current plugin has no analytics-only startup mode.

## First local run

The runtime's durable configuration normally lives under the private eliza state directory, not in the repository. On Linux/Termux the default is `~/.local/state/eliza/eliza.json`. Use the runtime resolver below so `ELIZA_STATE_DIR`, `XDG_STATE_HOME`, `ELIZA_CONFIG_PATH`, and namespace overrides are honored. Do not overwrite an existing configuration.

From the repository root:

```bash
config_path="$(bun --conditions=eliza-source -e 'import path from "node:path"; import { ensurePrivateDir, resolveConfigPath } from "./packages/agent/src/config/paths.ts"; const configPath = resolveConfigPath(); ensurePrivateDir(path.dirname(configPath)); process.stdout.write(configPath);')" || exit 1
if [ -e "$config_path" ]; then
  echo "Existing $config_path found. Leave it in place and merge the wallet disable and both env flags through Settings or by hand before launch."
else
  (umask 077; cp -n config/crypto-queen.local.example.json "$config_path")
fi
```

Run this after installing the repository dependencies. If a configuration already exists, merge the starter's safety settings before running the separate launch command below; the copy snippet preserves that file.

```bash
ELIZA_CHARACTER_PATH="$PWD/character.json" bun run dev:local
```

The root `character.json` is also discovered automatically in ordinary local/sandbox startup, but `ELIZA_CHARACTER_PATH` makes the intended character explicit.

## Read-only Solana setup

The starter profile does not require a signing key. The capabilities below belong to the full wallet plugin and are unavailable while it is disabled. Opting into that plugin has the EVM key-generation behavior described above. Add an RPC URL through the runtime's protected Settings/Vault path when Solana RPC-backed features are needed. Do not commit it to this repository.

Useful read-only wallet capabilities already provided by elizaOS include:

- `token_info`: token/market lookup through the wallet analytics services
- `search_address`: public-address portfolio lookup when the configured provider supports it
- DexScreener token/pair lookup
- Birdeye market/portfolio/trending data when configured
- Solana RPC-backed inspection when `SOLANA_RPC_URL` is configured

## Transaction modes

After deliberately enabling the wallet plugin, use these modes with their actual guarantees:

1. **Read only**: research tokens, addresses, liquidity, authorities, and market data.
2. **Prepare**: for non-bridge actions, return request fields and handler metadata only. This does not invoke the chain handler, construct unsigned transaction bytes, obtain a quote/route, or validate on-chain execution. Bridge prepare/dry-run delegates to its chain handler and may obtain route data; it is not a general transaction-construction guarantee.
3. **Simulate**: supported for Solana `swap` and `pump_fun_buy`; build the real unsigned transaction and run RPC simulation. Configure `SOLANA_RPC_URL` and a valid base58 wallet address in `SOLANA_PUBLIC_KEY` (or `WALLET_PUBLIC_KEY`) through protected Settings/Vault. A backend-provided Solana address also works. Use the intended wallet's public address; no private key or seed phrase is needed by the simulation handler. Missing or invalid public keys fail rather than creating one. The full plugin's startup can still generate an EVM signing key independently of simulation.
4. **Execute**: only after the wallet plugin's separate human confirmation turn.

Never put a seed phrase or raw private key in chat, this file, source control, or a prompt.

## Mobile limitation

The stock Android/iOS agent runtime currently filters its plugin load set to the mobile-safe allow-list. `@elizaos/plugin-wallet` is not in that stock-mobile allow-list, so this local profile should be validated in the Node/Termux or desktop runtime first. Mobile wallet/NovaMint integration is a separate implementation milestone and must not be represented as working until its bundle/runtime path is explicitly added and tested.
