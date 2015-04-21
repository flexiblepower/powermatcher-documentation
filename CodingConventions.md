# Coding Conventions

The coding conventions for the Powermatcher project are defined in an xml format file. More on importing this xml file in the [Setting Up The IDE](../SettingUpTheIDE#importing-the-formatting-xml-file) section.

### Conventions

* Use 4 spaces for indentation.
* Use indentation for each new block of code (e.g. class, method, if statement) except in the switch body (the case statement is on the same level, after the case statement do indent).
* All opening braces are on the same line as the statement starting the block.
* Use default white spaces. This means 1 space after a comma or semicolon, and 1 space before and after each special operator (e.g. + - / * etc.) with the exception of the dot (function call operator).
* There should never be white space at the end of a line.
* Use at least 1 empty line between methods. We may keep several private variable definitions together to indicate that they are related.
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

### Example code

Below is an piece of dummy code to show what the effect of these rules are:

```java
/**
 * Javadoc comments with some <code>HTML</code> tags.
 * <ul>
 * <li>And</li>
 * <li>a</li>
 * <li>list</li>
 * </ul>
 */
class Example {
    int[] myArray = { 1, 2, 3, 4, 5, 6 };
    int theInt = 1;

    String someString = "Hello";
    double aDouble = 3.0;

    /*
     * Some non-javadoc comment
     */
    void foo(int parameter1,
             int parameter2,
             int parameter3,
             int parameter4,
             int parameter5,
             int parameter6) {
        switch (parameter1) {
        case 0:
            Other.doFoo();
            break;
        default:
            // Single line comment
            Other.doBaz();
        }
    }

    void bar(List<Integer> v, int max) {
        for (int i = 0; i < max; i++) {
            if (i % 2 == 0) {
                v.add(i);
            } else {
                v.add(-i);
            }
        }
    }
}

enum MyEnum {
    UNDEFINED(0) {
        void foo() {
        }
    }
}

@interface MyAnnotation {
    int count() default 1;
}
```