# LazyMC

[LazyMC](https://github.com/timvisee/lazymc) is a Minecraft server proxy written in Rust. It automatically puts idle servers to sleep and wakes them when a player connects, saving CPU and memory when the server is not in use.

This egg uses a [modified fork by EfinaServer](https://github.com/EfinaServer/lazymc) that includes additional fixes and support on top of the original.

## Docker Images

| Label | Image |
|-------|-------|
| Java 21 | `ghcr.io/efinaserver/shells/java:21-lazymc` |
| Java 25 | `ghcr.io/efinaserver/shells/java:25-lazymc` |
| Java 11 | `ghcr.io/efinaserver/shells/java:11-lazymc` |
| Java 8 | `ghcr.io/efinaserver/shells/java:8-lazymc` |

## Startup Command

```
lazymc --public-address 0.0.0.0:{{SERVER_PORT}}
```

## Variables

Variables are grouped by function and listed in sort order.

### Server

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Server Startup Command | `LAZYMC_SERVER__COMMAND` | `java -Xmx1G -Xms1G -jar server.jar --nogui` | Command lazymc uses to start the Minecraft server |
| Freeze Process | `LAZYMC_SERVER__FREEZE_PROCESS` | `true` | Pause the server process instead of stopping it when all players leave (Linux/macOS only) |
| Wake on Start | `LAZYMC_SERVER__WAKE_ON_START` | `false` | Start the Minecraft server immediately when lazymc starts, without waiting for a player |
| Wake on Crash | `LAZYMC_SERVER__WAKE_ON_CRASH` | `false` | Automatically restart the server if it exits unexpectedly |
| Probe on Start | `LAZYMC_SERVER__PROBE_ON_START` | `false` | Connect to the server briefly on lazymc startup to detect its real version and protocol |
| Forge Mode | `LAZYMC_SERVER__FORGE` | `false` | Enable Forge compatibility mode (required for lobby join method with Forge) |
| Start Timeout | `LAZYMC_SERVER__START_TIMEOUT` | `300` | Max seconds to wait for the server to start before force-killing it |
| Stop Timeout | `LAZYMC_SERVER__STOP_TIMEOUT` | `150` | Max seconds to wait for the server to stop before force-killing it |
| Wake Whitelist | `LAZYMC_SERVER__WAKE_WHITELIST` | `false` | Only allow whitelisted players to wake a sleeping server |
| Block Banned IPs | `LAZYMC_SERVER__BLOCK_BANNED_IPS` | `true` | Reject connections from IPs in `banned-ips.json` |
| Drop Banned IPs | `LAZYMC_SERVER__DROP_BANNED_IPS` | `false` | Silently drop banned IPs at the network level (requires Block Banned IPs) |

### Public (Game Info shown while sleeping)

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Game Version | `LAZYMC_PUBLIC__VERSION` | `1.21.11` | Minecraft version string shown to clients while sleeping |
| Game Protocol | `LAZYMC_PUBLIC__PROTOCOL` | `774` | Minecraft protocol number matching the game version ([reference](https://git.io/J1Fvx)) |

### Time

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Sleep After | `LAZYMC_TIME__SLEEP_AFTER` | `60` | Seconds of inactivity before the server goes to sleep |
| Minimum Online Time | `LAZYMC_TIME__MINIMUM_ONLINE_TIME` | `60` | Minimum seconds the server stays online after starting |

### MOTD

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| MOTD (Sleeping) | `LAZYMC_MOTD__SLEEPING` | `☠ Server is sleeping\n§2☻ Join to start it up` | Server list MOTD shown while sleeping |
| MOTD (Starting) | `LAZYMC_MOTD__STARTING` | `§2☻ Server is starting...\n§7⌛ Please wait...` | Server list MOTD shown while starting |
| MOTD (Stopping) | `LAZYMC_MOTD__STOPPING` | `☠ Server going to sleep...\n⌛ Please wait...` | Server list MOTD shown while stopping |
| MOTD from Server | `LAZYMC_MOTD__FROM_SERVER` | `false` | Use the real server's MOTD after its first start instead of the static values above |

### Join

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Join Methods | `LAZYMC_JOIN__METHODS` | `hold,kick` | Comma-separated methods for handling connecting clients while the server starts (`hold`, `kick`, `forward`, `lobby`) |
| Hold Timeout | `LAZYMC_JOIN__HOLD__TIMEOUT` | `25` | Seconds to hold a client in loading screen while server starts (max 29; Minecraft client times out at 30) |
| Kick Message (Starting) | `LAZYMC_JOIN__KICK__STARTING` | *(see egg)* | Message sent to kicked clients while server is starting |
| Kick Message (Stopping) | `LAZYMC_JOIN__KICK__STOPPING` | *(see egg)* | Message sent to kicked clients while server is stopping |

### Lockout

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Lockout Enabled | `LAZYMC_LOCKOUT__ENABLED` | `false` | Block all incoming connections (useful during maintenance) |
| Lockout Message | `LAZYMC_LOCKOUT__MESSAGE` | *(see egg)* | Kick message shown when lockout is active |

### RCON

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| RCON Enabled | `LAZYMC_RCON__ENABLED` | `false` | Use RCON to stop the server gracefully instead of OS signals |
| RCON Port | `LAZYMC_RCON__PORT` | `25575` | Port for the server's RCON interface (must match `server.properties`) |
| RCON Password | `LAZYMC_RCON__PASSWORD` | *(empty)* | RCON password; leave empty to use Randomize Password |
| RCON Randomize Password | `LAZYMC_RCON__RANDOMIZE_PASSWORD` | `true` | Generate a random RCON password on each start and inject it into `server.properties` |

### Advanced

| Variable | Env | Default | Description |
|----------|-----|---------|-------------|
| Rewrite Server Properties | `LAZYMC_ADVANCED__REWRITE_SERVER_PROPERTIES` | `true` | Automatically update `server.properties` with values lazymc requires |
| Config Version | `LAZYMC_CONFIG__VERSION` | `0.2.11` | lazymc config schema version; do not modify unless upgrading config format |

## Notes

- The panel-assigned port is used by LazyMC as the public proxy address. The internal Minecraft server port is managed separately.
- Supports color codes in MOTD and kick messages using `§` followed by a color code (e.g. `§c` = red, `§7` = gray, `§r` = reset). Use `\n` for line breaks.
- `hold` join method keeps connecting clients in a loading screen; `kick` disconnects them with a message. Methods are tried in order.
