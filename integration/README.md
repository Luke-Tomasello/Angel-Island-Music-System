# integration

The three edits AIMS needs outside its own folder, one patch per file, made against stock RunUO 2.6 (each changed line
is marked `// AIMS:`). From your server's folder:

```
git apply integration/BaseInstrument.patch
git apply integration/SBBard.patch
git apply integration/BulletinBoards.patch
```

`git apply --check` tries a patch without changing anything. If your file has drifted from stock and a patch won't
apply, read it and make the same change by hand: each is a few lines.

| Patch | File | |
|---|---|---|
| `BaseInstrument.patch` | `Scripts\Items\Skill Items\Musical Instruments\BaseInstrument.cs` | Required: `ConsumeUse` is virtual (sheet music never wears out), and a music system instrument isn't picked as a bard's instrument |
| `SBBard.patch` | `Scripts\Mobiles\Vendors\SBInfo\SBBard.cs` | Bards sell manuscript paper, composition books and music boxes (set your own prices) |
| `BulletinBoards.patch` | `Scripts\Items\Misc\BulletinBoards.cs` | Optional: a composition book's **Paste** posts a whole song into the book |
