---
layout: post
title: Leverage command line aliases
published_at: 2016-10-15
collection: tricks
index: 1
intro: |
      Master your command line aliases to become effective at day-to-day tasks.
--- 

# Leverage command line aliases

Master your command line aliases to become effective at day-to-day tasks.  
Add Bash aliases for repetitive tasks to make your life easier.

Useful aliases I use:

```bash
alias c='composer'
alias a='php artisan'
alias s='php symfony'  
alias g='gulp'
alias t='tig --all'
```

With those aliases you can run CLI tasks in a much shorter way.  
Some of examples:  

```
$ s help
$ c install
$ g watch
```

## How to add aliases

1. Edit your `~/.bash_aliases` file (create it if you don't have one).
2. Add aliases from above.
3. Re-login to your OS account.
