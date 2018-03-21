---
layout: post
title: 'Tmux cheat sheet'
author: Christopher Batey
comments: true
tags:
 - tmux
---

Windows (tabs):

`c` create a window
`,` name a winddow
`n` next window
`p` previous window
`1-9` jump to window
`w` select window
`&` kill window

Panes:

`"` split horizontal
`%` split vertical
`o` next pane
`x` kill pane

Reshaping panes:

In .tmux.conf

```
bind -r C-h resize-pane -L                                                                                                                                                                                           
bind -r C-j resize-pane -D                                                                                                                                                                                           
bind -r C-k resize-pane -U                                                                                                                                                                                           
bind -r C-l resize-pane -R  
```


Misc

`d` detatch the session

`t` get the time



