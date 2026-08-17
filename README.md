# Continuum

A Minecraft server proxy with unparalleled server support, scalability,
and flexibility.

## Licence

Continuum is licensed under the **GNU General Public License v3.0**, the same
licence as Velocity, which it is derived from. The full text is in
[LICENSE](LICENSE).

## Features

* Everything vanilla Velocity offers, tracking the `dev/4.0.0` upstream.
* Toggleable removal of the reconfiguration stage on server switches
  (`remove-reconfig`).
* Toggleable client-world preservation across server switches
  (`keep-client-world-on-switch`).
* The `ClientWorldSwitches` API, exposing the client-visible entity ID to
  coordinating plugins for world-preserving switches.

## Remove Reconfig

By default, a player switching backend servers is sent back through the
configuration state, which shows the "Reconfiguring..." screen and makes the
resource pack, tab list and scoreboard flicker.

Setting `remove-reconfig = true` near the top of `velocity.toml` keeps the
player in the play state for the whole switch. The proxy answers the backend's
known-packs handshake on the client's behalf, replays the client brand, and
clears the scoreboard objectives, teams and boss bars the previous server left
behind. Titles are not reset either, so an in-progress teleport keeps its fade.

This is **off by default**.

## Keep Client World on Switch

Setting `keep-client-world-on-switch = true` lets a backend switch preserve the
client's loaded world instead of reloading it from the next backend's join-game
packet. The proxy tracks the client's dimension and entity ID on join and
respawn, and on switch it keeps the client in the play state when two
conditions hold:

* the destination backend joins the client into the **same dimension** the
  client was last told about, and
* the destination backend assigns the **entity ID the client already has** for
  itself (through Paper's internal entity-id API).

When those hold, the destination spawns the player into a world the client has
already loaded — no registry resend, no world reload, no freezing screen.

Requires `remove-reconfig = true` to be effective; without it the switch still
passes through the configuration state, which reloads the world. This is
**off by default**.

### Notes

* Every backend server must run the same Minecraft protocol version. The client
  keeps the registry data it received from the first server it joined, so a
  backend on a different version will desync the client and cause visual
  corruption or kicks.
* Leave the seamless-switch options off if your backends are not all on the
  same version.

## ClientWorldSwitches API

`com.velocitypowered.api.proxy.player.ClientWorldSwitches` tracks the entity ID
the client currently uses for its own player. A coordinating proxy plugin can
read it with `clientEntityId(UUID)` and pass it to the destination server for a
seamless switch, so the join-game packet on arrival reuses the client's view
instead of resetting it.

## Building

Continuum is built with [Gradle](https://gradle.org). We recommend using the
wrapper script (`./gradlew`) as our CI builds using it.

It is sufficient to run `./gradlew build` to run the full build cycle.

## Running

Once you've built Continuum, you can copy and run the `-all` JAR from
`proxy/build/libs`. Continuum will generate a default configuration file, and
you can configure it from there.

## Thanks to These Projects

* [Velocity](https://github.com/PaperMC/Velocity)
* [Velocity (SunnySMP Fork)](https://github.com/Sunny-SMP/Velocity)