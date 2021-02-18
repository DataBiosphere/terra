# IntelliJ for Power Users

## Editing

## Refactoring
### Move
### Rename

### Introduce
The family of Introduce refactorings 
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
## Searching

## Debugging

## Analysis
### Diagrams
### Inspections

## Plugins
### Gradle

## Database
