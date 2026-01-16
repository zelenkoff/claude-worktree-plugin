---
description: Merge current worktree to main and cleanup
---

# Finish Worktree

Commits any changes, merges to main branch, and cleans up the worktree.

```bash
set -e

# Get current info
CURRENT_PATH=$(pwd)
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
GIT_DIR=$(git rev-parse --git-dir)
GIT_COMMON_DIR=$(git rev-parse --git-common-dir)

# Detect default branch (main or master)
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@') || DEFAULT_BRANCH="main"
if [ -z "$DEFAULT_BRANCH" ]; then
  DEFAULT_BRANCH="main"
fi

# Check if we're in a worktree
if [ "$GIT_DIR" = "$GIT_COMMON_DIR" ]; then
  echo "❌ Not in a worktree!"
  echo ""
  echo "Run this command from inside a worktree directory."
  echo "Current location: $CURRENT_PATH"
  echo "Current branch: $CURRENT_BRANCH"
  echo ""
  echo "Use /worktree-list to see active worktrees"
  exit 1
fi

# Don't run on default branch
if [ "$CURRENT_BRANCH" = "$DEFAULT_BRANCH" ]; then
  echo "❌ Already on $DEFAULT_BRANCH, nothing to merge"
  exit 1
fi

FEATURE_BRANCH="$CURRENT_BRANCH"

echo "🔄 Finishing worktree: $FEATURE_BRANCH"
echo ""

# Check for uncommitted changes
if ! git diff --quiet HEAD 2>/dev/null || ! git diff --cached --quiet 2>/dev/null; then
  echo "📝 Committing uncommitted changes..."
  git add -A
  git commit -m "chore: final changes before merge" || true
fi

# Get main repo path
MAIN_REPO=$(cd "$GIT_COMMON_DIR/.." && pwd)

# Count commits to potentially squash
COMMIT_COUNT=$(git rev-list --count "$DEFAULT_BRANCH".."$FEATURE_BRANCH" 2>/dev/null || echo "0")

echo "📊 Found $COMMIT_COUNT commit(s) to merge"
echo "📂 Switching to main repo: $MAIN_REPO"
echo ""

cd "$MAIN_REPO"

# Checkout default branch and pull latest
echo "🌿 Checking out $DEFAULT_BRANCH..."
git checkout "$DEFAULT_BRANCH"
git pull --rebase 2>/dev/null || true

# Merge the feature branch (squash if multiple commits)
echo "🔀 Merging $FEATURE_BRANCH into $DEFAULT_BRANCH..."

if [ "$COMMIT_COUNT" -gt 1 ]; then
  echo "   (squashing $COMMIT_COUNT commits)"
  if git merge --squash "$FEATURE_BRANCH"; then
    git commit -m "feat: $FEATURE_BRANCH" --no-edit || git commit --no-edit
  else
    echo ""
    echo "❌ Merge conflict detected!"
    echo ""
    echo "📋 Conflicting files:"
    git diff --name-only --diff-filter=U | sed 's/^/   - /'
    echo ""
    echo "🔧 To resolve:"
    echo "   1. Fix conflicts in the files above"
    echo "   2. git add <resolved-files>"
    echo "   3. git commit"
    echo ""
    echo "🧹 After committing, cleanup:"
    echo "   git worktree remove \"$CURRENT_PATH\" --force"
    echo "   git branch -D \"$FEATURE_BRANCH\""
    echo ""
    echo "⏪ Or abort the merge:"
    echo "   git merge --abort"
    echo "   cd \"$CURRENT_PATH\""
    exit 1
  fi
else
  if ! git merge "$FEATURE_BRANCH" --no-edit; then
    echo ""
    echo "❌ Merge conflict detected!"
    echo ""
    echo "📋 Conflicting files:"
    git diff --name-only --diff-filter=U | sed 's/^/   - /'
    echo ""
    echo "🔧 To resolve:"
    echo "   1. Fix conflicts in the files above"
    echo "   2. git add <resolved-files>"
    echo "   3. git commit"
    echo ""
    echo "🧹 After committing, cleanup:"
    echo "   git worktree remove \"$CURRENT_PATH\" --force"
    echo "   git branch -D \"$FEATURE_BRANCH\""
    echo ""
    echo "⏪ Or abort the merge:"
    echo "   git merge --abort"
    echo "   cd \"$CURRENT_PATH\""
    exit 1
  fi
fi

echo "✅ Merge successful!"
echo ""

# Remove worktree
echo "🗑️  Removing worktree..."
git worktree remove "$CURRENT_PATH" --force 2>/dev/null || true

# Delete branch
echo "🗑️  Deleting branch $FEATURE_BRANCH..."
git branch -D "$FEATURE_BRANCH" 2>/dev/null || true

echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  ✅ Done!                                   │"
echo "│                                             │"
echo "│  Branch merged: $FEATURE_BRANCH             │"
echo "│  Worktree removed                           │"
echo "│  Current branch: $DEFAULT_BRANCH            │"
echo "└─────────────────────────────────────────────┘"
echo ""
echo "📍 Location: $MAIN_REPO"
```
