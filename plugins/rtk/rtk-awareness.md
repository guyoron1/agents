# RTK — Rust Token Killer

Token-optimized CLI proxy. When RTK is installed, the `rtk-rewrite.sh`
hook transparently rewrites commands (`git status` → `rtk git status`)
for 60–90% token savings on dev operations.

## Meta commands (use rtk directly)

```bash
rtk gain              # Show token savings analytics
rtk gain --history    # Show command usage history with savings
rtk discover          # Analyze Claude Code history for missed opportunities
rtk proxy <cmd>       # Execute raw command without filtering (for debugging)
```

## Verification

```bash
rtk --version         # Should show: rtk X.Y.Z
which rtk             # Verify correct binary
```

## Hook-based usage

All other commands are automatically rewritten by the PreToolUse hook.
Example: `git status` → `rtk git status` (transparent, 0 extra tokens).
