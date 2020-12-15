markdown notes:
=== Getting more done with fewer keystrokes.
 ===
== Prerequisites ==
New to Linux/Vim? Here are some tutorials, tips and even a game: [[https://www.internalfb.com/intern/dex/preparing-for-commandline-efficiency/|getting comfortable]]

== Motivation ==
  - health (avoid RSI/carpal-tunnel syndrome)
  - efficiency/sustainability
  - passion/compulsive-coding require special measures

== Prefer keyboard to mouse, when possible ==
  - excessive mouse use is bad for your wrists/hands
  - besides, the keyboard is more efficient/faster than mouse

== Save state; avoid disconnect ==
  - use mosh (mobile-friendly ssh):
https://www.internalfb.com/intern/wiki/index.php/Mosh
  - use lighthouse@home: sometimes faster and lower-latency than VPN,
but most importantly, more convenient. Request one from the [[https://www.internalfb.com/buy/store/additional_devices/lighthouse_at_home | Enterprise Store]] and then join this internal group: [[https://fb.facebook.com/groups/rapusers/]]

== Run `screen` or `tmux` on your devserver:==
Each is a terminal mutiplexer. If you use neither already, `tmux` may be the better choice, since it has better iTerm integration. For mouse-scrolling through tmux history with mosh, see [[https://fb.facebook.com/groups/hack.of.the.day/permalink/946239195424696/|this post in the HOTD group]].

Here is a very good screen vs. tmux comparison: https://wtanaka.com/node/8136

Why does using a terminal multiplexer help?

      -- upon laptop crash/reboot, you lose no state from the multiplexed terminal sessions on your devserver
      -- you can search scrolled screen content quickly and **mechanically**
      -- you can easily disconnect/reconnect a "session" -- no more HUPs
      -- e.g., you're in the middle of a 20-minute run that you did not protect via `nohup`, when you need to go to a meeting or catch a bus.
With neither mosh nor screen/tmux, you'd lose the ssh connection.

== Tmuxinator ==
[[https://github.com/tmuxinator/tmuxinator | Tmuxinator]] allows you to easily and declaratively define what your tmux session should look like in terms of panes and windows.

== Emacs + mosh + Iterm2 user?==
You can [[https://fb.facebook.com/groups/fbemacs/permalink/1294177860627429/ | make "Meta" work]] on your devserver.

<div id="history"></div>
== Tell tools to save your history ==
- bash:

      echo HISTSIZE=130000 HISTFILESIZE=-1 >> ~/.bashrc
      # date and time for each entry in the history
      echo HISTTIMEFORMAT="%d/%m/%y %T " >> ~/.bashrc
      # don't put duplicate lines in the history
      echo HISTCONTROL=ignoredups:ignorespace >> ~/.bashrc

- zsh ([[https://www.internalfb.com/intern/wiki/Development_Environment/zsh/ | getting started with zsh at FB]]):

      echo HISTSIZE=130000 SAVEHIST=130000 >> ~/.zshrc

- screen: make it save more lines of "scrollback":

      echo defscrollback 30000 >> ~/.screenrc

- tmux:

      echo set -g history-limit 30000 >> ~/.tmux.conf
      # fix vim colorscheme in tmux
      export TERM=screen-256color  >> ~/.bashrc

- gdb:

      echo set history save on >> ~/.gdbinit
      echo set history size unlimited >> ~/.gdbinit

== Avoid unnecessary prompts (avoid slow-down and risk of wrong answer) ==
Use --backup options to cp, ln, mv:

      alias cp='cp --backup=numbered'
      alias ln='ln --backup=numbered'
      alias mv='mv -f --backup=numbered'

This option is only supported by GNU coreutils, not by the BSD versions that come with OS X. It’s possible to install GNU coreutils, e.g. with `brew install coreutils`. To override the builtins, follow the instructions printed by `brew info coreutils`.

Use the "env " prefix to avoid the alias and instead invoke the bare program.

== Reuse! don't rewrite/retype even simple things ==
  - save lots of history: shell, gdb, screen/tmux
  - ensure that it is saved to disk between sessions
    -- for bash/zsh, ensure that each new command is appended to a
global history, then each new shell gets everything
    -- you can even ensure that a command typed in one terminal
becomes part of the history in every other one (for zsh, just run `fc -IR`; for bash, follow these instructions:
    [[https://news.ycombinator.com/item?id=3755276|Keep bash history in sync on disk and between multiple terminals]]

== Customize your shell prompt to avoid common errors ==
  - bash/zsh: display colored/signal-aware exit status in prompt;
see the my_prompt function in `/mnt/homedir/meyering/.zshrc`
  - display version-control-agnostic state in prompt: https://fburl.com/scmprompt
  - bash: when building larger pipelines it might be useful showing the
exit status of each part of the pipeline in your prompt. For this you can
use the ${PIPESTATUS[@]} array:

  export PS1="[Exit: \[\033[1;31m\]\${PIPESTATUS[@]/#0/\[\033[0m\]\[\033[1;32m\]0\[\033[1;31m\]}\[\033[0m\]] $ "

- Here is an example putting together the above (see [[https://phabricator.intern.facebook.com/P58619908|bashrc]] for PS1). The red exit status immediately shows that the command exited abnormally, even though $? will be 0.
{px/bQkJ}

- bash: There is a lot of customization available to create your `PS1` export. [[http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/bash-prompt-escape-sequences.html | The documentation]] goes over all modifiers, but you can also graphically construct `PS1` using [[http://ezprompt.net/ | ezprompt.net]] or [[http://bashrcgenerator.com/ | bashrcgenerator.com]] and then paste it into `~/.bashrc`.

<div id="search"></div>
== Use incremental search (^R, aka Control-R) ==
  - in your editor, of course
  - when searching shell history
  - when searching through contents of scrolled screen log
  - in gdb
  - in any other readline-based tool (they all honor `~/.inputrc`)

If you use vi/vim, run this and restart any existing bash shell:

    echo set editing-mode vi >> ~/.inputrc

If you use zsh in vi mode, add this to ensure ^R still does what we expect:

    echo 'bindkey "^R" history-incremental-search-backward' >> ~/.zshrc

== Some custom key bindings ==
`~/.inputrc` is the config file for readline, which is used in many interactive programs such as bash and gdb. The following key bindings will make your life easier.

Remember to **restart bash** after modifying `~/.inputrc`.

=== Cycle through commands to a partial match ===
Add the following two lines to your `~/.inputrc`.

      echo '"\e[A": history-search-backward' >> ~/.inputrc
      echo '"\e[B": history-search-forward' >> ~/.inputrc

if you use zsh, add to `~/.zshrc` instead, 

    echo 'bindkey "^[[A" history-search-backward' >> ~/.zshrc
    echo 'bindkey "^[[B" history-search-forward' >> ~/.zshrc

Type in a few chars of a command in history and use arrow (up/down) keys to cycle through all commands that start with the typed in chars.

=== Delete word for bash command line (^W, aka Control-W) ===
      echo '"\C-w": backward-delete-word' >> ~/.inputrc

== Tips/pointers ==
- rlor: why needed, how it works `~meyering/bin/rlor`
- in vim/vi, when advancing through a line of existing text, say to fix a typo,
don't use space or arrow keys.  Instead, use W or B, to advance or back up
a "big" word at a time.  Better still, use 'f' or 'F' to search forward or
backwards for target character.  Or best (sometimes), use fwd/reverse
incremental search.  For when you do use repeated cursor motion, ensure
that key-repeat kicks in sooner and repeats faster.
- in emacs and readline programs with default editing-mode, you can use C-left/C-right to jump from a word to another, and C-S/C-r for forward/backward incremental search.
- look at `~meyering/.vimrc`

=== Alternative to standard bash search ===
Searching through bash history is human, making it a pleasure is divine! 

[[https://github.com/junegunn/fzf | Fzf]] is a great tool to search through your shell history (have a look at the git repo for some screenshots). There are some pages you could refer to if you want to try it:

- [[https://www.internalfb.com/intern/wiki/FZF/#installation-on-your-dev]]
- [[https://www.internalfb.com/intern/wiki/Improving-your-workflow/#vundle]]
- [[https://www.internalfb.com/intern/wiki/Users/heucke/Fuzzy_Everywhere/]]


<div id="pcre"></div>
== Regular expressions: PCRE vs. ERE/BRE ==
Conciseness matters:

    - `\d` === `[0-9]`
    - `\w` === word-constituent
    - `\W` === non-word-constituent
    - `\b` === zero-width word-boundary (like \< and \>)
    - `\s` === whitespace
    - `\S` === non-whitespace

=== Get to know regular expressions **well** (esp. EREs and PCREs) ===
Which of these do you prefer?
        - `/'.*?'/`
        - `/'[^']*'/` (e.g., write a C-comment-remover: easy with PCRE's `.*?` )

=== essential command line tools ===
- bourne shell scripting: for hacky-small tools as well as command line use
learn how to use && and || -- much more concise than if/then/else/fi
- find + xargs
- sed, perl, grep (-A,-B,-C,-v), awk for extracting columns

=== cool, lesser-known tools ===
- glark
- GNU parallel

== Examples ==
- git-log-uniq-etc pipeline for finding reviewers, now known
as `~meyering/.zsh/find-reviewers`, but superceded by findpies(1)
- seq + sed
- show how lack of a VC-aware prompt can cause grief

== take-away ==
  - use mosh
  - use screen|tmux
  - save lots of shell history between sessions
  - increase screen/tmux scrollback history
  - save gdb command history
  - reuse commands from history; use reverse-incremental-search, don't retype
  - make your shell prompt display VCS state and $?
  - learn how to use PCRE regexps well
  - embrace (or at least "do not deride") perl on the command line
  - learn Vim or Emacs well
  - use Vim or Emacs commands to edit command lines, not arrow keys

== Links and References ==
  - General command-line tips: https://github.com/jlevy/the-art-of-command-line
  - An interactive cmd-line teaching tool: https://cmdchallenge.com
  - Perl one-liners: http://www.catonmat.net/blog/introduction-to-perl-one-liners/
  - Don't copy/paste from untrusted web pages: https://thejh.net/misc/website-terminal-copy-paste
  - Regex learning tools:
    -- https://regex101.com/
    -- https://regexcrossword.com/
    -- http://www.regexr.com/
    -- http://rampion.github.io/RegHex/
    -- https://play.google.com/store/apps/details?id=se.uddevalla.danielsson.simon.rex
