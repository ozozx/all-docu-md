# Lesson 1 SUMMARY

 1. The cursor is moved using either the arrow keys or the hjkl keys.
     h (left)   j (down)       k (up)       l (right)

 2. To start Neovim from the shell prompt type:

    $ nvim FILENAME

 3. To exit Neovim type: <Esc> :q! <Enter> to trash all changes.
                OR type: <Esc> :wq <Enter> to save the changes.

 4. To delete the character at the cursor type: x

 5. To insert or append text type:
    i insert text <Esc>     insert before the cursor.
    A append text <Esc>     append after the line.

NOTE: Pressing <Esc> will place you in Normal mode or will cancel
      an unwanted and partially completed command.


# Lesson 2 SUMMARY

 1. To delete from the cursor up to the next word type:    dw

 2. To delete from the cursor to the end of a line type:   d$

 3. To delete a whole line type:                           dd

 4. To repeat a motion prepend it with a number:           2w

 5. The format for a change command is:

        operator   [number]   motion

    where:

        operator -   is what to do, such as [d](d) for delete
        [number] -   is an optional count to repeat the motion
        motion   -   moves over the text to operate on, such as:
                        [w](w) (word),
                        [$]($) (to the end of line), etc.

 6. To move to the start of the line use a zero: [0](0)

 7. To undo previous actions, type:            u  (lowercase u)
    To undo all the changes on a line, type:   U  (capital U)
    To undo the undos, type:                   <C-r>


# Lesson 3 SUMMARY

 1. To put back text that has just been deleted, type [p](p). This puts the
    deleted text AFTER the cursor (if a line was deleted it will go on the
    line below the cursor).

 2. To replace the character under the cursor, type [r](r) and then the
    character you want to have there.

 3. The [change operator](c) allows you to change from the cursor to where
    the motion takes you. Type ce to change from the cursor to the
    end of the word, c$ to change to the end of a line, etc.

 4. The format for change is:

        c   [number]   motion


# Lesson 4 SUMMARY

 1. <C-g>     displays your location and the file status.
    G         moves to the end of the file.
    number G  moves to that line number.
    gg        moves to the first line.

 2. Typing / followed by a phrase searches FORWARD for the phrase.
    Typing ? followed by a phrase searches BACKWARD for the phrase.
    After a search type n to find the next occurrence in the same
    direction or N to search in the opposite direction.
    <C-o> takes you back to older positions, <C-i> to
    newer positions.

 3. Typing % while the cursor is on a (, ), [, ], {, or } goes to its
    match.

 4. To substitute new for the first old in a line type

        :s/old/new

    To substitute new for all olds on a line type

        :s/old/new/g

    To substitute phrases between two line #'s type

        :#,#s/old/new/g

    To substitute all occurrences in the file type

        :%s/old/new/g

    To ask for confirmation each time add 'c'

        :%s/old/new/gc


# Lesson 5 SUMMARY

 1. [:!command](:!cmd) executes an external command.

     Some useful examples are:
     :!ls                    - shows a directory listing
     :!rm  FILENAME          - removes file FILENAME

 2. [:w](:w) FILENAME              writes the current Neovim file to disk with
                             name FILENAME.

 3. [v](v)  motion  :w FILENAME   saves the Visually selected lines in file
                             FILENAME.

 4. [:r](:r) FILENAME              retrieves disk file FILENAME and puts it
                             below the cursor position.

 5. [:r !ls](:r!)                   reads the output of the ls command and
                             puts it below the cursor position.


# Lesson 6 SUMMARY

 1. Type o to open a line BELOW the cursor and start Insert mode.
    Type O to open a line ABOVE the cursor.

 2. Type a to insert text AFTER the cursor.
    Type A to insert text after the end of the line.

 3. The e command moves to the end of a word.

 4. The y operator copies text, p pastes it.

 5. Typing a capital R enters Replace mode until <Esc> is
    pressed.

 6. Typing "[:set](:set) xxx" sets the option "xxx". Some options are:

        'ic' 'ignorecase'   ignore upper/lower case when searching
        'is' 'incsearch'    show partial matches for a search phrase
        'hls' 'hlsearch'    highlight all matching phrases

    You can either use the long or the short option name.

 7. Prepend "no" to switch an option off:

        :set noic

 8. Prepend "inv" to invert an option:

        :set invic


# Lesson 7 SUMMARY

 1. Type :help
    or press <F1> or <Help> to open a help window.

 2. Type :help TOPIC to find help on TOPIC.

 3. Type <C-w><C-w> to jump to another window

 4. Type :q to close the help window

 5. While in command mode, press <C-d> to see possible completions.
    Press <Tab> to use the completion menu and select a match.

 6. Create your configuration file to save your preferred settings. You can
    revisit it with :e $MYVIMRC.


# Lesson 8 SUMMARY

  1. To store (yank, delete) text into, and retrieve (paste) from, a total of
     26 registers (a-z) 
  2. Yank a whole word from anywhere within a word: yiw
  3. Change a whole word from anywhere within a word: ciw
  4. Insert text directly from registers in insert mode: <CTRL-r>a

  5. Insert the results of simple arithmetic operations:
     <CTRL-r>=60\*60 <ENTER> in insert mode
  6. Insert the results of system calls:
     <CTRL-r>=system('ls -1') in insert mode

  7. Inspect registers with :reg
  8. Learn the final destination of whole line deletions: dd in
     the numbered registers, i.e. descending from register 1 - 9.  Appreciate
     that whole line deletions are preserved in the numbered registers longer
     than any other operation
  9. Learn the final destination of all yanks in the numbered registers and
     how ephemeral they are

 10. Place marks from command mode m[a-zA-Z0-9]
 11. Move line-wise to a mark with '

