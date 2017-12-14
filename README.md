

run boostrapper (add usual prefix `mono` on linux/mac), to setup the .net core version)

```bat
.paket\paket.bootstrapper.exe -f -v
```

now you have a `.paket\paket.cmd` (for win) and `.paket/paket` (for unix/mac).
as a note, both can be run with just `paket` if that directory is in `PATH`

```bat
.paket\paket restore -v
```

Now the dir `paket-files\paket\bin` is in `PATH`
so is possible to run installed commands, like

```bat
hello
```
