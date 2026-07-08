# Verse Authoring Rules — a drop-in ruleset for coding agents

Add this to your agent's system prompt / `AGENTS.md` when it writes **UEFN Verse**.

Derived from **1,418 real UEFN compiles**. The top 3 rules alone cover **73%
(nearly 3 in 4)** of observed compile failures. Every `RIGHT` example is verified
on the live UEFN compiler.

- Full human guide: **https://verseisland.com/guides/writing-verse-that-compiles**
- Verse-Coder model (free, runs on one GPU): **https://huggingface.co/BizaNator/Verse-Coder-30B-v1**
- More at **https://verseisland.com** — every example here was compiled on the live UEFN compiler (0 errors).
- License: CC BY 4.0 — attribute *Verse Island (verseisland.com)*.

Apply top-down: most `<override>` and "unknown identifier" errors are a missing
`using` block cascading downstream — fix imports first.

---

### 1. Import the API first — *34% of failures*
`editable`, `agent`, `button`, `trigger_device`, `creative_device` are not
built-ins. Put the `using` blocks at the top before you reference anything.
```verse
# WRONG — 'trigger_device' is an unknown identifier
my_game := class(creative_device):
    @editable Trigger : trigger_device = trigger_device{}

# RIGHT
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
my_game := class(creative_device):
    @editable Trigger : trigger_device = trigger_device{}
```

### 2. Device text is a `message`, not a `string` — *31%*
`SetText`/HUD/billboard setters take a `message`. Use the `<localizes>` pattern.
```verse
# WRONG
Billboard.SetText("Hello, island!")

# RIGHT
HelloIsland<localizes> : message = "Hello, island!"
Billboard.SetText(HelloIsland)
```

### 3. Real file shape — *25%*
`#` comments (not `//`), functions inside a class/module scope, no dangling `=`,
no invented keywords (`downto`, `as`), no reserved words as names.
```verse
# WRONG — // comment, floating fn, invented 'downto'
// count down
for (I := 10 downto 1): Print("{I}")

# RIGHT
CountDown()<suspends> : void =
    var Count : int = 10
    loop:
        if (Count <= 0) { break }
        Print("{Count}")
        set Count -= 1
        Sleep(1.0)
```

### 4. `<override>` needs a real parent — *16%* (usually a cascade of rule 1)
```verse
# WRONG — nothing to override (base class not imported)
my_game := class:
    OnBegin<override>()<suspends> : void = {}

# RIGHT
using { /Fortnite.com/Devices }
my_game := class(creative_device):
    OnBegin<override>()<suspends> : void = {}
```

### 5. Effectful calls need the right context — *16%*
A `<suspends>` call (e.g. `Sleep`) must be made from a `<suspends>` function;
`<decides>` bodies for failable logic; `spawn` takes exactly one call.
```verse
# WRONG
Start() : void = Sleep(1.0)

# RIGHT
Start()<suspends> : void = Sleep(1.0)
```

### 6. Never invent API members — *14%*
`HideProps`, `MoveToEase`, `Enable` don't exist. Use the members the type
declares (and note which ones suspend).
```verse
# WRONG
Prop.HideProps()
Prop.MoveToEase(NewPosition, 1.0)

# RIGHT — MoveTo suspends, so call it from a <suspends> fn
MoveProp(Prop : creative_prop, NewPosition : vector3, NewRotation : rotation)<suspends> : void =
    Prop.Hide()
    Prop.MoveTo(NewPosition, NewRotation, 1.0)
```

### 7. Match the event-handler signature — *12%*
Subscribed handlers must match the event's payload. Many device events send an
**optional** agent (`?agent`), not a plain `agent`.
```verse
# WRONG — TriggeredEvent sends ?agent
OnTriggered(Agent : agent) : void = {}
MyTrigger.TriggeredEvent.Subscribe(OnTriggered)

# RIGHT
OnTriggered(MaybeAgent : ?agent) : void = {}
MyTrigger.TriggeredEvent.Subscribe(OnTriggered)
```

### 8. Failable calls use `[]`, not `()` — *5%*
Square brackets, inside a failure context (`if`, `for`, a `<decides>` body).
```verse
# WRONG
Character := Agent.GetFortCharacter()

# RIGHT  (GetFortCharacter lives in /Fortnite.com/Characters — import it, see rule 1)
if (Character := Agent.GetFortCharacter[]):
    Character.GetTransform()
```

---

## TL;DR checklist
- [ ] `using {}` block for every API you touch — **do this first**
- [ ] device text = `message` (`<localizes>`), never a raw `string`
- [ ] `#` comments, methods in scope, no invented keywords
- [ ] `<override>` only on a real inherited method
- [ ] `<suspends>` / `<decides>` wherever the effect demands it
- [ ] never invent members — check the reference
- [ ] handler signatures match the event payload (`?agent` vs `agent`)
- [ ] failable calls use `[]` inside a failure context

Built and verified with [verseisland.com](https://verseisland.com) 🏝️
