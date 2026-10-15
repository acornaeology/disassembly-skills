# disassembly-skills

A Claude Code plugin carrying a method for producing verified, densely
annotated disassemblies of 6502 and 65C02 machine code (Acorn, BBC Micro and
similar) with [dasmos](https://acornaeology.github.io/dasmos/) and, optionally,
[fantasm](https://acornaeology.github.io/fantasm/).

It is the method behind the [acornaeology](https://acornaeology.uk)
disassemblies of Acorn NFS, ANFS, ADFS and others, distilled so it can be
applied to other ROMs and disc-loaded programs.

## What's in it

One skill, **6502-disassembly**, which Claude loads when you work on a
disassembly:

| File | Covers |
|---|---|
| `SKILL.md` | Principles and the working loop |
| `references/workflow.md` | Seeding entry points, relocation, disc-loaded and multi-file programs, deriving related versions |
| `references/idioms.md` | The 6502 idioms that cause most mis-decodes, and how to model them |
| `references/annotation.md` | Comment, banner, label and operand standards |
| `references/defects.md` | Defects that survive the round trip, and how to hunt them |
| `references/verification.md` | The round trip, confirming meaning, attributions |
| `references/dasmos.md` | A minimal driver and the dasmos calls that matter |
| `references/fantasm.md` | Adopting a project with fantasm, and the commands used |
| `references/project-claude-md.md` | A `CLAUDE.md` template for your project |

## Install

At the prompt of a Claude Code session in a terminal:

```
/plugin install disassembly-skills --marketplace acornaeology/disassembly-skills
```

From a shell:

```sh
claude plugin install disassembly-skills --marketplace acornaeology/disassembly-skills
```

In the Claude desktop app: click + next to the prompt box, then Plugins, Add
marketplace with `acornaeology/disassembly-skills`, then Add plugin.

## Using it

Install dasmos (and fantasm, if you want its checks) in your project's
environment, write a `CLAUDE.md` from the template, and ask Claude to work on
the disassembly. The skill loads when the task matches; you can also name it
("use the 6502-disassembly skill").

## Licence

MIT. See [LICENSE](LICENSE).
