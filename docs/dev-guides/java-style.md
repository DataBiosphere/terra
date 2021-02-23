# Java Style Guide
[broad.io/terra-java-style](http://broad.io/terra-java-style)

## tl;dr

*   All Terra Java code should adhere to the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) where possible.
*   All repositories should be set up with auto-formatting and static analysis.
*   For stylistic concerns that cannot be automatically enforced, see the guidelines below.

## Static analysis and code formatting

Developers should strive to spend as little time as possible worrying about and discussing code style. Code formatting and static analysis programs can be helpful tools in pursuit of that goal, since automatic enforcement at compile-time relieves the team from discussing such issues in PR reviews.

All Terra repos should use the following tools:

*   [Spotless Gradle plugin](https://plugins.gradle.org/plugin/com.diffplug.gradle.spotless) for auto-formatting. (Devs may also find it useful to set up [google-java-format for IntelliJ](https://plugins.jetbrains.com/plugin/8527-google-java-format).)
*   [Spotbugs Gradle plugin](https://plugins.gradle.org/plugin/com.github.spotbugs) for static analysis.

We will continually evaluate additional static analysis tools (such as [PMD](https://docs.gradle.org/current/userguide/pmd_plugin.html) or [Error Prone](https://github.com/tbroyer/gradle-errorprone-plugin)) for inclusion in the above list. We should error towards adopting any tool that can provide low-friction enforcement of some of the key conventions or best practices below.

## Guidelines & best practices

The sections below summarize our teams’ collective wisdom regarding various Java coding topics. Some advice is meant to be stronger than others — look out for the following keywords:

*   **always** – always follow this practice.
*   **should** – strong guidance; should be followed with few exceptions.
*   **prefer** – softer guidance, with numerous exceptions.
*   **may / optional / at your discretion** – indicates a matter of taste or personal style.

To learn more about Java best practices, a widely-recommended book is Effective Java, 3rd edition (see [summary notes](https://github.com/david-sauvage/effective-java-summary)).

### Alphabetization

Unless there is a meaningful non-alphabetical order, _prefer_ alphabetized variable lists, enum entries, etc. in order to make lists easier to visually scan.

For example, these subheadings are alphabetized for lack of a meaningful non-alphabetic order.


### Closing resources

Code that uses `Closeable` resources (such as input streams or sockets) _should_ use the try-with-resources pattern:

```
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br =
                   new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

### Dependency injection

For Spring-aware code (e.g. most service and terra-common-lib code), dependencies _should_ be @Autowired via the class constructor. Manual instantiation of @Component classes _should_ be rare.

_Exception_: Stairway is not Spring-aware, so it is a common pattern to create a dependency-injected class that wraps Stairway functionality.

#### Exceptions

Checked exceptions place a burden on the users of an API; they _should_ be used only if the exception cannot be prevented by proper use of the API, and the user can take some meaningful action when encountering the exception.

Guidance for unchecked exceptions:

*   More specific exceptions _should_ be thrown instead of generic RuntimeExceptions.
    *   IllegalArgumentException, ConcurrentModificationException, UnsupportedOperationException are commonly reused exceptions from the Java library.
    *   Other Terra-specific runtime exceptions _should_ be centralized in [terra-common-lib](https://github.com/DataBiosphere/terra-common-lib/tree/develop/src/main/java/bio/terra/common/exception).
*   Service repos may also house application-specific exceptions ([WSM example](https://github.com/DataBiosphere/terra-workspace-manager/tree/dev/src/main/java/bio/terra/workspace/common/exception)).
*   Each exception _should _indicate which HTTP response code it corresponds to.

#### Final

*   **Classes** – unless a class has been designed and documented for inheritance, it _should_ be marked as final.
*   **Static variables** – almost _always_ mark as final.
*   **Instance variables** – _always_ mark as final if it is never reassigned.
*   **Parameters and local variables** – _should not_ generally be marked as final, due to the noisiness of the “final” keyword. It may be used for occasional semantic emphasis, but widespread use is discouraged.


#### Java 8 features

Use Java 8 features where helpful. See [https://leanpub.com/whatsnewinjava8/read](https://leanpub.com/whatsnewinjava8/read) and [https://www.oracle.com/java/technologies/javase/8-whats-new.html](https://www.oracle.com/java/technologies/javase/8-whats-new.html) for more details.

Here is a quick run-down of Java 8 topics:

*   **java.time.* – **the JavaTime library _should_ be used for date-time manipulation. Duration is especially common. For non-core extensions, consider the [https://www.threeten.org/threeten-extra/](https://www.threeten.org/threeten-extra/) library.
*   **Lambda expressions** – _prefer_ lambda expressions for short and simple bits of code. If a lambda gets too complex, extract it into a named method.
*   **Streams** – _prefer_ using Streams for operations that can be framed as a simple, compact stream expression. On the other hand: streams can interfere with exception handling, and complex streams may be more difficult to reason about than equivalent non-stream code, so judgement is required.

#### Nullable annotations

@Nullable _may_ be used to indicate parameters or return values where null values are expected. When used, this annotation _should_ be in addition to (but not instead of!) Javadoc comments indicating where and when null values are expected.

@NotNull can be noisy; it _should not_ be used, unless there is a particular reason that readers might otherwise expect an argument to be nullable.

#### Optional

Optional is... optional :)

Guidelines:

*   Optional _should not_ be used as a function parameter ([ref](https://stackoverflow.com/questions/31922866/why-should-java-8s-optional-not-be-used-in-arguments)).
*   Usage within function bodies and as a return value is a _matter of preference_.
*   _Prefer_ consistency within a class or service repo.


#### Visibility

A class _should_ expose only the minimum-required set of methods and classes for use by clients. This is especially critical for common library developers, but service-level code also benefits from package-level visibility control.

_Always_ use the @VisibleForTesting annotation on methods which are made public just for testing purposes.
