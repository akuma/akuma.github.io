---
title: 使用 tmux
date: 2017-02-24 15:53:46
tags: linux tmux
---

## 配置 ssh agent forwarding

### .ssh/rc

```sh
#!/bin/bash

# http://techblog.appnexus.com/2011/managing-ssh-sockets-in-gnu-screen/
# https://gist.github.com/martijnvermaat/8070533
# http://stackoverflow.com/questions/21378569/how-to-auto-update-ssh-agent-environment-variables-when-attaching-to-existing-tm

# Fix SSH auth socket location so agent forwarding works with screen.
if test "$SSH_AUTH_SOCK" ; then
    ln -sf $SSH_AUTH_SOCK ~/.ssh/ssh_auth_sock.$(hostname)
fi

# Don't break x11 Forwarding:
# Taken from the sshd(8) manpage.
if read proto cookie && [ -n "$DISPLAY" ]; then
        if [ `echo $DISPLAY | cut -c1-10` = 'localhost:' ]; then
                # X11UseLocalhost=yes
                echo add unix:`echo $DISPLAY |
                    cut -c11-` $proto $cookie
        else
                # X11UseLocalhost=no
                echo add $DISPLAY $proto $cookie
        fi | xauth -q -
fi
```

### .tmux.conf

```
# .tmux.conf

# -- general -------------------------------------------------------------------

set -g default-terminal 'screen-256color' # colors!
set -s quiet on                           # disable various messages
set -s escape-time 0                      # fastest command sequences
set -sg repeat-time 600                   # increase repeat timeout
setw -g xterm-keys on

set -g prefix2 C-a                        # GNU-Screen compatible prefix
bind C-a send-prefix -2

set -q -g status-utf8 on                  # expect UTF-8 (tmux < 2.2)
setw -q -g utf8 on

set -g history-limit 5000                 # boost history

# reload configuration
bind r source-file ~/.tmux.conf \; display 'tmux.conf reloaded!'

# vi style copy/paste
setw -g mode-keys vi
bind Escape copy-mode
#unbind p
#bind p paste-buffer        # doesn't work
bind -t vi-copy 'v' begin-selection
#bind -t vi-copy 'y' copy-selection
#bind -t vi-copy y copy-pipe 'reattach-to-user-namespace pbcopy' # For macOS


# -- display -------------------------------------------------------------------

set -g base-index 1         # start windows numbering at 1
setw -g pane-base-index 1   # make pane numbering consistent with windows

setw -g automatic-rename on # rename window to reflect current program
set -g renumber-windows on  # renumber windows when a window is closed

set -g set-titles on                        # set terminal title
set -g set-titles-string '#h ❐ #S ● #I #W'

set -g display-panes-time 800 # slightly longer pane indicators display time
set -g display-time 1000      # slightly longer status messages display time

set -g status-interval 10     # redraw status line every 10 seconds

# clear both screen and history
bind -n C-l send-keys C-l \; run 'sleep 0.05 && tmux clear-history'

# activity
set -g monitor-activity on
set -g visual-activity on

# set the status line style
set -g status-left-length 40
set -g status-left '#[fg=green][#S] #[fg=red]w:#I #[fg=yellow]p:#P'
set -g status-right '#h %D %H:%M'
set -g status-justify centre

# set the status line's colors
set -g status-fg green
set -g status-bg black

# set the color of the window list
setw -g window-status-fg black
setw -g window-status-bg cyan
setw -g window-status-attr dim

# set colors for the active window
setw -g window-status-current-fg white
setw -g window-status-current-bg red
setw -g window-status-current-attr bright

# pane colors
set -g pane-border-fg default
set -g pane-border-bg default
set -g pane-active-border-fg green
set -g pane-active-border-bg default


# -- navigation ----------------------------------------------------------------

# find session
bind C-f command-prompt -p find-session 'switch-client -t %%'

# pane navigation
bind -r h select-pane -L  # move left
bind -r j select-pane -D  # move down
bind -r k select-pane -U  # move up
bind -r l select-pane -R  # move right
bind > swap-pane -D       # swap current pane with the next one
bind < swap-pane -U       # swap current pane with the previous one

# pane resizing
bind -r H resize-pane -L 2
bind -r J resize-pane -D 2
bind -r K resize-pane -U 2
bind -r L resize-pane -R 2

# pane splitting
bind | split-window -h
bind - split-window -v

# pane synchronization
bind C-z setw synchronize-panes

# window navigation
unbind n
unbind p
bind -r C-n next-window     # select next window
bind -r C-p previous-window # select previous window
bind Tab last-window        # move to last active window

# mouse support
#set -g mode-mouse off
set -g mouse-select-pane on
set -g mouse-resize-pane on
set -g mouse-select-window on

# ssh agent forwarding for tmux
set -g update-environment -r
setenv -g SSH_AUTH_SOCK $HOME/.ssh/ssh_auth_sock.$HOSTNAME
```
