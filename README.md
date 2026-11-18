# [TF2] Vote Scramble

A [SourceMod](https://www.sourcemod.net/) plugin for Team Fortress 2 that lets players vote to scramble (reshuffle) the teams. When the vote passes, the teams are scrambled and the map's round/time limits are adjusted so the scramble doesn't artificially extend the map.

- **Version:** 2.0
- **Authors:** Nanochip, viora, raspy, leighmacdonald
- **URL:** https://uncletopia.com
- **License:** MIT (see [LICENSE](LICENSE))

## Features

- **Two-stage voting.** Players first raise a chat vote; once enough players have voiced support, a formal in-game (NativeVotes) vote is held for everyone to approve. Both stages are configurable, and the second vote can optionally be skipped.
- **Chat and command triggers.** Type a command or simply say `votescramble` / `scramble` in chat.
- **Admin force option.** Admins with the `ADMFLAG_VOTE` flag can force a scramble vote immediately.
- **Cooldown protection.** Prevents repeated scramble votes from being started back to back.
- **Map-limit compensation.** Subtracts rounds/minutes elapsed since the last scramble from `mp_maxrounds`, `mp_winlimit`, and `mp_timelimit` so the map doesn't get extended by the reshuffle.
- **Optional balanced scrambles.** If [gbans](https://github.com/uncletopia/gbans) is available, its balanced scramble is used instead of the built-in `mp_scrambleteams`. Falls back gracefully to the standard scramble when gbans is missing or the scramble fails.

## Requirements

- SourceMod 1.10+ (uses the `newdecls required` syntax and NativeVotes)
- `nativevotes` include (vendored in [`include/`](include/nativevotes.inc))
- `sdktools` include (ships with SourceMod)
- `gbans` include for balanced scrambles (vendored in [`include/`](include/gbans.inc)); the gbans plugin itself is optional at runtime — without it the plugin falls back to `mp_scrambleteams`

The third-party includes live in `include/`. Compile with that directory on the include path, e.g.:

```sh
spcomp -i include votescramble.sp -o votescramble.smx
```

The release workflow does this automatically using SourceMod 1.12.

## Installation

1. Copy `votescramble.sp` into your SourceMod `addons/sourcemod/scripting` directory.
2. Compile it to produce `votescramble.smx`.
3. Copy `votescramble.smx` into `addons/sourcemod/plugins` on your server.
4. (Optional) Install gbans to enable balanced scrambles. When its include isn't present at compile time, the plugin automatically falls back to `mp_scrambleteams`.
5. Restart the server or reload the plugin (`sm plugins load votescramble`).

A config file with all the `votescramble_*` convars is generated automatically at `cfg/sourcemod/plugin.votescramble.cfg` on first load.

## Commands

| Command | Access | Description |
| --- | --- | --- |
| `sm_votescramble` | All players | Add your vote to start a team scramble vote. |
| `sm_vscramble` | All players | Alias for `sm_votescramble`. |
| `sm_scramble` | All players | Alias for `sm_votescramble`. |
| `sm_forcescramble` | `ADMFLAG_VOTE` | Immediately start a scramble vote, bypassing the chat-vote stage. |

Players can also type `votescramble`, `vscramble`, `scramble`, or `scrimblo` in chat to trigger the same behavior as `sm_votescramble`.

## ConVars

| ConVar | Default | Description |
| --- | --- | --- |
| `votescramble_version` | `2.0` | Plugin version (not recorded in config). |
| `votescramble_time` | `20.0` | Duration, in seconds, of the NativeVotes menu vote. |
| `votescramble_delay` | `180.0` | Cooldown, in seconds, before another scramble vote can be initiated. |
| `votescramble_chat_percentage` | `0.20` | Fraction of players required for the initial chat vote to pass (e.g. `0.20` = 20%). Range: 0.05–1.0. |
| `votescramble_menu_percentage` | `0.60` | Fraction of "yes" votes required for the menu vote to pass (e.g. `0.60` = 60%). Range: 0.05–1.0. |
| `votescramble_minimum` | `3` | Minimum number of votes needed to trigger the menu vote, regardless of percentage. |
| `votescramble_skip_second_vote` | `0` | When `1`, skip the menu vote and scramble as soon as the chat vote succeeds. |
| `votescramble_use_gbans` | `1` | When gbans is compiled in and loaded, use its balanced scramble instead of `mp_scrambleteams`. |

## How it works

1. A player starts a chat vote using a command or chat keyword. Each player may vote once.
2. The required number of chat votes is calculated as `ceil(active players * votescramble_chat_percentage)`, with a floor of `votescramble_minimum`. Disconnects update the count automatically.
3. When enough chat votes are in, a cooldown begins and a NativeVotes menu ("Scramble teams?") is shown to everyone, unless `votescramble_skip_second_vote` is enabled.
4. If at least `votescramble_menu_percentage` of the menu votes are "Yes", the teams are scrambled.
5. After the scramble, the plugin subtracts the rounds/minutes played since the last scramble from the active map limits (`mp_maxrounds`, `mp_winlimit`, `mp_timelimit`) to avoid extending the map. It applies the scramble on the next round start (or handles the gbans case, which scrambles immediately).

If another vote is already in progress, the plugin waits and retries after 10 seconds.

## License

This project is released under the [MIT License](LICENSE).
