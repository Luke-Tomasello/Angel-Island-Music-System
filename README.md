# Angel Island Music System (AIMS)

Real music in Ultima Online. With AIMS, players compose and perform music on the game's own instruments: they write
songs in composition books, turn them into sheet music anyone can play, and publish them to music boxes, where other
players listen, and buy them. It has been the music of the Angel Island shards since 2006.

Why it exists, and whose work it is: **[Why I Built AIMS](AIMusicSystem/Docs/WHY.md)**.

## Drop it into your RunUO server

AIMS runs on Angel Island, but it was made to drop into other shards. To prove it, we took a stock RunUO 2.6, moved it
to .NET 8, and dropped AIMS in: this folder, plus three small edits outside it. Everything below is what that took, and
every step was tested on that server, from composing a song to selling it between two accounts.

| In this repository | |
|---|---|
| `AIMusicSystem/` | The music system: copy it into your server's `Scripts\Engines\` |
| `integration/` | The three edits outside the folder, as patches you can apply or read |

The whole Angel Island server, with AIMS's Angel Island extras (placeable music devices, award instruments) and
**MidiToAims**, a composer's tool that turns MIDI files into songs, is in the
**[Angel Island Server](https://github.com/Luke-Tomasello/Angel-Island-Server)** repository.

## Before you start

- **.NET 8.** AIMS is written in modern C#. A server that compiles its scripts at startup with RunUO's built-in script
  compiler can't build it: compile your scripts with the .NET 8 SDK instead (an SDK project that builds the core and
  `Scripts\` into one program).
- **RunUO 2.6, or a server close to it.** AIMS needs nothing from the core that stock RunUO 2.6 doesn't have. A server
  further from stock may need small changes, all of them in `AIMusicSystem\Host\`.

## Step 1: Copy the folder

Copy `AIMusicSystem` into your server's `Scripts\Engines\`:

| Folder | What it is |
|---|---|
| `Core\` | The music engine: `[play`, timing, notes and sounds |
| `Compiler\` | Expands a song's sections and repeats |
| `RSM\` | Sheet music, manuscript paper and the composition book |
| `Library\` | The shard-wide library of published songs |
| `MusicBox\` | The music box |
| `Host\` | Everything AIMS needs from your shard, in one place ([Step 5](#step-5-wire-aims-to-your-shard)) |
| `Tools\MusicRecorder.cs` | `[MusicRecorder`: a performance written into a composition book |
| `Docs\` | `WHY.md`, and `README.md`, the reference for developers |

Don't define the `ANGEL_ISLAND` compile symbol: code under `#if ANGEL_ISLAND` is for Angel Island's own core.

## Step 2: Three small edits outside the folder

They're in `integration/`, one patch per file, made against stock RunUO 2.6. Apply them from your server's folder, or
read them and make the same changes by hand (each changed line is marked `// AIMS:`):

```
git apply integration/BaseInstrument.patch
git apply integration/SBBard.patch
git apply integration/BulletinBoards.patch
```

1. **`BaseInstrument.patch`** (required). Sheet music is an instrument to the game: it must never wear out
   (`ConsumeUse` becomes virtual), and it must never be picked as a bard's instrument.
2. **`SBBard.patch`** (so players can buy the items). Bards sell manuscript paper, composition books and music boxes;
   set your own prices.
3. **`BulletinBoards.patch`** (optional). A whole composition can be pasted into a book in one go: the book's **Paste**
   opens the client's bulletin board window, and this sends the post to the book. Without it, players type into the
   book instead.

## Step 3: Build and run

Build and start your server. There is nothing to configure: AIMS starts with defaults that work on stock RunUO.

## Step 4: Check that it works

Do these in order. Each one says what you should see.

1. **Play notes.** Say `[play a b c`. You hear three harp notes.
2. **Buy the tools.** A bard sells manuscript paper, music composition books and music boxes.
3. **Write a song.** Put a composition book in your backpack and type a short song into it:
   ```
   # Song: Test Song
   # Author: You
   # Price: 1000
   say '[play c d e f g'
   wait 500
   say '[play g f e d c'
   ```
   (Or single-click the book, choose **Paste**, and paste a whole song in one post, if you applied the third patch.)
   The price makes it a premium song, sold through music boxes; `# Price: 0` makes it free.
4. **Preview it.** Say `[play preview` and target the book. It plays. Say `[play preview` again to stop.
5. **Make sheet music.** Double-click the manuscript paper and target the book. You get rolled up sheet music named
   *Test Song*. Double-click it: it plays.
6. **Set up a music box.** Lock a music box down in your house, single-click it and rename it (Angel Island's is
   *Alexa*). Say `Alexa, help`: it lists its commands.
7. **Publish.** Say `Alexa, submit` and target your sheet music. Staff of Administrator level and up are told a new song
   needs review; it plays on your own box meanwhile, marked as pending approval.
8. **Review it.** As an Owner, say `Alexa, review` (it plays the song), then `Alexa, approve test` (or `deny`).
9. **Sell it.** A player on a **second account** puts gold in their bank, says `Alexa, play test` (a 10-second
   sample), then `Alexa, buy test`. The price leaves the buyer's bank and lands in the composer's, and the song now
   plays in full for the buyer.
10. **Check the books.** The buyer says `Alexa, purchases`; the composer says `Alexa, sales`.
11. **Record a performance.** Say `[MusicRecorder start`, play a few `[play` lines and say a lyric, then
    `[MusicRecorder stop` and target a composition book: your performance is in the book, with the pauses.

A second account from the same computer: stock RunUO allows one account per IP address. For testing, raise
`MaxAccountsPerIP` in `Scripts\Accounting\AccountHandler.cs`.

## Step 5: Wire AIMS to your shard

AIMS provides the vehicle; your shard provides the destination. Everything AIMS needs from a shard goes through one
class, `AimsHost` (`Host\AimsHost.cs`), and every member has a default that works on stock RunUO. To change one,
derive a class from `AimsHost` anywhere in your scripts and override what you want; AIMS finds it on its own.

```csharp
namespace Server
{
    public class MyShardAimsHost : AimsHost
    {
        // staff may buy music and use test commands on our Test Center
        public override bool IsTestCenter => Server.Misc.TestCenter.Enabled;

        // our leaderboard counts music sales
        public override void OnMusicSold(Mobile composer, int price) { /* add to your leaderboard */ }
        public override bool HasLeaderboard => true;
    }
}
```

| Member | What AIMS uses it for | Default |
|---|---|---|
| `LogException`, `WriteLine`, `WriteLog` | Errors, console lines, log files (`MusicRecorder.log`) | The console; files in `Logs\` |
| `SendMessageNearby`, `SendSystemMessage` | Status lines for staff near a music item | Staff within 10 tiles |
| `DataDirectory`, `LogsDirectory` | Where song files and logs go | `Data\` and `Logs\` beside the server |
| `AccountGuid` | Who composed a song and who owns it, for good | A stable number from the account name |
| `IsHousehold` | A composer isn't paid for a household member's purchase | The same account |
| `Get/SetMusicKarma` | Music Karma: earned by composing, publishing, sales and plays; 1,000 unlocks sound effects | Kept by AIMS |
| `Get/SetFilterMusic` | `[FilterMusic`: a player's mute for other players' music | Kept by AIMS |
| `Withdraw` | Paying for a song | The buyer's bank box |
| `Dupe`, `MarkStaffOwned` | Staff copies of sheet music (`dupe`, `[DupeMusic`) | The sheet copies itself; nothing tracked |
| `MarkTemporary` | Temporary playback copies (AIMS deletes its own; this is your safety net) | Nothing |
| `MoveToInternal` | Where the library keeps its sheets | Off the map (`Internalize`) |
| `SystemMobile` | Old Angel Island saves only | None |
| `IsTestCenter` | `flushall`, and staff buying music | No |
| `PlayAccessLevel` | Who may use `[play` | Players |
| `MusicEngineLog` | `Logs\MusicEngine.log`: playback timing measurements | On |
| `OnMusicSold`, `HasLeaderboard` | A sale, for your leaderboard; and whether to tell the composer to check it | Nothing; no |

Angel Island's own host, `AngelIslandAimsHost`, wires all of these to its systems: it's a complete example of what a
shard can connect (in the Angel Island Server repository, `scripts/Engines/AIMusicHost`).

## Where AIMS keeps its data

| File | What |
|---|---|
| `Saves\Music.bin` | The library: every published song and its record |
| `Saves\MusicContexts.bin` | Each player's `[play` settings |
| `Saves\AIMS\Players.bin` | Music Karma and `[FilterMusic`, when your host doesn't keep them |
| `Logs\MusicEngine.log` | Playback measurements (turn off with `MusicEngineLog`) |

## Troubleshooting

### Houses are the wrong size, or you drop through the floor

Not AIMS, but you'll meet it placing a music box. `multi.mul` changed format with the High Seas client (7.0.9.0). In
`Scripts\Misc\MapDefinitions.cs`, `MultiComponentList.PostHSFormat` must match your client files: `false` for files
from before 7.0.9.0, `true` from then on. Stock RunUO 2.6 says `true`.

### "Staff cannot buy music on the production shard"

Staff can only buy music on a Test Center. Test with a player account, or override `IsTestCenter`.

### "Unauthorized" for `flushall`

`flushall` removes every song on the shard: an Owner on a Test Center only.

### Cyan "Music engine" lines in the console

A Debug build prints the playback measurements; a Release build doesn't. `Logs\MusicEngine.log` gets them either way,
unless your host turns `MusicEngineLog` off.

## Hear it

- The 2022 [Battle of the Bards](https://www.youtube.com/watch?v=GCqLpk5YO1c), where Angel Island's composers shared
  their work
- [Ultima Online Free Shard](https://www.youtube.com/@UltimaOnlineFreeShard) on YouTube
- Songs by Galois, Angel Island's Grandmaster Composer, shared with his permission, are in the Angel Island Server
  repository (`AI Music Composer/Data`)

## Credits and license

AIMS began with Rhiannon's music player in 2006, with fixes from Pix and Adam, and grew into the music system it is
through Adam's work over the years (above all his 2022 rewrite), shaped by Angel Island's composers. If you run it,
please keep the names and the history that came with it: [Why I Built AIMS](AIMusicSystem/Docs/WHY.md).

GPL (see `LICENSE`). Built on RunUO, by the RunUO Software Team.
