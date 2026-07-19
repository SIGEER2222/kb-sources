# SC2 Bank Persistence

Banks are SC2's per-player save files. A map declares which banks it uses via
`BankList.xml`; the engine reads/writes them to the player's local machine
between sessions.

## File location

Banks live in `Documents/StarCraft II/Banks/<map-author-id>/<bank-name>.SC2Bank`.
The exact author-id path is derived from the map's author; multiple maps from
the same author share the same bank namespace.

## File format

A `.SC2Bank` file is XML with a signature:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Bank version="1">
    <Section name="progress">
        <Key name="mission1">
            <Value text="completed"/>
        </Key>
        <Key name="score">
            <Value fixed="1250.0"/>
        </Key>
    </Section>
</Bank>
```

- `<Section>` groups related keys.
- `<Key>` is identified by `name` within its section.
- `<Value>` has typed attributes: `text`, `fixed`, `int`, `unit`, `point`, etc.
- The signature block at the end of the file (added by the engine) is required
  for the bank to load; tampering with values invalidates the signature.

## API

Common natives (see `galaxy/natives-reference.md`):

- `BankLoad(name, player)` → `bank` (or `null` if the file does not exist).
- `BankSave(bank, player)` → `bool`.
- `BankWait(bank, player)` blocks until the save completes.
- `BankKeyExists(bank, section, key)` → `bool`.
- `BankValueGetAsInt` / `AsFixed` / `AsString` / `AsUnit` / `AsPoint` typed getters.
- `BankValueSetFromInt` / `FromFixed` / `FromString` / `FromUnit` / `FromPoint`
  setters.
- `BankLastRestoredBank(player)` returns the bank restored at game start.

## Encryption and signing

The engine signs banks with a per-map key derived from the map's identity. To
deter tampering:

- Authors can call `BankOptionSet(bank, c_bankOptionSignature, true)` to require
  signature verification (default on).
- Custom encryption of stored values is the author's responsibility; the engine
  only verifies the signature, not the value contents.

## Patterns

### Save on victory

```galaxy
void SaveProgress () {
    bank b = BankLoad("MyMap", 1);
    BankValueSetFromInt(b, "progress", "missionsCompleted", gv_completedCount);
    BankSave(b, 1);
    BankWait(b, 1);
}
```

### Restore on load

```galaxy
void RestoreProgress () {
    bank b = BankLastRestoredBank(1);
    if (BankKeyExists(b, "progress", "missionsCompleted")) {
        gv_completedCount = BankValueGetAsInt(b, "progress", "missionsCompleted");
    }
}
```

## Pitfalls

- Banks are stored locally per player; multiplayer games use each player's own
  bank file.
- Loading a bank that does not exist returns `null`; always null-check.
- `BankSave` is asynchronous; if you need a save to complete before continuing,
  call `BankWait`.
- The 7vs1 coop test launcher writes a per-run test bank under
  `out/.test.lock` to coordinate test sessions; this is separate from in-game
  banks.

## See also

- `galaxy/natives-reference.md`
- `runtime-contracts/observer.md`
- `document-structure/overview.md`
