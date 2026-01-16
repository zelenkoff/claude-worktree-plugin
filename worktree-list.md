---
description: List all active git worktrees
---

# List Worktrees

```bash
echo "📂 Active Worktrees"
echo "==================="
echo ""

git worktree list --porcelain | while read line; do
  case "$line" in
    worktree\ *)
      path="${line#worktree }"
      ;;
    HEAD\ *)
      head="${line#HEAD }"
      ;;
    branch\ *)
      branch="${line#branch refs/heads/}"
      echo "  📁 $path"
      echo "     └── 🌿 $branch"
      echo ""
      ;;
  esac
done

# Count worktrees
TOTAL=$(git worktree list | wc -l)
FEATURES=$((TOTAL - 1))

echo "─────────────────────────────"
echo "Total: $TOTAL worktree(s) ($FEATURES feature branch(es))"
echo ""

if [ "$FEATURES" -eq 0 ]; then
  echo "💡 Create a new worktree:"
  echo "   /worktree-start <feature-name>"
else
  echo "💡 Commands:"
  echo "   /worktree-start <name>  Create new worktree"
  echo "   /worktree-finish        Merge & cleanup (from inside worktree)"
  echo "   /worktree-abort         Abandon without merging"
fi
```
