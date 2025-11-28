# BMB_DSL

**BMB_DSL** is the domain-specific language used to author dialogue trees for **Adventure64**.
It’s built to be readable, compact, and friendly for designers who prefer writing over clicking. The language mirrors the structure of the dialogue nodes, supports conditions and options, and includes a few special keywords for the Adventure64 Dialogue System.

This extension adds syntax highlighting and editor quality-of-life tools for `.bmbd` and `.bmbdsl` files in VSCode.

## Features

- Syntax highlighting for all core BMB_DSL keywords
- Highlighted comments, strings, labels, and hook events
- Visual clarity for OPTIONS / ENDOPTIONS blocks

## Language Overview

### SimpleDialogue Statement

BMB_DSL is composed of statements.

Here is a SimpleDialogue statement:

```bmbdsl
[Dialogue 1] "Hello, how are you?"
```

`[Dialogue 1]` is a **label**. it identifies a node in the dialogue tree. They are always surrounded by square brackets `[ ]`.

`"Hello, how are you?"` is a string. It is the dialogue itself. They are always surrounded by quotation marks `" "`.

The string can also be replaced by a string table entry path, like so:

```bmbdsl
[Dialogue 1] Gameplay.World.NpcDialogue.DIALOGUE_1
```

A SimpleDialogue statement chains into the next statement automatically. 

Identation is optional and does not affect the structure, which means all these variations are valid:

```bmbdsl
# Same Identation
[Dialogue 1] "Hello, how are you?"
[Dialogue 2] "I have lots to say to you."

# Next-level indentation
[Dialogue 1] "Hello, how are you?"
  [Dialogue 2] "I have lots to say to you."

# Same line (not recommended) 
[Dialogue 1] "Hello, how are you?" [Dialogue 2] "I have lots to say to you."
```
Comments may be added using #, but only when the # appears at the very start of a line. 

Comments cannot follow statements or appear mid-line.

If there is no next statement, the keyword `END` can be used.

Keywords are case-insensitive.

```bmbdsl
[Dialogue 1] "We are done speaking."
  End

```

### BranchNode Statement

A BranchNode statement is made using the `BRANCH`, `IF` and `ELSE` keywords.

```bmbdsl
Branch [Options Dialogue]
If Condition1
  [Branch 1] "Condition1 was met."
    End
# Any amount of IF statements can be added
If Condition2
  [Branch 2] "Condition2 was met."
    End
# The Default Case is required
Else
  [Default Branch] "No condition was met"
    End
```

`Condition1` and `Condition2` are conditions. 
They match the name of existing Conditions in Adventure64, and will cause a compilation error if they don't exist.

### OptionsDialogue

An OptionsDialogue statement is made with the `OPTIONS` and `ENDOPTIONS` keywords.

```bmbdsl
Options [Option Dialogue] "What will you pick?"
- "Yes"
  [Option 1] "You answered yes." End
- "No"
  [Option 2] "You answered no." End
- "Maybe"
  [Option 3] "You answered maybe." End
EndOptions
```

All the options have to be preceeded by a `-`.

They are refered as a DialogueOption.

They take a string or table entry path, and are followed by a statement.

The keyword `ENDOPTIONS` marks the end of the options list and is required.

### Jump Statement

A Jump statement is made with the `JUMP` keyword.
```bmbdsl
Jump [Dialogue 1]
```
It is followed by a label, and indicates a jump to an existing node.


### Hooks

The language also supports hooks.

Hooks are written with the `HOOK` keyword.

```bmbdsl
Hook OnEnter=PlaySound
```
`OnEnter` is the event name.
`PlaySound` is the invoked hook. 

hooks have to match the name of an existing hook in Adventure64, and will cause a compilation error if they don't.

```bmbdsl
Branch [Branch Node]
If Condition
# When the node is entered, the hook will be triggered
  Hook OnEnter=PlaySound
  [Dialogue 1] "Hello!"
    End
Else
  Options [Options Dialogue]
# When the "Yes" Option is picked, both hooks will trigger in order
  Hook OnSelect=MarkCompleted
  Hook OnSelect=PlaySound
  - "Yes"
    Hook OnAdvance=PlayAnimation
    [Dialogue 2] "Yeehaa!"
      End
  - "No" End
```
Hooks only apply to the statement immediately following them.

Here are the supported hooks for each type of statement.

| Statement / Event       | OnEnter | OnAdvance | OnSelect |
|-------------------------|:-------:|:---------:|:--------:|
| **SimpleDialogue**      |   ✔️    |     ✔️     |    ✖️     |
| **OptionsDialogue**     |   ✔️    |     ✖️     |    ✖️     |
| **BranchNode**          |   ✔️    |     ✖️     |    ✖️     |
| **DialogueOption**      |   ✖️    |     ✖️     |    ✔️     |

*DialogueOption is not a statement, so it only supports OnSelect.

### File Structure

The main body of a BMB_DSL file must contain exactly one root statement.
This is the entry point of the dialogue tree.

Any nested statement must always resolve into a terminal statement such as:
`END` or `JUMP`.

The language does not allow dangling or unclosed branches.
