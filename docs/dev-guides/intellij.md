# IntelliJ for Power Users
IntelliJ has many secrets, and the easiest way to discover them is on accident. However, it's worth
compiling some of the most useful ones in one place, so that one can benefit more quickly from them.
All keyboard shortcuts are for the MacOS scheme.

## Editing
### Autocomplete
### Generate
### Comment
Cmd+/ will toggle a line comment on the current line or selected section of code.
### Javadoc
Ctrl+Option+Q renders any javadoc around the cursor in place. 
## Refactoring
### Move
### Rename

### Introduce
The family of Introduce refactorings is worth the price of admission to IntelliJ. It can rapidly turn
OK code into great code.
#### Constant
Cmd+Opt+C will create a class constant from a selected expression. It will suggest an UPPER_SNAKE_CASE
name which can be changed.
![img](./images/intellij/introduce_constant.png)
#### Variable
Similarly, Cmd+Opt+V creates a variable in the local scope from a selected expression.
#### Method
To introduce a method, select an expression or group of expressions with a well-defined set of inputs
and zero or one output. Not all selections are valid methods, naturally, and occasionally it's necessarily
to refactor slightly so that the tool gets the right result. Frequently, it considers values to be constant
that are desired as parameters. Those can be fixed with the Change Signature refactoring.

#### Change Signature
The Change Signature refactoring, cmd+F6, allows renaming and rearranging parameters to a method or
constructor signature. It fixes up subclass overrides and allows previewing the result.
![img](./images/intellij/change_signature.png)

## Navigation
### Back and Forward
Cmd+Opt+left/right navigate back and forth through history. This isuseful especially for jumping around
withing one or two files.
### Recent Files
Cmd+E opens the Recent Files dialog. This list can be refined by using initial letter searches of filenames.
It's especially useful when flipping to the most recently used file, since that's the file that's selected
initially. On the left side is a list of recently active tool panes for quick navigation.
![img](./images/intellij/recent_files_list.png)
![img](./images/intellij/recent_files_selection.png)
### Goto Line
Cmd+G brings up an input box taking a line number to visit. A column can also be provided after a colon.
### Autowired Spring Dependencies
(IntelliJ Ultimate Only) Green spring symbols next to autowired constructor parameters or other injection
points allow navigation to the source of the bean in either a configuration or component or service. 
## Searching
There are a number of powerful, fast options for searching. Taking advantage of the different modes
is very helpful for navigating a large project.
### Text
#### Find
Cmd+F finds text in the current file. Options are available for Regex search, exact case match, and
whole words only.
#### Replace
Cmd+R does find & replace in the current file. It has the same options as Find, and can replace all
occurrences at once or visit and replace or exclude individual ones.
#### Find in Path
Ctrl+Shift+F searches textually (without regard to scope or syntax). A file type filter allows narrowing
targets by extension. 
### Targeted Search
#### Classes
Cmd+N searches all class names. It matches on capital letters in the class name, so the class
`WsmControlledResource` would be matched by "WCR".
#### Files
Similarly, Cmd+Shift+N searches file names in the project.
#### Symbols
Cmd+Shift+Opt+N searches all symbols in the project.
#### All
Pressing shift twice in succession launches find in All categories.
### Implementations

## Debugging

## Analysis
### Diagrams
### Inspections

## Plugins
### Gradle

## Database

## Equipment
### Keyboard
It's very helpful to have a full-size keyboard with all of Ctrl, Cmd, and Option on both sides. Having
a missing control key on the right side is like trying to play a piano where all the C sharp keys are missing
on the right-hand side. It's just painful.

Additionally, it's very helpful to have a mechanical keyboard with a crisp stroke. This gives confidence
in keystrokes, especially with the modifier keys.
