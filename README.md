# claude-worktree

Simple git worktree management for running multiple Claude Code agents in parallel.

## Why?

When you run multiple Claude Code instances on the same codebase, they step on each other's changes. Git worktrees let you create isolated copies of your repo where each agent can work independently.

This plugin gives you 4 simple commands to manage that workflow without thinking about git internals.

## Installation

```bash
/plugin marketplace add eugenezelenkov/claude-worktree
/plugin install worktree
```

## Commands

| Command | Description |
|---------|-------------|
| `/worktree-start <name>` | Create a new worktree for a feature |
| `/worktree-finish` | Merge worktree to main and cleanup |
| `/worktree-abort` | Abandon worktree without merging |
| `/worktree-list` | List all active worktrees |

## Workflow

### Single Agent

```
/worktree-start my-feature        # Creates ./trees/my-feature
cd trees/my-feature && claude

# ... do your work ...

/worktree-finish                   # Merges to main, cleans up
```

### Multiple Parallel Agents

Open multiple terminals:

```bash
# Terminal 1
/worktree-start feature-auth
cd trees/feature-auth && claude
# Work on authentication...

# Terminal 2  
/worktree-start feature-payments
cd trees/feature-payments && claude
# Work on payments...

# Terminal 3
/worktree-start feature-emails
cd trees/feature-emails && claude
# Work on email notifications...
```

Each agent works in complete isolation. When done:

```bash
# In each worktree when ready:
/worktree-finish
```

### Abandoning Work

If you want to discard changes and delete the worktree:

```bash
/worktree-abort    # Deletes worktree + branch, no merge
```

## Directory Structure

```
your-project/
├── src/
├── ...
└── trees/              # Auto-created, auto-gitignored
    ├── feature-auth/       # Worktree 1
    ├── feature-payments/   # Worktree 2
    └── feature-emails/     # Worktree 3
```

## What Happens Under the Hood

**`/worktree-start <name>`:**
1. Creates `./trees/` directory (adds to .gitignore)
2. Runs `git worktree add ./trees/<name> -b <name>`
3. You now have an isolated copy with its own branch

**`/worktree-finish`:**
1. Commits any uncommitted changes
2. Switches to main repo
3. Merges (squashes if multiple commits)
4. Removes worktree directory
5. Deletes the feature branch

**`/worktree-abort`:**
1. Switches to main repo  
2. Force removes worktree
3. Force deletes branch
4. All changes are lost

## Handling Merge Conflicts

If `/worktree-finish` encounters conflicts with changes merged from other worktrees:

```bash
# The command will show you which files conflict
❌ Merge conflict detected!

📋 Conflicting files:
   - src/app.js
   - src/config.js

# Option 1: Resolve the conflicts
# 1. Edit the conflicting files to resolve conflicts
# 2. Stage the resolved files:
git add src/app.js src/config.js
# 3. Commit the merge:
git commit
# 4. Manual cleanup:
git worktree remove ./trees/feature-name --force
git branch -D feature-name

# Option 2: Abort the merge
git merge --abort
cd ./trees/feature-name
# Fix conflicts in your worktree, or coordinate with other features
```

## Tips

- Keep features independent to avoid merge conflicts
- Finish worktrees in logical order (foundational changes first)
- Run `/worktree-list` to see all active worktrees
- Each worktree shares git history but has isolated files
- When conflicts occur, you'll get clear instructions with file names

## License

MIT
