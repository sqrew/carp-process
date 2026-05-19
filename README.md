# carp-process

A foundational subprocess management library for the [Carp](https://github.com/carp-lang/Carp) programming language.

`carp-process` provides a well-behaved synchronous runtime for spawning and managing child processes with POSIX semantics. It handles the nuances of fork/exec, pipe management, and signal-aware process waiting.

## Features

- **Seeded Command Pattern**: Ergonomic building of process arguments without shell-injection risks.
- **Robust Redirection**: Capture `stdout` and `stderr` separately or combined.
- **Stdin Piping**: Support for passing strings to a child process's standard input.
- **Lifecycle Management**: Structured `ExitStatus` for distinguishing normal exits from signal terminations.
- **Resource Hygiene**: Strict use of `FD_CLOEXEC` and systematic descriptor cleanup to prevent leaks.
- **Syscall Resilience**: Explicit handling of `EINTR` and `EPIPE`.

## Usage

```carp
(load "carp-process/process.carp")
(use Process)

(defn main []
  (let [cmd (Command.new "ls")
        cmd-with-args (Command.arg cmd "-la")
        output (run &cmd-with-args)]
    (do
      (IO.println (Output.stdout &output))
      (let [status (Output.status &output)]
        (if @(ExitStatus.exited? status)
          (IO.println &(format "Exited with code: %d" @(ExitStatus.code status)))
          (IO.println &(format "Killed by signal: %d" @(ExitStatus.signal status))))))))
```

## Realistic Expectations & Safety

As a synchronous systems library, users should be aware of the following architectural trade-offs:

### 1. The Deadlock Risk
The `Process.run` function drains `stdout` and then `stderr` sequentially. **This can deadlock** if the child process concurrently generates enough output to fill the pipe buffers for both streams. 
- **Guidance**: For processes with high-volume or concurrent output, use `Process.run-combined` to merge streams at the OS level.

### 2. Synchronous Memory Model
All output is read entirely into memory before the function returns.
- **Guidance**: Not suitable for streaming massive log files or data pipelines that exceed available RAM.

### 3. Signal Safety
This library uses `_exit` in child processes to ensure async-signal safety and avoid corrupting shared runtime state after a fork. It also ignores `SIGPIPE` by default to rely on stable `EPIPE` error handling.

## Testing

Run the test suite with:

```bash
carp -x test/process_test.carp
```

## API

### `Command`
- `(Command.new path)`: Create a new command.
- `(Command.arg cmd arg)`: Add an argument to the command.
- `(Command.set-stdin cmd str)`: Set the string to be piped to stdin.

### `Process`
- `(Process.spawn path args)`: Spawn a process and return a `(Result Process.T String)`.
- `(Process.run cmd)`: Run to completion, capturing streams separately (**Deadlock prone for large output**).
- `(Process.run-combined cmd)`: Run to completion, merging stderr into stdout (**Deadlock safe**).
- `(Process.wait p)`: Block until exit, returning a structured `ExitStatus`.
- `(Process.terminate p)`: Send `SIGTERM` to the process.
- `(Process.kill! p)`: Send `SIGKILL` to the process.

## License

MIT
