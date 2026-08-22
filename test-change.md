# hi i made some changes to learn worktrees

### What are worktrees? 
Worktrees allow you to make changes while in another branch.
Instead of stashing your changes and switching branches, 
where there is a risk you'll forget the stash you made.
Why not use worktrees instead? + it's also useful for working with AI agents

Create a worktree 
```bash
git worktree add <name-of-worktree> <derived-branch>
```

Switch to a new branch and create worktree 
```bash 
git worktree add <name-of-worktree> -b <new-branch>
```

Enter a worktree
```bash 
cd ./<name-of-worktree> # this will make you change branches, but the worktree exists in your working directory
```

From there, inside of the worktree you can do: 
- Commit changes 
- Make different worktrees 
- Push changes 
- Pull changes 
```
```
```
```
