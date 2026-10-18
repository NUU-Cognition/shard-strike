# 0.1.2

- One workflow for each mode (NUU Orbh Task 748). `wkfl-strk-strike_start`, `wkfl-strk-discuss_start`, and `wkfl-strk-ops_do` have `> [!headless]` callouts in their stages. `hwkfl-strk-create_and_do` is now `wkfl-strk-create_and_do`, a workflow for a headless session. `init-strk` has a `# Headless` section, with the table that maps the old names of a Strike TUI launch prompt to the workflows. The `hwkfl-strk-*` files and `hinit-strk` are deleted. A headless session loads the shard with `flint shard start strk`; `hstart` still loads this init, with a notice.

# 0.1.0

- Initial shard scaffold
