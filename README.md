# iBhagwan's (n)vim cheatsheet

\*\*This cheatsheet was inspired by [Hackjutsu's Vim cheatsheet](https://github.com/hackjutsu/vim-cheatsheet) and Laurent Gregoire's excellent [Vim Quick Reference Card](http://tnerual.eriogerg.free.fr/vimqrc.html).

I've been using vi for over 20 years but always limited it's use for basically only editing \*nix system config files, recently I (re-)discovered (n)vim as the magical editor that it is and decided to make it my main editor for coding, writing markdown and the likes so down the rabbit hole I went. The more I researched the more I fell in love with the software, the below is the accumulation of all my notes and findings.

The cheatsheet assumes at least minimal understanding of motions and operators, to better understand motions, operators and how it all comes together in what is referred to as "the vim language" I highly recommend reading [Jim Denis's stackoverflow answer: Your problem with Vim is that you don't grok vi](https://gist.github.com/nifl/1178878).

Note that everything below works with vanilla vim/nvim and does not require the use of any plugins. I am not opposed to plugins and even use a few which I find extremely helpful but as a philosophy I try to use as much built-in functionality (with minimal keymap changes) as I can before turning to plugins. As you can see below there is **so much** you can do with vanilla Vim that I decided to leave plugins out of the scope of this document. If you'd like to take a look at my custom mappings and plugins, refer to my nvim lua configuration at [nvim-lua](https://github.com/ibhagwan/nvim-lua).

If you'd like to take your Vim to the next level, I highly recommend watching all of [Drew Neil's VimCasts](http://vimcasts.org/episodes/page/8/) they are absolutely wonderful and will blow your mind. When you're ready to jump in the water and test your skills [Vimgolf](http://www.vimgolf.com) is fantastic exercise which will cement all your Vim knowledge and add new tricks to your arsenal. Download [Vimgolf client from here](https://github.com/igrigorik/vimgolf) or it's [lesser known python client by Daniel Stein](https://github.com/dstein64/vimgolf) if you wish to avoid the ruby dependencies.

**Thanks to [/u/-romainl-](https://www.reddit.com/user/-romainl-/) for [his great feedback](https://www.reddit.com/r/vim/comments/gha79v/i_wrote_an_advanced_comprehensive_cheatsheet_for/fq7qeop?utm_source=share&utm_medium=web2x) correcting previous inaccuracies.**


### Table of contents:

- [Saving & Exiting Vim](#saving-exiting-vim)
- [Navigation](#navigation)
    + [Movement](#movement)
    + [Scrolling](#scrolling)
- [Editing](#Editing)
    + [Insert & Exit](#insert-exit)
    + [Undo & Repeat](#undo-repeat)
    + [Basic Editing](#basic-editing)
    + [INSERT mode](#insert-mode)
    + [Cut, copy & paste](#cut-copy-and-paste)
- [VISUAL mode](#visual-mode)
    + [Selections](#selections)
    + [Operators](#operators)
- [Text Objects](#text-objects)
- [Search & Replace](#search-replace)
    + [REGEX Examples](#regex-examples)
    + [Multiple Files](#multiple-files)
- [Macros](#macros)
    + [Recursive Macros](#recursive-macros)
- [Marks](#marks)
- [Files and Windows](#files-and-windows)
- [Tabs](#tabs)
- [Terminal](#term)
- [Spellcheck](#spellcheck)
- [Misc Commands](#misc-commands)
- [Ctrl-R and the Expression Register](#ctrl-r-and-the-expression-register)
- [Comparing buffers with vimdiff](#comparing-buffers-with-vimdiff)
- [Folding](#folding)

## <a id="saving-exiting-vim">Saving & Exiting Vim</a>
```vim
:w              " write the current file
:w {file}       " write to {file}
:wq {file}      " write to {file} and quit
:saveas {file}  " write to {file}
:update {file}  " write only if buffer was modified
:w !sudo tee %  " write the current file using sudo
:wq :x ZZ       " write the current file and quit
:q              " quit (fails if there are unsaved changes)
:q! ZQ          " quit and throw away unsaved changes
:cq             " quit Vim with exit code
```

**NOTE:** the `:cq` command is useful in situations when we need to exit-cancel, for example when running `<Esc>v` in bash shell `set -o vi`, when exiting using `:cq` the command won't be executed automatically. Another example is when exiting the output window of `git rebase`, `:cq` will cancel the rebase operation.

## <a id="navigation">Navigation</a>
### <a id="movement">Movement</a>
```vim
    k           " up
h       l       " left, right
    j           " down
```
```vim
+ -             " first non-blank char line above, below
H M L           " [H]igh, [M]iddle, [L]ow line of the window
{n}H {n}L       " line {n} from the top, bottom of the window
b w             " beginning of word left, right (punctuation considered words)
B W             " beginning of WORD left, right (spaces separate words)
e ge            " end of word left, right (punctuation considered words)
E gE            " end of WORD left, right (spaces separate words)
0 g0            " start of the line, visual-line
^ g^            " first non-blank character of the line, visual-line
$ g$            " last character of the line, visual-line (include <CR>)
_ g_            " first, last non-blank character of the line (not including <CR>)
{n}_            " down {n-1} lines on first non-blank character
gm              " middle of the line
{n}gg {n}G      " goto line {n}, default first, last line of buffer
:{n}<CR>        " goto line {n}, (ex mode)
{n}%            " jump to buf % (e.g. 50% jump to middle of buffer)
{n}|            " jump to screen column {n} of current line
f{c} F{c}       " next, previous occurrence of character {c}
t{c} T{c}       " before ([t]ill) next, previous occurrence of {c}
; ,             " repeat last fFtT in the same, opposite direction
( )             " previous, next sentence (jumps after the next '.' or EOL)
{ }             " previous, next paragraph (jumps to next empty line)
%               " jump to matching parenthesis ([{}])
[[ ]]           " backward, forward start section
[] ][           " backward, forward end of section
[( ])           " backward, forward unclosed (, )
[{ ]}           " backward, forward unclosed {, }
```
**Note:** When navigating lines with `hjkl^$`, if a line is too long and is wrapped, pressing `j` will move down to the next line even if the current line is wrapping 2 or more lines, to go down 1 "visual" line use `gj` instead. The `g` prefix works the same for other navigation commands `hjkl^$` (`g^` and `g$` for start and end of visual line). A very useful mapping is to map `<up><down>` or `jk` to move by visual lines as long as they aren't prefixed with {count} so we can still use up and down motions (e.g. `10j`) for whole lines:

```
nnoremap <expr> <Down> (v:count == 0 ? 'g<down>' : '<down>')
vnoremap <expr> <Down> (v:count == 0 ? 'g<down>' : '<down>')
nnoremap <expr> <Up> (v:count == 0 ? 'g<up>' : '<up>')
vnoremap <expr> <Up> (v:count == 0 ? 'g<up>' : '<up>')
```

### <a id="scrolling">Scrolling</a>
```vim
zz or z.        " center screen on cursor
zb or z-        " scroll the screen so the cursor is at the (b)ottom
zt or z<cr>     " scroll the screen so the cursor is at the (t)op
zh zl           " scroll one character to the left, right
zH zL           " scroll half screen to the left, right
<ctrl-b>        " move back one full screen
<ctrl-f>        " move forward one full screen
<ctrl-d>        " move forward 1/2 a screen
<ctrl-u>        " move back 1/2 a screen
<ctrl-e>        " scroll the screen up
<ctrl-y>        " scroll the screen down
<ctrl-o>        " jump to previous location in :jumps
<ctrl-i>        " jump to next location in :jumps
``              " jump to last jump location
`.              " jump to last edit location
'.              " jump to start of line of last edit
g; g,           " cycle backwards, forwards in `:changes` (edit locations)
```

## <a id="Editing">Editing</a>
### <a id="insert-exit">Insert & Exit</a>
```vim
<Esc>           " exit insert mode
<ctrl-c>        " exit insert mode
i a             " insert before, after the cursor
I A             " insert at beginning, end of line
gi gI           " insert at last insert location, first column
o O             " open a new line below, above the current line
R               " enter REPLACE mode, cursor overwrites everything
gR              " like R but without affecting layout
```

### <a id="undo-repeat">Undo & Repeat</a>
```vim
u U             " undo last command, restore last changed line
<ctrl-r>        " redo
.               " repeat last edit
{n}.            " repeat last edit {n} times
```

### <a id="basic-editing">Basic Editing</a>
```vim
x X             " delete a single character under, before cursor
{n}x {n}X       " repeat x, X {n} times
r{c}            " replace character under cursor with {c}
gr{c}           " like r, without affecting layout
~               " switch case a single character
g~{m}           " switch case of motion {m} (i.e. `g~iw~` for word)
gu{m} gU{m}     " lower, upper case of motion {m}
J gJ            " join current line with next, without space
dd D            " delete (cut) entire line, to end of line
dw              " delete (cut) to the next word
cc C            " change (replace) entire line, to end of line
cw              " change (replace) to the end of the current word
caw             " change (replace) the current word (including spaces)
ciw             " change (replace) the current word (not including spaces)
ce              " change (replace) forwards to the end of a word
cb              " change (replace) backwards to the start of a word
c0              " change (replace) to the start of the line
c$              " change (replace) to the end of the line
c/pattern       " change (replace) to first occurrence of 'pattern'
s               " delete character and substitute text (equal to `cl`)
S               " delete line and substitute text (equal to `cc`)
xp              " transpose two letters (delete and paste)
ylxp            " transpose two letters (yank, delete and paste)
vyxp            " transpose two letters (yank, delete and paste)
>>              " indent current line right
<<              " indent current line left
==              " re-indent line (using 'equalprg' if specified)
=%              " indent current block of code
gg=G            " re-indent entire buffer
<ctrl-a>        " find number in current line and increment by 1
<ctrl-x>        " find number in current line and decrement by 1
{num}<ctrl-a>   " find number in current line and increment by {num}
{count}:        " will translate {count} to :{range} in ex mode
```
**Notes:**
- All (c)hange commands end with the editor in INSERT mode

- All double-char commands (i.e. `dd`, `cc`, `yy`, etc) can be thought of as a shortcut to `0{operator}$`, the breakdown is as follows:
```vim
    0           " go to the start of the line
    {operator}  " {operator} of your choice
    $           " go the end of the line
```

- The above can also be combined with a {count} prefix, for example 2dd will delete 2 lines below. A similar result could also be achieved using the full expression of the operator and motion: `{operator}{count}{motion}`, i.e. `d1j` will delete 2 lines down (cursor line + 1 down) and `y2w` will yank 2 words forward. In similar fashion the VISUAL mode operator `v` can be used, e.g. `vi}` will visually select everything inside the curly braces.

### <a id="insert-mode">INSERT mode</a>
```vim
<ctrl-w>        " delete a word backwards
<ctrl-u>        " undo edit on current line, <BS><CR> on empty lines
<ctrl-d>        " shift left one shift width
<ctrl-y>        " put text from the above line column
<ctrl-e>        " put text from the below line column
<ctrl-r>{reg}   " put register {reg}
<ctrl-g>u       " break the (u)ndo chain
<ctrl-v>{c}     " insert char {c} literally
<ctrl-v>{n}     " insert char by decimal value
<ctrl-v>u{n}    " insert unicode char by decimal value
<ctrl-a>        " insert previously inserted text (equal to `<ctrl-r>.`)
<ctrl-@@>       " same as <ctrl-a> + exit INSERT mode
<ctrl-p>        " auto-complete after cursor
<ctrl-n>        " auto-complete before cursor
<ctrl-o>{cmd}   " execute one NORMAL mode command (see below note)
<alt-{cmd}>     " like <ctrl-o>, does not work on all keyboards
^x^e ^x^y       " scroll up, down
```
**Note:** Any NORMAL mode motion can be run from INSERT mode by using `<ctrl-o>{op}` or `<alt+{op}>` (only works on some keyboards). Alternatively, if we're in ex mode we can use `:norm {cmd}`. For example: say we want to append text at the end of the line we can simply press `<alt+A>` or `<ctrl-o>A` which will take us to the end of the line, all without leaving INSERT mode. The same can be achieved from ex mode with `<Esc>:norm A`, even though the latter isn't useful for this specific example it can be useful in many other cases where more complex commands are required (e.g. using the `global` command: `:g/regex/norm >>` will indent all lines matching the regex)

### <a id="cut-copy-and-paste">Cut, copy and paste</a>
```vim
p P             " (p)ut or (p)aste clipboard after, before cursor
]p [P           " like p, P with indent adjusted
gp gP           " like p, P leaving cursor after new text
yy              " yank (copy) a line
{n}yy           " yank (copy) {n} lines
yl              " yank a single character (l = to the right)
vy              " yank a single character (using VISUAL mode)
yw              " yank (copy) to the next word
yiw             " yank (copy) (i)nner word (entire word)
yaw             " yank (copy) (a) word (entire word, including spaces
y$              " yank (copy) to end of line
dd              " delete (cut) a line
{n}dd           " delete (cut) {n} lines
dw              " delete (cut) to the next word
D               " delete (cut) to the end of the line
d$              " delete (cut) to the end of the line
d^              " delete (cut) to the first non-blank character of the line
d0              " delete (cut) to the beginning of the line
d/pattern       " delete (cut) to first occurrence of 'pattern'
x X             " delete (cut) character under, before the cursor
"+p             " paste the `+` register (the clipboard)
"0p             " paste the yank `0` register
"{reg}p         " paste {reg} (:registers)
"{reg}yy        " yank current line into {reg} (:registers)
"_dd            " delete line into the 'blackhole' register (no clipboard)

```
**Copy paste using ex mode:**
```vim
:{range}y           " yank {range} into the default register
:{range}m{line}     " move {range} below {line}
:{range}co{line}    " copy {range} to below {line}
:{range}t{line}     " shortcut to the `:co[opy]` command above
:-2co .             " copy line 2 above cursor to line below cursor
:+2t 0              " copy line 2 below cursor to start of file
:.,$t $             " copy all lines from the cursor to end of file to end of file
:1,3co 5            " copy lines 1-3 to below line 5
```

**Notes:**
- To paste a register directly from INSERT or ex mode use `<ctrl-r>` followed by a register name.

- By default the `Y` command is mapped to `yy` (yank line) which isn't consisnt with the behaviors of `C` and `D` (from cursor to end of line), a very useful mapping for both NORMAL and VISUAL mode is:

```vim
    nnoremap Y y$
    xnoremap Y <Esc>y$gv
```

- By default all modification operators `d c x` copy the modified text to the unnamed `"` register (unless `set clipboard` was set) which can be confusing at first. For example, let's say we want to overwrite a word with yanked text, we would naturally do `ciw<Esc>p` or `ciw<ctrl-r>"` only to find out the same word would be pasted (and not our yanked text), to work around that we can tell the `c` operator to copy the text into the 'blackhole' register instead: `"_ciw`. Alternatively we can also use the yank register `0` which contains the content of the last yank operation using `"0p` or `"0P` (to paste before the cursor). A few useful mappings for my leader key (by default `\`, personally I use `\<space>`) are below, so if I want to change a word without it polluting my registers I would run `<leader>bciw`, similarly if I wish to delete a line I would run `<leader>dd`:

```vim
    nnoremap <leader>b "_
    nnoremap <leader>d "_d
    nnoremap <leader>D "_D
    nnoremap <leader>dd "_dd
    xnoremap <leader>b "_
    xnoremap <leader>D "_D
    xnoremap <leader>D "_D
    xnoremap <leader>x "_x
```

- In addition to being copied to the default register (`"` or `*`), every 'deleted' text is also copied into the 'small delete' registers {1-9}. To view the latest deletes run `:reg[isters]` or `:di[splay]`. A neat trick to cycle through the delete registers is to use `"1p` and then run `u.`, the undo+repeat advances the 'pasted register' so it will perform `"2p`, `"3p` and so forth.

## <a id="visual-mode">VISUAL mode</a>
### <a id="selections">Selections</a>
```vim
<Esc>           " exit visual mode
<ctrl-c>        " exit visual mode
v               " start VISUAL mode in 'character' mode
V               " start VISUAL mode in 'line' mode
<ctrl-v>        " start VISUAL mode in 'block' mode
o               " move to other end of marked area
O               " move to other corner of block
$               " select to end of line (include newline)
g_              " select to end of line (exclude newline)
aw              " select a word
ab              " select a block with ()
aB              " select a block with {}
ib              " select inner block with ()
iB              " select inner block with {}
gv              " NORMAL mode: reselect last visual selection
```

**Notes:**
- The above are just some of the available commands as any text object or motion can be used, e.g. `viw` will visually select current word or `vat` will select an entire tag: `<a>some text</a>`

- A neat trick you can do with VISUAL mode is using visual modes as motion operators, If you perform `d2j`, it will delete all three lines. That’s because `j` is a linewise motion. If you instead pressed `d<c-V>2j`, it would convert the motion to blockwise and delete just the column characters instead. For more information read [Hillel Wayne's blog: At least one Vim trick you might not know](https://www.hillelwayne.com/post/intermediate-vim/).

- VISUAL mode has a nice feature to expand selection based on blocks, say we wanted to select the inside of an `if` condition in C, we do so with `vi(` or `vi)`, if we wanted to expand the selection to the outer block we can just run `i{` or `i}` (without having to cancel the selection and press `v` again).

### <a id="operators">Operators</a>
```vim
~               " switch case
d               " delete
c               " change
y               " yank
>               " indent right 
<               " indent left 
!               " filter through external command 
=               " re-indent line (using 'equalprg' if specified)
gq              " format lines to 'textwidth' length (cursor moves to end)
gw              " format lines to 'textwidth' length (cursor stays in place)
gu              " make selection lower-case
gU              " make selection UPPER-case
v<ctrl-a>       " increment digit under the cursor
{num}<ctrl-a>   " increment current selection by {num} (lines separately increment)
{num}<ctrl-x>   " decrement current selection by {num} (lines separately decrement)
{num}g<ctrl-a>  " increment current selection by {num} (lines serially increment)
```

**Notes:**
- Some operators, namely the repeat `.` and paste `pP` operators have unique behaviors when used after a VISUAL 'block' mode edit, that is extremely useful when editing text as blocks. Say we wanted to append semicolon to the end of a paragraph we could simply do `<ctrl-v>}A;<Esc>` we can then repeat the operation using `.` which will replicate the change to the same block range under the cursor. Similarly we can use the paste before and after the cursor `p` and `P` to paste entire columns, the following will duplicate the first column of a paragraph: `<ctrl-v>}yp` which you can easily undo as an atomic operation with `u`.

- If you want to preform an {ex} command on visual selection press `:` (with selected visuals), vim will automatically prefix your ex command with the visual range `:'<,'>` so you can execute any command on the selected text, e.g. `:'<,'>norm @q` will execute macro `q` on all visually selected lines.

- A shortcut to the above can be done with the bang `!` operator, this will automatically put is in ex command mode with the visual selection already entered, the equivalent of `:'<,'>!` ready for entering a command, this is useful in many situations, for example to sort a visual block we can just do `viB!sort` which is the equivalent `viB:!sort`. The same can also be done from NORMAL mode using `!iBsort`.

- When indenting (or any other operation) in VISUAL mode with `<>` you will find that once indented you lose the current selection, to visually reselect the text you can use `gv`, so to indent while keeping current selection use `<gv` and `>gv` respectively. Personally I never want to lose my selection when indenting hence I use the below mappings in [my keymaps](https://github.com/ibhagwan/nvim-lua/blob/main/lua/keymaps.lua):

```vim
vmap < <gv
vmap > >gv
```
## <a id="text-objects">Text Objects</a></a>
```vim
aw              " a word (includes surrounding white space
iw              " inner word (does not include surrounding white space)
as              " a sentence
is              " inner sentence
ap              " a paragraph
ip              " inner paragraph
a"              " a double quoted string
i"              " inner double quoted string
a'              " a single quoted string
i'              " inner single quoted string
a`              " a back quoted string
i`              " inner back quoted string
a) or ab        " a parenthesized block
i) or ib        " inner parenthesized block
a]              " a bracketed block
i]              " inner bracketed block
a} or aB        " a brace block
i} or iB        " inner brace block
at              " a tag block e.g. <a></a>
it              " inner tag block e.g. <a></a>
a>              " a single tag
i>              " inner single tag
gn              " next occurrence of search pattern
```
**Notes:**
- Text objects can be used with any operator, e.g. `dit` will delete "some text" from `<a>some text</a>` and `yat` will copy the entire tag into the clipboard. For more information [Jared Caroll's: Vim Text Objects: The Definitive Guide](https://blog.carbonfive.com/vim-text-objects-the-definitive-guide/).

- `gn` is a very useful text object, few examples: `cgn` will change the next search pattern match, `vgn` will visually select all text from the cursor to the next match. For more information read [Bennet Hardwick's blog: 8 Vim tips and tricks for advanced beginners](https://bennetthardwick.com/blog/2019-01-06-beginner-advanced-vim-tips-and-tricks/).

- Any text object can also be used with a count. For example, we have the below text (cursor at ^):
```vim
    if (function(param1, param2, param3)) {
                                 ^
```
If we run `di)` we will delete all 3 parameters. However we can also run `d2i(` to delete inside the parent parenthesis, thus deleting the entire condition and resulting in the text `if () {`. I found this very useful tip (and others) in [Antoyo's blog](https://blog.antoyo.xyz/vim-tips).

## <a id="search-replace">Search & Replace</a>
```vim
# *                         " search word under cursor backward, forward
g# or g*                    " like #, * but also find partial matches
/pattern                    " search for pattern
/pattern<ctrl-g>            " go to next match without exiting search mode `/`
/pattern/{-+n}              " put cursor {-+n}th line below/above the match
/pattern/e{-+n}             " put cursor {-+n}th char before/after match (e)nd
/pattern/b{-+n}             " put cursor {-+n}th char before/after match (b)egin
?pattern                    " search backward for pattern
/<CR>                       " repeat search in same direction
?<CR>                       " repeat search in opposite direction
/<ctrl-r>/                  " put last search pattern into search mode
/<ctrl-r><ctrl-w>           " put word under cursor into search mode
/<ctrl-r><ctrl-a>           " put WORD under cursor into search mode
/\v{pattern}                " Search forwards using Vim's 'very magic' pattern
                            " special characters can be used without esc seq
n N                         " repeat search in same, opposite direction
& or :&&                    " repeat last substitute in the same line
g&                          " repeat last substitute on all lines
:noh                        " remove highlighting of search matches
:s/old/new/                 " replace first occurrence of old with new
:s/old/new/i                " replace first occurrence of old with new (case insensitive)
:s/old/new/g                " replace all old with new throughout line ('globally')
:%s/old/new                 " replace first occurrence of old with new throughout file
:%s/old/new/g               " replace all old with new throughout file ('globally')
:%s/old/new/gc              " replace all old with new throughout file with confirmations
:%s/old/'&'/g               " surround all occurrences of old with ' (i.e. 'old')
:s/\%Vold/new/              " replace all occurrences of old with new in visual selection
:{range} s/old/new/         " replace all old with in {range} (e.g. `:10,20s/...`)
:{num},$ s/old/new/         " replace all old with new from line {num} to last line
:g/regex/{ex}               " run :{ex} for every line matching regex
:g!/regex/{ex}              " run :{ex} for every line NOT matching regex
:v/regex/{ex}               " same as above, shortcut to `:g!`
:g/regex/y {reg}            " copy all matching lines to register {reg}
                            " capitalize {reg} to append to {reg}
:g/regex/m $                " move all matching lines to end of file
:g/regex/-1j                " for every matching line, go up one line and join
:%s/\s\+$//e                " remove all trailing whitespaces throughout buffer
:g/^/m0                     " reverse order of all lines in current buffer
:v/./d                      " delete all empty lines (same as `:g!/./d`)
/\%>10l\%<20l{regex}        " search for regex between lines 10-20 (excluding ln 20)
/\v%>10l%<20l{regex}        " same as above using 'very magic'
/\%u{xxxx}                  " search for unicode character `u+{xxxx}` (e.g. `u+0061` for `a`)
```

**Notes:**

- You can repeat the last substitute interactively with `g&`

- For more information on the different `.../g` modifiers read `:help s_flags`

- For more information on the "very magic" pattern read [Vim fandom: Simplifying regular expressions using magic and no-magic](https://vim.fandom.com/wiki/Simplifying_regular_expressions_using_magic_and_no-magic).

- The `g` and `v` ex commands are extremely useful, few examples: `:g/pattern/d` or `:g/pattern/norm dd` will delete all lines matching `pattern`, you can also supply a range to the command: `:g/pattern/-1d` will delete one line above all lines matching `pattern`, same can be achieved with `:g/pattern/norm 1jdd`

- `:y {reg}` copies the range to the register {reg}. If {reg} is a capital letter register, this appends to the existing register, i.e. if we do `let @a = '' | %g/regex/y A` it will copy all lines matching regex in the entire file to register a. We can then copy the register to the system clipboard with `let @+ = @a`.

- Important note regarding the use of `:s` vs `:g`: `s` is a shortcut for `substitute` and `g` is a shortcut for `global`, therefore `s` should be used when doing search and replace (substitution) and `g` should be used when you'd like to execute a command for every match (not necessarily substitution), each has it's strength and it's a matter of using the right tool for the job. Example: say we wanted to substitute `bar` with `blah` for every line containing `foo`, we could achieve the same result with both but as you can see the `g` command would be a better fit in this case:

```vim
    g/foo/s/bar/blah/g  

    :%s/foo/\=substitute(getline('.'), 'bar','blah','g')
```

### <a id="regex-examples">REGEX Examples</a>
### Search examples
```vim
/^fred.*joe.*bill           " line beginning with fred, followed by joe then bill
/^[A-J]                     " line beginning A-J
/^[A-J][a-z]\+\s            " line beginning A-J then one or more lowercase characters then space or tab
/fred\_.\{-}joe             " fred then anything then joe (over multiple lines)
/fred\_s\{-}joe             " fred then any whitespace (including newlines) then joe
/fred\|joe                  " fred OR joe
```

### Substitution examples
```vim
:%s/fred/joe/igc                " general substitute command
:%s/\r//g                       " delete DOS Carriage Returns (^M)
:'a,'bg/fred/s/dick/joe/gc      " VERY USEFUL
:s/\(.*\):\(.*\)/\2 : \1/       " reverse fields separated by :
" non-greedy matching \{-}      " `:help /\{-}`
:%s/^.\{-}pdf/new.pdf/          " to first pdf)
:s/fred/<c-r>a/g                " substitute 'fred' with contents of register 'a'
:%s/^\(.*\)\n\1/\1$/            " delete duplicate lines
" running multiple commands
:%s/suck\|buck/loopy/gc         " ORing
:s/__date__/\=strftime("%c")/   " insert datestring
:%s/\f\+\.gif\>/\r&\r/g | v/\.gif$/d | %s/gif/jpg/
```

### <a id="multiple-files">Multiple Files</a>

> To learn more I recommend watching
> [Drew Neil: Searching Multiple Files with vimgrep](http://vimcasts.org/episodes/search-multiple-files-with-vimgrep/)

```vim
:vimgrep /pattern/ {file} " search for a pattern using vimgrep
:grep /pattern/ {file}    " search for a pattern using an external program
:copen                    " open the 'quickfix` window containing all matches
:cn                       " jump to the next match
:cp                       " jump to the previous match
:cdo                      " execute a command for each quickfix entry
:cfdo                     " execute a command for each quickfix file
```

When just starting to use vim search and replace an entire project at first
seems overly complex, plugins can definitely help here (fzf comes to mind) but
once you understand how to use the quickfix list it's actually quite easy.

Vim comes builtin with the `vimgrep` utility which searches for a regex patter
and populates the quickfix list accordingly. From there, you can manipulate
matches using the `cdo|cfdo` combo.

Searching the project is as easy as:
```vim
" search the current file
:vimgrep /foo/ %
" search the entire project
:vimgrep /foo/ **/*
```

Personally I prefer to use `:grep` in conjunction with the
[`rg`](https://github.com/BurntSushi/ripgrep) utility, to do so add the below
to your `init.vim`:
```vim
set grepprg=rg\ --vimgrep\ --no-heading\ --smart-case\ --hidden
set grepformat=%f:%l:%c:%m
```

And then run the same search:
```vim
" search the current file
:grep 'foo' %
" search the entire project
:grep 'foo'
" or if you prefer to use the location-list
:lgrep 'foo'
```

From here it's very easy to search and replace all matches with
[any substitute command](#search-replace):
```vim
" if you want to run the command once for each entry use `cdo`
" otherwise the below should suffice as `%s` searches the entire file
:cfdo %s/foo/bar/g
" save all changes
:wall
```

## <a id="macros">Macros</a>
```vim
q{a-z}                  " start recording macro to register {a-z}
q{A-Z}                  " append recording macro to register {a-z}
q                       " stop macro recording
@{a-z}                  " execute macro {a-z}
{count}@{a-z}           " execute macro {a-z} on {count} lines
@@                      " repeat execution of last macro
:@{reg}                 " execute {reg} as an ex command
@:                      " repeat execution of last ex command e.g. `:s/...`
@.                      " execute last inserted text as macro
@='[cmds]'              " execute commands through the expression register
:'<,'>normal @q         " execute macro `q` on visual selection
"{a-z}p                 " paste macro {a-z} (register)
"{a-z}y$                " yank into macro {a-z} to end of line
```

**Notes:**
- Macros are essentially just a saved series of keystrokes, if you’re recording a macro and make a mistake, don’t start over, instead, undo it and keep going normally, using macro `q` as an example (recorded with `qq`), once you’re finished with the macro, press `"qp` to paste it to an empty line, remove the mistaken keystrokes and the undo and copy it back into the `q` register with `"qy$`.

- Don’t leave undos in your macro. If you undo in a macro to correct a mistake, always be sure to manually remove the mistake and the undo from the macro. In replay mode, an undo will undo the entire macro up until that point, erasing all of your hard work and bleeding the macro out into the rest of your text.

- The above was taken from [Hillel Wayne's blog: Vim Macro Tricks](https://www.hillelwayne.com/vim-macro-trickz/).

- Alternatively, you can resume recording a macro using the capitalized version of the registers, say we started recording into the `q` register with `qq` we can "pause" the macro recording with `q` and later resume recording with `qQ` (note the capitalized `Q`).

- Macros are not limited to the current buffer, if you wish to transform text between windows just add a windows switch command to your macro (e.g. `<ctrl-w>w`). This makes it very useful to mass manipulate text between different buffers.

- A neat trick to running macros quickly is to use the `.` register which is the last inputed text. Let's say we want to run a macro that does the simple task of transposing two adjacent characters `xp`, what we can do is enter INSERT mode, enter the macro keys, press `u` for undo and then execute the macro with `@.`, thus the entire sequence equals `ixp<Esc>u` and now we can run our macro with `@.`. Alternatively, if you'd like your macro to contain a `<cr>` (down one line) you can run `O~$~<Esc>dd` and then run the macro with `2@"` (since our text was 'cut' into the default register) - this will toggle capitalization of the first and last character in 2 subsequent lines.

- Building on the above, the same `xp` macro can be run using the expression register by running `@='xp'` and then repeat the macro execution with `@@`. You can run more complex commands using the full format `{count}@='[commands]'`. For example, running `3@='v2<ctrl-a>w'` on the text `1 5 8` will increment each digit by 2 resulting in `3 7 10`. This example is equivalent to `qqv2<ctrl-a>wq2@q`: which records the edit to macro `q` and then executes it two more times. (note: if you wish to input special characters (`<Esc>`, `<CR>`, etc.) in the command line, prefix the special character with `<ctrl-v>` e.g. `<ctrl-v><Esc>`.


### <a id="recursive-macros">Recursive Macros</a>

A recursive macro, as the name suggests, is a macro that calls itself, in Vim, the call is usually done at the end of the macro, so a recursive macro will usually end with a call to itself `@q` followed by `q` to stop the macro recording. A simple recursive macro that deletes all vimscript comments in an entire buffer would look like this:
```vim
    qqq             " empty the q register, VERY IMPORTANT, read below why
    qqf"D+@qq       " execute on one line and record our recursive macro
    @q              " execute until fail
```

If you wish to test your macro before executing on the entire buffer you can take advantage of appending to a register using a capitalized register letter, let's re-do our example from above while also testing our macro:
```vim
    qqq             " empty the q register, VERY IMPORTANT, read below why
    qqf"D+q         " execute on one line and record our macro
    @q              " execute one time and check the results
    qQ@qq           " execute one more time and add the recursion call
    @q              " execute until fail
```

As you can see from above, recursive macros can be very useful if you wish to run a macro on the entire buffer without having to worry about the number of times required to run the macro, it's similar to running `9999@q` (in a file with less than 9999 lines). Note that this is different than running the ex command `:%norm @q`, in the latter case the macro would run on the entire buffer regardless if there are some lines with errors (e.g. can't find the `"`), this gives us the flexibility to choose between the two approaches, if we want to run until failure we use a recursive macro, if we wish to run on the entire buffer regardless of errors we use the `:%norm @{reg}` or `:0,$norm @{reg}` variant.

**Note:** It is VERY IMPORTANT to clean the destination register (`q` in our case) by running `q{reg}q` **before we start the recording**, the reason for this is when you're recording the macro for first time you are also running it when you're adding the recursion call `@q`, if our register isn't empty the register commands will be executed and most likely mess up our edit.

## <a id="marks">Marks</a>
```vim
:marks          " list current marks
m{a-z,A-Z}      " set mark at cursor position
                " {a-z} = local file marks
                " {A=Z} = cross file marks
'{a-z,A-Z}      " goto first non-blank character in mark {a-z,A-Z}
`{a-z,A-Z}      " goto exact cursor position of mark {a-z,A-Z}
```

## <a id="files-and-windows">Files and Windows</a>
```vim
:e {file}       " edit a file in a new buffer
:find {file}    " find and open file in &path (set path?)
gf              " find and open the file under the cursor
gF              " find and open the file under the cursor with line number
:cd %:h         " change pwd to folder of current buffer (all buffers)
:lcd %:h        " change pwd to folder of current buffer (local buffer)
:ls             " list all open buffers
:bnext or :bn   " go to the next buffer
:bprev or :bp   " go to the previous buffer
:bd             " delete a buffer (close a file)
:sp {file}      " open a file in a new buffer and split window
:vsp {file}     " open a file in a new buffer and vertically split window
:windo {ex}     " run :{ex} on all windows (e.g. :windo q - quits all windows)
:bufdo {ex}     " same as above for buffers
:on             " make current window the 'only' window (close all others)
:hide or :q     " hide (close) current window
:new            " new window in a horizontal split
:vnew           " new window in a vertical split
<ctrl-w>o       " same as `:on` above
<ctrl-w>w       " jump to next window
<ctrl-w>r       " swap windows 
<ctrl-w>q       " quit a window
<ctrl-w>s       " split window horizontally
<ctrl-w>v       " split window vertically
<ctrl-w>h       " move cursor to the left window (vertical split)
<ctrl-w>l       " move cursor to the right window (vertical split)
<ctrl-w>j       " move cursor to the window below (horizontal split)
<ctrl-w>k       " move cursor to the window above (horizontal split)
<ctrl-^>        " jump to previous buffer
<ctrl-g>        " display filename of current buffer (file)
1<ctrl-g>       " display full path of current buffer (file)
2<ctrl-g>       " display buf # and full path of current buffer (file)
```

**Notes:**
- `:bufdo` and `:windo` can be used for multiple file/window operations, for example to search and replace in all open buffers you can run `:bufdo %s/old/new/g`

- `:find` and `gf` are both dependent on the `&path` variable which by default contains `.` which is the current working directory (`:pwd`), for said commands to be effective be sure to always open Vim from your project directory or use `cd` to change Vim's working directory (best done using `cd %:h`). If you would like `:find` and `gf` to find files recursively in all folders specified by `&path` be sure `path` contains `**`. Use `:set path+=**` if you wish to add `**` to the current `&path`. Personally, I define my path as: `set path=.,,,$PWD/**` (search current directory and project directory recursively).


## <a id="tabs">Tabs</a>
```vim
:tabnew or :tabnew {file}   " open a file in a new tab
<ctrl-w>T                   " move the current split window into its own tab
gt or :tabnext or :tabn     " goto to the next tab
gT or :tabprev or :tabp     " goto to the previous tab
{num}gt                     " goto to tab {num}
:tabmove {num}              " move current tab to the {num}th position (indexed from 0)
:tabclose or :tabc          " close the current tab and all its windows
:tabonly or :tabo           " close all tabs except for the current one
:tabdo {ex}                 " run :{ex} on all tabs (e.g. :tabdo q - closes all opened tabs)
```

## <a id="term">Terminal</a>
### Vim:
```vim
:term                       " open terminal window inside vim (default shell)
:term zsh                   " open terminal window inside vim (zsh)
:vert term zsh              " opens a new terminal in a horizontal split (zsh)
```
### Neovim:
```vim
:term                       " open terminal window inside vim
:split term://zsh           " opens a new terminal in a horizontal split
:vsplit term://zsh          " opens a new terminal in a vertical split
:new term://bash            " alternates for above commands
:vnew term://bash           " alternates for above commands
<ctrl-\> <ctrl-N>           " go back to NORMAL mode
```
### Both:
```vim
<ctrl-z>                    " suspend vim to background, `fg` in term to resume
:sus[pend][!] or st[op][!]  " suspend vim (equal to ctrl-z) if `!` is specified
                            " vim will write changes to all buffers before suspend
```

## <a id="spellcheck">Spellcheck</a>
```vim
:set spell                  " enable spellcheck for current buffer
:set nospell                " disable spellcheck for current buffer
:set spelllang={lang}       " set spellcheck lang, i.e. `en_us`
[s                          " find previous misspelled word
]s                          " find next misspelled word
z=                          " see spellcheck suggestions
{num}z=                     " automatically accept (change) suggestion {num}
zg                          " add current word to dictionary
zw                          " add current word to dictionary as a 'bad' word
zug                         " undo `zg`
zuw                         " undo `zw`
```

## <a id="misc-commands">Misc commands</a>
```vim
K                           " lookup keyword under cursor with `man`
ga                          " display dec/hex/oct values of character under cursor
g8                          " display hex value of utf-8 character under cursor
g?                          " rot13 a character (cycle 13 alphanumeric chars backwards)
ggg?G                       " rot13 entire buffer
g<                          " view last command (:<cmd>!) output
<ctrl-l>                    " redraw window, clear status message (error messages, search, etc)
<ctrl-g>                    " display filename and position
g<ctrl-g>                   " display cursor, line and column position
```
```vim
:help {keyword}             " view help for {keyword}
:help {keyword}<ctrl-d>     " list help of matching {keyword}
:helpgrep {keyword}         " help search for {keyword}, open results with :cwindow
:read {file}                " read {file} below the cursor
:!{cmd}                     " execute shell command {cmd}
:!%                         " execute current file
:!%:p                       " execute current file (use full path)
:.!{cmd or !!{cmd}          " filter current line through {cmd}
!!date                      " replace current line with output of `date`
:read !{cmd}                " read output of {cmd} below cursor
:<ctrl-f> or q:             " enter command line window, <ctrl-c> or :q to exit
q/ or q?                    " same as above but for searches
:echo &{opt}                " echo Vim option to command line (expand values)
:set {opt}?                 " echo Vim option to commend line (no expand)
:source                     " 'source': execute a file as a series of commands
:source $MYVIMRC            " source your $MYVIMRC
:retab                      " retab current buffer according to `expandtab` and `shiftwidth`
:earlier {time}             " revert a file to earlier time, e.g. `earlier 1m`
:later {time}               " revert a file to later time
:visual :edit               " leave :Ex mode
:<ctrl-b>                   " go to start of line in command line mode
:<ctrl-e>                   " go to end of line in command line mode
```

## <a id="ctrl-r-and-the-expression-register">Ctrl-R and the Expression Register</a>

The expression register (`=`) is used to evaluate expressions and can be accessed using `"=` from NORMAL and VISUAL modes and  `<ctrl-r>` from INSERT and ex command modes, it is very useful to insert registers and other input into the command line or directly to the document when you don't want to leave INSERT mode (useful so you can repeat the entire edit with `.`):

### INSERT and ex modes:
```vim
:registers {reg}            " display registers {reg} and their values
:display "0                 " display value of registers `"` and `0'
:{line}put                  " put clipboard register under {line}
:put{reg}                   " put value of {reg} in the line below
:put={expr}                 " put value of {expr} in the line below
:let @{reg}={expr}          " manually assign value to {reg}
:let @*=expand('%')         " expand current file path into `*` (clipboard)
<ctrl-r>{reg}               " put register {reg}
<ctrl-r>={expr}             " put the value of {expr}
<ctrl-r>=[1,2,3]            " put 1<cr>2<cr>3<cr>
<ctrl-r>%                   " put file path of current buffer
<ctrl-r>-                   " put last deleted text
:<ctrl-r><ctrl-w>           " put word under cursor
:<ctrl-r><ctrl-a>           " put WORD under cursor (includes punctuation)
:<ctrl-r><ctrl-l>           " put current cursor line
:<ctrl-r><ctrl-f>           " put file path under cursor (does not expand ~)
:<ctrl-r><ctrl-p>           " put file path under cursor (expands ~)
```

### NORMAL and VISUAL modes:
```vim
"={expr}                    " evaluate {expr} into the expression register
"={expr}p                   " evaluate {expr} and paste (also saved in the register)
"={expr}P                   " save as above, paste before the cursor
```

**Notes:**
- For more information on evaluating expressions `:help expression`. For even more information read [Aaron Bieber's: Master Vim Registers With Ctrl R](https://blog.aaronbieber.com/2013/12/03/master-vim-registers-with-ctrl-r.html) and watch [Vimcasts: Simple calculations with Vim's expression register calculations with Vim's expression register](http://vimcasts.org/episodes/simple-calculations-with-vims-expression-register/)

- When inserting a register with `<ctrl-r>{reg}` and then repeating the edit with `.` you will find out that same text will be entered even if the register contents has changed, to have the actual command enter the `.` register we need to use `<ctrl-r><ctrl-o>{reg}` instead. Best shown with an example, say we have the below text and we wish to add parens around the text

```vim
    one           " we want to change this to => (one)
    two           " we want to change this to => (two)
```

  We can modify the first line using `ciw(<ctrl-r>+)` which will result in "(one)" but when we repeat the action for "two" the result would still be "(one)" as the actual text is saved in the register. To circumvent this we can use `ciw(<ctrl-r><ctrl-o>+)` instead which inserts the actual command `^R^O+` into the register, that way when we repeat the action with `.` the result would be as expected "(two)".

- For more information regarding `<ctrl-r><ctrl-o>` read `:help i_ctrl-r_ctrl-o` and watch [Drew Neil's vimcast: Pasting from INSERT mode](http://vimcasts.org/episodes/pasting-from-insert-mode/)

## <a id="comparing-buffers-with-vimdiff">Comparing buffers with vimdiff</a>
```vim
:windo diffthis         " Edit current windows in diff mode
:windo diffoff          " Exit diff mode
:diffput {bufspec}      " Put diff into {bufspec} from current buffer
:diffget {bufspec}      " Pull diff from {bufspec} into current buffer
do                      " diff (o)btain, NORMAL mode for `:diffget`
dp                      " diff (p)ut, NORMAL mode for `:diffput`
[c                      " jump to previous (c)hange
]c                      " jump to next (c)hange
:windo set scrollbind   " bind all current windows scrolling
:windo set noscrollbind " unbind all current windows scrolling
```

**NOTE:** 

- The shortcuts `do` and `dp` also run `:diffupdate`, so the equivalent of `dp` can be thought of as `:diffput {target buffer} | diffupdate`

- `{targetbuffer}` above is automatically resolved as the 'other buffer' on a 2-way split, that works for both `do` and `dp`. However, `do` cannot sensibly decide which buffer to use in a 3-way split (git merge conflict resolution) and therefore cannot be used in this case. `dp` assumes we always want to 'push' changes to the working copy which is always the middle buffer and therefore can be used in a 3-way split from either the left (target) or right (merge) buffers.

## <a id="folding">Folding</a>
Fold methods (set with `set foldmethod=<method>`:

```vim
manual                  " folds must be defined by commands (i.e. `zf`)
indent                  " folds are defined by lines of equal indent
expr                    " folds are defined by `foldexpr`
marker                  " folds are defined by `{{{` and `}}}`
sytax                   " folds are defined by syntax highlighting
diff                    " folds are defined by non-modified text
```

```vim
zo                      " open fold under cursor
zO                      " open folds under cursor recursively
zc                      " close fold under cursor
zC                      " close folds under cursor recursively
za                      " toggle fold under cursor
zA                      " toggle folds under cursor recursively
zM                      " close all folds, set `foldlevel` to 0
zR                      " open all folds, set `foldlevel` to highest level
zn                      " sets `nofoldenable`, disable folding
zN                      " set `foldenable`, restore all previous folds
zi                      " toggle between `foldenable` and `nofoldenable`
```

Fold management when `foldmethod` is set to `manual` or `marker`:

```vim
zf{motion}              " create a fold from current line to motion
{visual}zf              " create a fold around visual selection
{count}zF               " create a fold for {count} lines
:{range}fo[ld]          " create a fold for {range} (e.g. `:.,+3fo`)
zd                      " delete surrounding fold at the cursor
zD                      " recursively delete surrounding fold
zE                      " delete all folds in current window
```
