# Astrore Extension Publishing

Astrore uses an HTTPS JSON registry to discover third-party extensions. Extension packages are
`tar.gz` archives whose root contains `manifest.json`.

## Runtime choices

- `wasi`: preferred for third-party extensions. It is reserved for Astrore's sandboxed WASM runtime.
- `external-mcp`: compatibility mode for Python, Node.js, Rust binaries, and other MCP servers.
  These extensions run with the current user's operating-system permissions and are always shown
  as high risk.

External MCP extensions are never started silently. Users must confirm their declared permissions
before every start.

## Manifest

```json
{
  "id": "developer-extension",
  "name": "Developer Extension",
  "version": "1.0.0",
  "description": "What the extension does",
  "author": "Developer",
  "icon": "Puzzle",
  "runtime": "external-mcp",
  "homepage": "https://github.com/developer/extension",
  "entry": {
    "command": "python",
    "args": ["server.py"],
    "env": {}
  },
  "permissions": ["server.status.read"]
}
```

IDs may only contain ASCII letters, numbers, `-`, and `_`. The package directory name, manifest ID,
and registry ID must match.

## Permissions

Use narrow permissions. Recommended names include:

- `server.status.read`
- `server.players.read`
- `server.command.send`
- `storage.extension-private`
- `network:api.example.com`

Permissions are declarations shown to users. External MCP processes cannot be fully sandboxed, so
Astrore treats them as high risk even when they declare no permissions. WASI permissions will be
enforced by the sandbox runtime.

## Publish and update

1. Package the extension so `manifest.json` is at the archive root.
2. Publish the immutable `tar.gz` file in a GitHub Release or another HTTPS host.
3. Calculate its SHA-256 and byte size.
4. Add or update its entry in an Astrore-compatible registry.
5. Increase the semantic version for every release.

Registry format:

```json
{
  "schemaVersion": 1,
  "extensions": [
    {
      "id": "developer-extension",
      "name": "Developer Extension",
      "version": "1.1.0",
      "description": "What the extension does",
      "author": "Developer",
      "runtime": "wasi",
      "downloadUrl": "https://example.com/developer-extension-1.1.0.tar.gz",
      "sha256": "64-character-sha256",
      "size": 12345,
      "homepage": "https://github.com/developer/extension",
      "permissions": ["server.status.read"],
      "verified": false
    }
  ]
}
```

Astrore compares the registry version with the installed manifest. Updates use the same install
pipeline: HTTPS download, 25 MB size limit, SHA-256 verification, safe archive path validation,
manifest validation, old-version backup, then atomic replacement.

Third-party communities can host their own registry. Users paste its HTTPS URL into the online
extension store. The official registry is maintained through reviewed pull requests.

For the official registry, contributors submit a pull request changing `registry/index.json`.
`npm run registry:check` validates IDs, semantic versions, HTTPS URLs, SHA-256 values, permissions,
duplicate entries, and the 25 MB package limit. Only maintainers may mark an entry as `verified`.
