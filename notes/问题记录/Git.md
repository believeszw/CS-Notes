# Git

## sign_and_send_pubkey: signing failed: agent refused operation

```shell
ssh-add
```

## error: cannot lock ref 'refs/remotes/origin/jira/MS-14216/arsenal': 'refs/remotes/origin/jira/MS-14216' exists; cannot create 'refs/remotes/origin/jira/MS-14216/arsenal'
pull 代码时遇到上面的错误，In most cases, users should run git gc, which calls git prune. See the section "NOTES", below.

```shell
git pull --prune
```
