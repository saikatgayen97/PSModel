# PSModel Workspace Guidelines

All development and automated agent tasks in this repository must strictly adhere to the developer rules defined in [developer/DEVELOPER_RULES.md](developer/DEVELOPER_RULES.md):

1. **Data**: All `.npy`, `.npz`, and `.fits` files must be saved in `data/`.
2. **Catalogues**: All catalogue `.csv` files must be saved in `catalogue/`.
3. **Documentation**: All documentation, guides, and manuals must be in `doc/`.
4. **Scripts**: All Python scripts and executables must be placed in `scripts/`.
5. **Plots**: All plots and figures (`.png`, `.pdf`, `.svg`) must be saved in `plots/`.
6. **Execution & Logs**: All commands and runs must be executed from `workdir/`, with all logs written to `workdir/logs/`.
7. **Developer Rules**: All developer rules and guidelines must reside in `developer/`.
8. **Releases**: `release/` contains the `VERSION` file tracking the version number and all release tar files.
9. **Agent Lock (`agent.lock`)**: `agent.lock` must be `true` when the agent is idle/not working. When an agent begins modifying files, it checks `agent.lock`; if `true`, it proceeds and sets it to `false`. When done, it resets `agent.lock` to `true`.

