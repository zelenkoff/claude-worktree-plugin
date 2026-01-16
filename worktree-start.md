---
description: Create a new git worktree for parallel development
---

# Create Worktree for: $ARGUMENTS

Create an isolated worktree to work on a feature without affecting main.

```bash
set -e

FEATURE_NAME="$1"
if [ -z "$FEATURE_NAME" ]; then
  echo "❌ Usage: /worktree-start <feature-name>"
  echo ""
  echo "Example: /worktree-start add-oauth"
  exit 1
fi

# Sanitize feature name (replace spaces with dashes, remove special chars)
FEATURE_NAME=$(echo "$FEATURE_NAME" | tr ' ' '-' | tr -cd '[:alnum:]-_')

TREES_DIR="./trees"
WORKTREE_PATH="$TREES_DIR/$FEATURE_NAME"

# Create trees directory if needed
mkdir -p "$TREES_DIR"

# Add trees to gitignore if not already there
if [ -f .gitignore ]; then
  grep -q "^trees/$" .gitignore 2>/dev/null || echo "trees/" >> .gitignore
else
  echo "trees/" > .gitignore
fi

# Check if worktree already exists
if [ -d "$WORKTREE_PATH" ]; then
  echo "⚠️  Worktree '$FEATURE_NAME' already exists"
  echo ""
  echo "📂 Path: $WORKTREE_PATH"
  echo ""
  echo "To continue working:"
  echo "   cd $WORKTREE_PATH"
  exit 0
fi

# Get current branch as base
BASE_BRANCH=$(git rev-parse --abbrev-ref HEAD)

# Ensure we have latest changes
git pull --rebase 2>/dev/null || true

# Create worktree with new branch
echo "🌿 Creating worktree '$FEATURE_NAME' from '$BASE_BRANCH'..."
git worktree add "$WORKTREE_PATH" -b "$FEATURE_NAME"

echo ""
echo "✅ Worktree created!"
echo ""
echo "┌─────────────────────────────────────────────┐"
echo "│  📂 Path:   $WORKTREE_PATH"
echo "│  🌿 Branch: $FEATURE_NAME"
echo "│  📍 Base:   $BASE_BRANCH"
echo "└─────────────────────────────────────────────┘"
echo ""
echo "👉 Next: cd $WORKTREE_PATH && claude"
echo ""
echo "When done, run /worktree-finish from inside the worktree"
```
