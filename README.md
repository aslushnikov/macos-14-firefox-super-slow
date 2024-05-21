# Reduced repro for https://github.com/microsoft/playwright/issues/30705

This repo demonstrates the execution time of the following command on the Github Action's `macos-14` runners:

```
firefox --headless --screenshot /tmp/foo.png about:blank
```

- **Expected**: the command takes <= 2 seconds (at least as it is on [`macos-12`](https://github.com/aslushnikov/macos-14-firefox-super-slow/actions/runs/9169549868/job/25210171212) runners)
- **Actual**: the operation is 6+ seconds - see ["Measure Screenshot Time"](https://github.com/aslushnikov/macos-14-firefox-super-slow/actions/runs/9169549868/job/25210170926) step.

Additional details:

- This was reported in https://github.com/microsoft/playwright/issues/30705
- The `macos-14` runner is actually an `aarch64` runner, while `macos-12` runner is x86-64 runners.
- Enterprise Github Teams have access to `macos-14-large` (x86-64) and `macos-14-xlarge` (aarch64) runners.
  The issue reproduces on `macos-14-xlarge` as well, so it seems to be bound to aarch64 architecture.

# Local repro

While the issue doesn't reproduce on a local Apple Silicon Mac, it **does reproduce** under apple nested virtualization.

This is easily done with [tart](https://github.com/cirruslabs/tart/):

1. Run guest MacOS Sonoma on Apple Silicon:

    ```
    brew install cirruslabs/cli/tart
    tart clone ghcr.io/cirruslabs/macos-sonoma-base:latest sonoma-base
    tart run sonoma-base
    ```

2. Inside guest operating system, install firefox and run the command to measure time:

    ```
    brew install firefox
    firefox --version
    time firefox --headless --screenshot /tmp/foo.png about:blank
    ```

    <img width="1136" alt="image" src="https://github.com/aslushnikov/macos-14-firefox-super-slow/assets/746130/22fbc687-ab8f-4c96-83c9-d4c6552fb6c0">

