# process

Run other programs from Quirk: wait for them and capture their output, or
start them and talk to them over pipes.

```quirk
use process

r := process.run(["git", "status", "--short"])
if r.ok {
    print(r.stdout)
} else {
    print("git failed: " + r.stderr)
}

branch := process.output(["git", "rev-parse", "--abbrev-ref", "HEAD"])

sorted := process.run(["sort"], input="b\na\nc\n").stdout     // "a\nb\nc\n"

p := process.spawn(["python3", "-u", "worker.py"])
p.write("job 1\n")
print(p.read_line())
p.close_stdin()
code := p.wait()
```

## API

| | |
|---|---|
| `process.run(args, ...)` | Run to completion; returns a `RunResult` (`code`, `stdout`, `stderr`, `timed_out`, `ok`) |
| `process.shell(command, ...)` | The same through `/bin/sh -c` (pipes, globs, `$VARS`) |
| `process.output(args, ...)` | Just stdout, trailing newline removed; throws `ProcessError` on a non-zero exit |
| `process.spawn(args, ...)` | Start it and return a `Process`: `write`, `close_stdin`, `read`, `read_line`, `lines`, `read_err`, `communicate`, `wait`, `wait_for`, `done`, `signal`, `terminate`, `kill` |
| `process.pid()` | This process's id |

`args` is a List — the program and its arguments, passed as-is with no
shell in between. Options: `cwd`, `env` (Map, added to this process's
environment; `inherit_env=false` makes it the whole environment),
`input`, `timeout_ms`, `check`, `capture`, `merge_stderr`; `spawn` takes
`stdin` / `stdout` / `stderr` = `"pipe"`, `"inherit"`, `"null"` (and
`stderr="stdout"`).

A program that can't be found throws `FileNotFoundError`. Inside an
`async` task, waits let the other tasks run, and `task.cancel()` kills the
program and raises `CancelledError`.

Needs the Quirk compiler 7.17.0+ (the `Process_*` runtime). Bundled with
the compiler at `<QUIRK_HOME>/packages/process/`, so `use process` works
out of the box; this repo is its canonical home:

```bash
quirk install process            # latest tag from this repo
quirk install process@1.0.0      # pinned
```

POSIX only (Linux, macOS) for now.

## License

MIT (see [`LICENSE`](LICENSE)).
