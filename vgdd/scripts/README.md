# Scripts

Helper scripts invoked by skills/hooks — e.g. wrapping
`Unity -runTests -batchmode` (with `xvfb-run` when headless) and eval runners.
Note: Unity `-executeMethod` only calls **parameterless static** methods; pass
arguments via environment variables.
