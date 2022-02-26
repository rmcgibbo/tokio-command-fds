# tokio-command-fds

A library for passing arbitrary file descriptors when spawning child processes.
(Forked from https://github.com/google/command-fds to add support for tokio)

## Example

```rust
#[tokio::main(flavor="current_thread")]
async fn main() {
    use tokio_command_fds::{CommandFdExt, FdMapping};
    use std::fs::File;
    use std::os::unix::io::AsRawFd;
    use tokio::process::Command;

    // Open a file.
    let file = File::open("Cargo.toml").unwrap();

    // Prepare to run `ls -l /proc/self/fd` with some FDs mapped.
    let mut command = tokio::process::Command::new("ls");
    command.arg("-l").arg("/proc/self/fd");
    command
        .fd_mappings(vec![
            // Map `file` as FD 3 in the child process.
            FdMapping {
                parent_fd: file.as_raw_fd(),
                child_fd: 3,
            },
            // Map this process's stdin as FD 5 in the child process.
            FdMapping {
                parent_fd: 0,
                child_fd: 5,
            },
        ])
        .unwrap();

    // Spawn the child process.
    let mut child = command.spawn().unwrap();
    child.wait().await.unwrap();
}
```

## License

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).
