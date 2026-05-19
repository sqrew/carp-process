# Examples

## Running a Command (High Level)

The `Process.run` function is the easiest way to execute a command and capture its output.

```carp
(load "carp-process/process.carp")
(use Process)

(defn main []
  (let [cmd (Command.new "ls")
        cmd-args (Command.arg cmd "-la")
        output (run &cmd-args)]
    (do
      (IO.println (Output.stdout &output))
      (let [status (Output.status &output)]
        (if @(ExitStatus.exited? status)
          (IO.println &(format "Exited with code: %d" @(ExitStatus.code status)))
          (IO.println &(format "Killed by signal: %d" @(ExitStatus.signal status))))))))
```

## Deadlock-Safe Execution

For processes that generate a lot of output to both `stdout` and `stderr` concurrently, use `run-combined` to merge the streams at the OS level and avoid pipe-buffer deadlocks.

```carp
(let [cmd (Command.new "python3")
      cmd-args (Command.arg cmd "heavy_output.py")
      output (run-combined &cmd-args)]
  (IO.println (Output.stdout &output)))
```

## Stdin Piping

You can pass a string to the process's standard input:

```carp
(let [cmd (Command.new "grep")
      cmd-arg (Command.arg cmd "carp")
      cmd-in (Command.set-stdin cmd-arg "hello\ncarp\nworld")
      output (run &cmd-in)]
  (IO.println (Output.stdout &output))) ;; Outputs "carp"
```

## Low-Level Spawning

If you need more control (like keeping the process running in the background), use `spawn`:

```carp
(let [p-res (spawn "sleep" &[@"10"])]
  (match p-res
    (Result.Success p) (do
                         (IO.println "Process running...")
                         (let [status (wait &p)]
                           (IO.println "Done.")))
    (Result.Error err) (IO.errorln &err)))
```
