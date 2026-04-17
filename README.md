# AMP custom templates for Paper + Velocity

This bundle includes two AMP Generic Module templates:

- `amp-paper-generic.*` for a Paper backend server
- `amp-velocity-generic.*` for a Velocity proxy

## Important notes

1. **Use separate AMP instances**
   Run Velocity and each Paper backend as separate AMP instances.

2. **Set the JAR download URLs in AMP**
   These templates use direct `ServerJarURL` / `VelocityJarURL` settings so the install path is predictable (`server.jar` and `velocity.jar`).

3. **Java version matters**
   The templates default to installing `openjdk-21-jre-headless` in AMP containers. That works well for modern Velocity and for Paper 1.20 through 1.21.11. If you target newer Paper lines that require Java 25, switch the Docker image or Java package accordingly.

4. **Velocity config seeding**
   The Velocity template seeds `velocity.toml` and `forwarding.secret` on first start. After first boot, you can keep editing `velocity.toml` normally.

5. **Paper proxy forwarding is still a manual step**
   Because AMP Generic metaconfig does not natively manage Paper's YAML config files, you should manually edit `config/paper-global.yml` after first boot to enable Velocity modern forwarding and set the secret.

## Typical flow

1. Put these files in your AMP template repo.
2. Add the repo to AMP under `Configuration -> Instance Deployment -> Configuration Repositories`.
3. Fetch latest and refresh AMP.
4. Create one Velocity instance using the Velocity template.
5. Create one or more Paper instances using the Paper template.
6. In each Paper instance, set `ServerJarURL` to the Paper JAR you want.
7. In the Velocity instance, set `VelocityJarURL` to the Velocity JAR you want.
8. Start the Paper backend once so it generates its normal files.
9. Edit Paper `server.properties` / `config/paper-global.yml` for Velocity forwarding.
10. Start Velocity.
