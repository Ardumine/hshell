# hshell

The HCore interactive shell — provides the REPL, script runner, and all built-in
filesystem/process commands.

## Files

```
hshell/
├── HCore.Packages.HShell.csproj   # Project — targets net10.0, refs Base + ReadLine
├── Shell/                         # Module directory
│   ├── ShellImplement.cs          # Shell REPL, script runner, command dispatch
│   ├── ModDescriptor.cs           # IModuleDescriptor (name: HCore.Packages.HShell.Shell)
│   └── Commands/
│       ├── CommandRegistry.cs     # Name → ICommand dictionary
│       ├── ManifestCommand.cs     # Lazy proxy for manifest-declared one-shot commands
│       ├── FileSystemCommands.cs  # ls, cd, cat, mkdir, rm, touch, mv, write, etc.
│       ├── ProcessCommands.cs     # spawn, run, kill
│       ├── ServiceCommand.cs      # service — reaches IServiceManager on init
│       ├── AfcpCommand.cs         # afcp — reaches IAfcpKernel on @afcp
│       ├── HelpCommand.cs         # help, exit
│       └── AutoCompletionHandler.cs # Tab-completion for commands, paths, sub-commands
├── manifest.json                  # Package metadata
├── mpd                            # HCore.Packages.HShell.dll
└── README.md
```

## Build & Install

```bash
# 1. Clone alongside your hcore repo
git clone https://github.com/Ardumine/hshell.git
# Expected layout:
#   ardumine/hcore/   ← kernel (with HCore.Modules.Base)
#   ardumine/hshell/  ← this module

# 2. Build (deploys to hcore/FS/packs/ via PostBuild)
dotnet build
```

The shell is a system package — it must be pre-installed in `FS/packs/HCore.Packages.HShell/`
(part of the base distribution tarball). It cannot be installed via `hpm install` alone
because hpm requires the shell to be running (chicken-and-egg).

## Architecture

The shell implements `IShell : IRunnable` from `HCore.Modules.Base`. Init spawns it as
a child: `SpawnChildByName<IShell>("HCore.Packages.HShell.Shell", "console", null)`.

Commands extend `ICommand` from `HCore.Modules.Base`. External packages register commands
two ways:

- **Manifest-declared** (oneshot): `manifest.json` → shell spawns child per invocation
- **Runtime-registered** (persistent): module calls `shell.RegisterCommand(new MyCmd())`

See [PACKAGE_SYSTEM.md](https://github.com/Ardumine/hcore/blob/master/docs/packages/PACKAGE_SYSTEM.md).
