# Coding Conventions

The coding conventions for the FPAI projects are captured in the project settings, which provide a Eclipse formatter (e.g. see [fpai.api formatter settings](https://github.com/flexiblepower/fpai-core/blob/development/flexiblepower.api/.settings/org.eclipse.jdt.core.prefs)) and some save actions (e.g. see [fpai.api save actions settings](https://github.com/flexiblepower/fpai-core/blob/development/flexiblepower.api/.settings/org.eclipse.jdt.ui.prefs)). A summary of the conventions:

* Use 4 spaces for indentation.
* Use indentation for each new block of code (e.g. class, method, if statement) except in the switch body (the case statement is on the same level, after the case statement do indent).
* All opening braces are on the same line as the statement starting the block.
* Use default white spaces. This means 1 space after a comma or semicolon, and 1 space before and after each special operator (e.g. + - / * etc.) with the exception of the dot (function call operator).
* There should never be white space at the end of a line.
* Use at 1 empty line between methods. We may keep several private variable definitions together to indicate that they are related.
* There must be 1 empty line at the end of the file.
* All empty block statements must contains a newline.
* Keep `} else {`, `} else if {`, `} while()`, `} catch() {` and `} finally {` on the same line.
* Maximum line width is 120 characters for both code and comments.
* Wrapping only occurs when it can't fit on a single line. If wrapping is needed, the all elements are placed on a new line.
* Comments are indented the same as the code where they belong to.
* Multi-lined comments or javadoc use a `*` for every new line.
* The import statements should be organized (e.g. no unused imports and no import all from a package).
* Block statements should always use brackets, even when it is a single statement.
* Make private variable in classes final as much as possible. For data object, prefer to make them [immutable](http://en.wikipedia.org/wiki/Immutable_object#Java).
* For member access, the `this.` qualifier should only be used when necessary.