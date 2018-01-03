# How works

The repo tools are declared in the `paket.dependencies` as `repotool`, like

```
repotool myhello 0.4.0
```

These works like normal nupkg for resolution, but are standalone and doesnt contribute to project references.
The version resolved by `paket install` will be locked (as usual) in `paket.lock`, and that specific version will be used (as usual) by `paket restore`

SPEC:

- A nupkg who is a repo tools, just need to add console app in `tools` directory.
- can support multiple runtimes in same nupkg
  - .net framework console app (require .NET Framework or mono installed)
  - .net core console app published as FDD (require .net core runtime installed)

PAKET BEHAVIOUR:

idea is to have wrapper script in a fixed location to invoke a tool (but each tool can use a different runtime or be in different install location)
the preferred runtime to use is not locked

- paket install/restore the nupkg as usual (settings for nupkg like storage:none/groups/source/etc apply too)
- paket after downloading the nupkg (on restore/install) will create some shell scripts to invoke these
- the shell script, in `paket-files/bin` can be invoked as `paket-files/bin/mytool`:
    - for win is `mytool.cmd` a normal batch script
    - for osx/linux is `mytool` a normal shell script
- the shell script will invoke:
    - for .net fw console app: `mono mytool.exe` on osx/unix, directly `mytool.exe` on win
    - for .net core console app: `dotnet mytool.dll`
- doesnt matter where the packages are restored, if in `packages` dir or in `storage:none` (user nuget cache dir).
- normally is enough `paket-files/bin/mytool` (with right dir separator) in both windows/unix, if run as shell command
- adding `paket-files\bin` to `PATH`, the commands can be run as `mytool` directly (without `paket-files/bin` full path)
- paket will create a helper script to help modify the `PATH` env var:
  - for win: `paket-files\bin\add_to_PATH`
  - for unix: `source paket-files/bin/add_to_PATH.sh`

Just .NET Framework and .NET Core console app are supported (atm)

- any `tools/*.exe` is considered a .NET Framework console app.
    - see `FAKE` nupkg as example (useful for old tools)
- any `tools/net{version}/*.exe` is considered a .NET Framework console app.
    - see `RepoTool.Sample` nupkg as example
- any `tools/netcoreapp{version}/{pkgName}.dll` is considered a .NET Core console app.
    - see `myhello` nupkg as example
    - the console app must be an FDD, so require .net core runtime installed
    - no way to specify an alias (yet), the name of entry point assembly should be the same of nupkg

paket after `install` or `restore` will create some shell script in `paket-files\bin` to invoke these tools

## KNOWN BUGS or NOT IMPLEMENTED YET

- [ ] configure alias, especially for .net core tools
- [ ] chmod+x for shell scripts
- [ ] warning if the prefferred runtime is not supported on restore by a tool
- [ ] powershell helper script to change `$env:PATH`

## RAW ideas

- [ ] use native wrappers instead of shell scripts
- [ ] kill the child tool it if parent process (the script) is termined

## Examples

**NOTE** Each example assume a clean repo, so feel free to `git clean -xdf` at start

Examples use windows dir separator, sry, but fixing that should works on unix or windows WLS

### 1 - Basic example, from install

```bat
.paket\paket install
```

that will:

- create the dir `paket-files\paket\bin` with the shell scripts
- lock the repo tools in the `paket.lock` like `myhello (0.4.0) - repotool: true`

Is possible to run installed commands, like

```bat
paket-files\bin\FAKE --version
paket-files\bin\hello 1 2 3
paket-files\bin\myhello a b c
```

### 2 - Basic example, from restore

do `.paket\paket install`, remove the `paket-files` directory, but leave `paket.lock`

now run `.paket\paket restore`

same as example 1

```bat
paket-files\bin\myhello a b c
```

### 3 - Multi runtimes

A repo tools can support multiple runtime. so both .net core and .net.

by default, the preferred runtime is the one of paket executable (atm is .net)
but can be configured unsing `PAKET_REPOTOOL_PREFERRED_RUNTIME` env var

valid values: `net`/`netcoreapp`

set the env var:

```bat
set PAKET_REPOTOOL_PREFERRED_RUNTIME=netcoreapp
```

now let's install

```bat
.paket\paket install
```

and run a .net core tool, like the .net version

```bat
paket-files\bin\myhello
```

in the example, the `myhello` tool will now use .net core.

if you cat the `paket-files\bin\mytool.sh` will be:

```bat
dotnet "%~dp0..\..\packages\myhello\tools\netcoreapp2.0\myhello.dll" %*
```

normally is:

```bat
"%~dp0..\..\packages\myhello\tools\net45\myhello.exe" %*
```

run as `dotnet` will respect the `global.json` file, if exist

### 4 - type full path is annoying, use the PATH

```
.paket\paket install
```

now add the `paket-files\bin` to `PATH`

```
set PATH=%CD%\paket-files\bin;%PATH%
```

and run the commands

```
FAKE --version
hello 1 2 3
myhello a b c
```

The working directory doesnt matter anymore

```
mkdir prova
cd prova
myhello
```

### 5 - change the PATH is annoying, use the helper script

manuall change the `PATH` is annoying and error prone.

after install/restore, paket generate an helper script (two, by os)

- in windows command prompt: `paket-files\bin\add_to_PATH`
- in unix shell: `source paket-files/bin/add_to_PATH.sh`

this will just add the `paket-files/bin` dir as prefix in `PATH` env var for current session

so now

```
FAKE --version
```

works, same as example 4
