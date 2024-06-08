## Creating and pushing a new repository

```bash
git config --global user.email your@email.com
git config --global user.name your-name
git init -b main
git add .
git commit -m "commit-message"
curl -H "Authorization: token your-github-token(development-tools..)" https://api.github.com/user/repos -d '{"name":"name-you-want"}'
git remote add origin <repo-url>
git push -u origin main
```

## Push existing repository

```bash
git remote add origin <repo-url>
git branch -M main
git push -u origin main
```

## Create new branch and checkout to it

```bash
git branch <branch-name>
git checkout <branch-name>
```

## Merge from branch into main/master

```bash
git checkout main
git pull origin main
git merge <branch-name>
git push origin main
```

## Changes were made to the repo and you cant commit

```bash
git add .
git stash
git pull --rebase
git stash pop
git push
```

## Pull from main into the development branch

```bash
git checkout <branch-name>
git fetch origin
git merge origin/main
```
