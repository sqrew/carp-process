# carp-process

A foundational subprocess management library for the [Carp](https://github.com/carp-lang/Carp) programming language.

## Features

- **Command Pattern**: Ergonomic API for building process arguments.
- **Output Capture**: Easily capture `stdout` and `stderr`.
- **Process Management**: Wait, terminate, or kill child processes.
- **Pipes**: Low-level access to pipe-based communication.

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
      (IO.println &(format "Exit code: %d" @(Output.exit-code &output))))))
```

## Testing

Run the test suite with:

```bash
carp -x test/process_test.carp
```

### `Command`
- `(Command.new path)`: Create a new command.
- `(Command.arg cmd arg)`: Add an argument to the command.

### `Process`
- `(Process.spawn path args)`: Spawn a process and return a `(Result Process.T String)`.
- `(Process.run cmd)`: Run a command to completion and capture output. Returns `Process.Output`.
- `(Process.wait p)`: Block until the process exits and return the exit code.
- `(Process.terminate p)`: Send `SIGTERM` to the process.
- `(Process.kill! p)`: Send `SIGKILL` to the process.

## License

MIT
