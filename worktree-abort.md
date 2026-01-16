---
description: Abandon current worktree without merging
---

# Abort Worktree

Removes the current worktree and deletes the branch WITHOUT merging changes.

⚠️ **All uncommitted and committed changes in this worktree will be lost!**

```bash
set -e

CURRENT_PATH=$(pwd)
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
GIT_DIR=$(git rev-parse --git-dir)
GIT_COMMON_DIR=$(git rev-parse --git-common-dir)

# Detect default branch
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@') || DEFAULT_BRANCH="main"
if [ -z "$DEFAULT_BRANCH" ]; then
  DEFAULT_BRANCH="main"
fi

# Check if we're in a worktree
if [ "$GIT_DIR" = "$GIT_COMMON_DIR" ]; then
  echo "❌ Not in a worktree!"
  echo ""
  echo "Run this from inside a worktree to abort it."
  echo "Use /worktree-list to see active worktrees"
  exit 1
fi

if [ "$CURRENT_BRANCH" = "$DEFAULT_BRANCH" ]; then
  echo "❌ Cannot abort the main worktree"
  exit 1
fi

FEATURE_BRANCH="$CURRENT_BRANCH"
MAIN_REPO=$(cd "$GIT_COMMON_DIR/.." && pwd)

echo "⚠️  About to ABANDON worktree: $FEATURE_BRANCH"
echo ""
echo "   Path: $CURRENT_PATH"
echo "   Branch: $FEATURE_BRANCH"
echo ""
echo "   All changes will be LOST!"
echo ""

# Switch to main repo
cd "$MAIN_REPO"
git checkout "$DEFAULT_BRANCH"

# Force remove worktree
echo "🗑️  Removing worktree..."
git worktree remove "$CURRENT_PATH" --force 2>/dev/null || true

# Force delete branch
echo "🗑️  Deleting branch..."
git branch -D "$FEATURE_BRANCH" 2>/dev/null || true

echo ""
echo "✅ Worktree '$FEATURE_BRANCH' abandoned"
echo ""
echo "📍 You are now in: $MAIN_REPO on $DEFAULT_BRANCH"
```
