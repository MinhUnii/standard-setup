# clone
git clone --bare <url> .bare

# let git tracking all branch from remote
git --git-dir=.bare config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"

# download all infor for every branch
git --git-dir=.bare fetch origin

# Tao worktree cho nhanh main
git --git-dir=.bare worktree add main