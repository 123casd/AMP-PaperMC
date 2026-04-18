# AMP custom templates for Paper + Velocity

This repository contains two AMP Generic Module templates for PaperMC:

- `amp-paper-generic.*` for a Paper backend server
- `amp-velocity-generic.*` for a Velocity proxy

These templates are intended to run **Paper and Velocity as separate AMP instances**.

## What is required for AMP to see the templates

- `manifest.json` must exist in the repo root.
- The template files should live in the repo root so AMP can resolve the manifests correctly.
- After updating the repo, run **Fetch Latest** in AMP before creating a new instance.
- When testing template changes, prefer a **fresh instance** instead of reusing an older broken one.

## Important implementation notes

### 1) JAR URLs must be set in the instance

These templates use direct download URLs so the installed filenames stay predictable:

- Paper downloads to `server.jar`
- Velocity downloads to `velocity.jar`

You must set these in AMP unless you add defaults in the config manifests:

- `ServerJarURL`
- `VelocityJarURL`

If either field is blank, AMP can throw an invalid request URI error during update.

### 2) Use a flat-root layout

Both templates should run from the instance root:

```ini
App.RootDir=./
App.BaseDirectory=./
```

Do **not** use `./Paper/` or `./Velocity/` unless all update and startup stages are also rewritten to target those subdirectories.

### 3) Linux Java should use an absolute path

The working fix for both templates was to use an absolute Linux Java path:

```ini
App.ExecutableLinux=/usr/bin/java
```

Using a bare `java` caused AMP to try launching a non-existent executable inside the instance directory.

### 4) Velocity config seeding should happen during update

For this template layout, Velocity worked reliably when these files were created in the **update** stage rather than the pre-start stage:

- `forwarding.secret`
- `velocity.toml`

If `velocity.jar` downloads but those two files do not appear, move the file creation steps into `amp-velocity-genericupdates.json`.

### 5) Paper forwarding still needs a manual edit

AMP Generic metaconfig does not natively manage Paper's YAML proxy forwarding settings.

After first boot of a backend Paper server behind Velocity, manually edit Paper's config to enable Velocity modern forwarding and set the shared secret in `config/paper-global.yml`.

### 6) Backend Paper settings behind Velocity

When a Paper instance is running behind Velocity on the same host, the usual backend settings are:

- `BindAddress=127.0.0.1`
- `OnlineMode=false`

Velocity should keep modern forwarding enabled and use the same shared secret as Paper.

## Known working behavior

### Paper

The working Paper template behavior is:

- updates download the selected Paper jar as `server.jar`
- pre-start creates `eula.txt`
- startup runs Java with `-jar server.jar --nogui`
- the server starts from the instance root

### Velocity

The working Velocity template behavior is:

- updates download the selected Velocity jar as `velocity.jar`
- updates create `forwarding.secret`
- updates seed `velocity.toml` if it does not already exist
- startup runs Java with `-jar velocity.jar`
- the proxy starts from the instance root

## Typical flow

1. Put the files in your AMP template repo.
2. Ensure `manifest.json` is present in the repo root.
3. Add the repo in AMP under `Configuration -> Instance Deployment -> Configuration Repositories`.
4. Run **Fetch Latest**.
5. Create one Velocity instance using the Velocity template.
6. Create one or more Paper instances using the Paper template.
7. Set `ServerJarURL` in each Paper instance.
8. Set `VelocityJarURL` in the Velocity instance.
9. Start a Paper backend once so it generates its normal files.
10. Edit Paper's forwarding settings for Velocity in `config/paper-global.yml`.
11. Start Velocity.
12. Point Velocity's backend address at the Paper instance.

## Troubleshooting

### AMP downloads the jar every time but the server does not start

Check:

- `App.RootDir=./`
- `App.BaseDirectory=./`
- `App.ExecutableLinux=/usr/bin/java`

### Paper or Velocity shows no useful server logs

If there is no game/proxy log output at all, check the AMP log first. If AMP says the specified executable does not exist, the Java path is wrong.

### Velocity jar exists but `velocity.toml` does not

That usually means the file-seeding stage did not execute where expected. Move `velocity.toml` and `forwarding.secret` creation into `amp-velocity-genericupdates.json`.
