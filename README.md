

run boostrapper (add usual prefix `mono` on linux/mac), to setup the .net core version)

```bat
.paket\paket.bootstrapper.exe -f -v
```

now you have a `.paket\paket.cmd` (for win) and `.paket/paket` (for unix/mac).
as a note, both can be run with just `paket` if that directory is in `PATH`

```bat

```bat
.paket\paket restore -v
```

To build it, as usual

```bat
dotnet build c1 -v n
```
