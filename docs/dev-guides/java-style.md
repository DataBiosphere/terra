# Java Style Guide

## tl;dr

*   All Terra Java code should adhere to the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) where possible.
*   All repositories should be set up with auto-formatting and static analysis.
*   For stylistic concerns that cannot be automatically enforced, see the guidelines below.

## Java style addendums

None yet.

## Static analysis and code formatting

We believe developers should strive to spend as little time as possible worrying about and discussing code style. Code formatting and static analysis programs can be helpful tools in pursuit of that goal, since automatic enforcement at compile-time relieves the team from discussing such issues in PR reviews.

All Terra repos should use the following tools:

*   [Spotless Gradle plugin](https://plugins.gradle.org/plugin/com.diffplug.gradle.spotless) for auto-formatting. (See [example](https://github.com/DataBiosphere/terra-resource-buffer/blob/678a55c7a07b076dec318f4850a47a17ca3f56d9/build.gradle#L181). Devs may also find it useful to set up [google-java-format for IntelliJ](https://plugins.jetbrains.com/plugin/8527-google-java-format).)
*   [Spotbugs Gradle plugin](https://plugins.gradle.org/plugin/com.github.spotbugs) for static analysis. (See [example](https://github.com/DataBiosphere/terra-resource-buffer/blob/678a55c7a07b076dec318f4850a47a17ca3f56d9/build.gradle#L241).)

We will continually evaluate additional static analysis tools (such as [PMD](https://docs.gradle.org/current/userguide/pmd_plugin.html), [Error Prone](https://github.com/tbroyer/gradle-errorprone-plugin), [Codacy](https://app.codacy.com/app), [Checker framework](https://checkerframework.org/)) for inclusion in the above list. We should error towards adopting any tool that can provide low-friction enforcement of some of the key conventions or best practices below.

## Guidelines & best practices

The sections below summarize our teams’ collective wisdom regarding various Java coding topics. Some advice is meant to be stronger than others — look out for the following keywords:

*   **always** – always follow this practice.
*   **should** – strong guidance; should be followed with few exceptions.
*   **prefer** – softer guidance, with numerous exceptions.
*   **may / optional / at your discretion** – indicates a matter of taste or personal style.

To learn more about Java best practices, a widely-recommended book is Effective Java, 3rd edition. Many of the guidelines below are paraphrasing Effective Java chapters; look for explicit references to chapters from the book, with links to the [https://github.com/david-sauvage/effective-java-summary](https://github.com/david-sauvage/effective-java-summary) notes repo.

### Alphabetization

Unless there is a meaningful non-alphabetical order, _prefer_ alphabetized variable lists, enum entries, etc. in order to make lists easier to visually scan. (And in cases where there is some meaningful alternate ordering, add an explicit comment so future editors can maintain that order.)

For example, these subheadings are alphabetized for lack of a meaningful non-alphabetic order.

### Classes

We adopt a few general guidelines:

1. Minimize accessibility of classes and members.
2. Favor composition over inheritance.
3. Design and document for inheritance or else prohibit it.
4. Prefer interfaces over abstract classes.

See [EJ3 #15-25](https://github.com/david-sauvage/effective-java-summary#classes-and-interfaces) for more tips.

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

Note that some Streams, such as those returned from the `java.nio.file.Files` API, may be bound to system resources and must be explicitly closed ([docs](https://github.com/JetBrains/jdk8u_jdk/blob/master/src/share/classes/java/nio/file/Files.java#L3717)). Take care to use the try-with-resources pattern in those cases.

### Comments

Comments, as distinguished from Javadoc (see [Javadoc](#Javadoc) section below), _should_ be used to inform the reader when code cannot be self-explanatory.

A few scenarios deserving of a code comment:
* When it's not obvious *why* a given line of code exists. 
* If a future maintainer might reasonably undo this change or inadvertently break something non-obvious.
* Where the structure or reasoning behind the code is well-explained by an external resource (e.g. StackOverflow link, Jira ticket, or design doc).

### Dependency injection

For Spring-aware code (e.g. most service and terra-common-lib code), dependencies _should_ be @Autowired via the class constructor. Manual instantiation of @Component classes _should_ be rare.

_Exception_: Stairway is not Spring-aware, so it is a common pattern to create a dependency-injected class that wraps Stairway functionality.

### Exceptions

**Checked exceptions** place a burden on the users of an API. They are a tool best used when _both_ of the following are true: (1) the exception cannot be prevented by proper use of the API, and (2) the user can take some meaningful action when encountering the exception. Unless both conditions are met, checked exceptions _should_ be avoided. (See [EJ3 #71](https://github.com/david-sauvage/effective-java-summary#exceptions) for more discussion.)

Guidance for **unchecked exceptions**:

*   More specific exceptions _should_ be thrown instead of generic RuntimeExceptions.
    *   IllegalArgumentException, ConcurrentModificationException, UnsupportedOperationException are commonly reused exceptions from the Java library.
    *   Other Terra-specific runtime exceptions _should_ be centralized in [terra-common-lib](https://github.com/DataBiosphere/terra-common-lib/tree/develop/src/main/java/bio/terra/common/exception).
*   Service repos may also house application-specific exceptions ([WSM example](https://github.com/DataBiosphere/terra-workspace-manager/tree/dev/src/main/java/bio/terra/workspace/common/exception)).
*   In many cases, exceptions are explicitly associated with an HTTP status code ([example](https://github.com/DataBiosphere/terra-workspace-manager/blob/4a33150f2143d163e89bfdda839e4ebf04f09b03/src/main/java/bio/terra/workspace/common/exception/ErrorReportException.java#L11)) and used to auto-generate an appropriate response ([example](https://github.com/DataBiosphere/terra-workspace-manager/blob/4a33150f2143d163e89bfdda839e4ebf04f09b03/src/main/java/bio/terra/workspace/app/controller/GlobalExceptionHandler.java#L33)).

### Final

*   **Classes** – unless a class has been designed and documented for inheritance, it _should_ be marked as final.
*   **Function parameters **– _prefer_ not marking as final, due to the noisiness of the “final” keyword. Use where necessary for occasional semantic emphasis.
*   **Instance variables** – _always_ mark as final if it is never reassigned.
*   **Local variables** – use of final is _optional_. Short and simple methods can help avoid inappropriate variable reassignment.
*   **Static variables** – almost _always_ mark as final.

### Immutability

Quoting from Effective Java: "immutable classes are easier to design, implement, and use than mutable classes. They are less prone to error and are more secure." See [recommendations from Effective Java](https://github.com/HugoMatilla/Effective-JAVA-Summary#15-minimize-mutability) on how to minimize mutability. [Immutables](https://immutables.github.io/) is a nice library for immutable data transfer objects, as is [Guava](https://github.com/google/guava/wiki/ImmutableCollectionsExplained) for immutable collections.

### Java 8 features

Use Java 8 features where helpful. See [https://leanpub.com/whatsnewinjava8/read](https://leanpub.com/whatsnewinjava8/read) and [https://www.oracle.com/java/technologies/javase/8-whats-new.html](https://www.oracle.com/java/technologies/javase/8-whats-new.html) for more details.

Here is a quick run-down of Java 8 topics:

*   **java.time.* – **the JavaTime library _should_ be used for date-time manipulation. Duration is especially common. For non-core extensions, consider the [https://www.threeten.org/threeten-extra/](https://www.threeten.org/threeten-extra/) library.
*   **Lambda expressions** – _prefer_ lambda expressions for short and simple bits of code. If a lambda gets too complex, extract it into a named method.
*   **Method references** – _prefer_ method references where possible (e.g. `someStream().map(MyClass::getBar)`).
*   **Streams** – _prefer_ using Streams for operations that can be framed as a simple, compact stream expression. On the other hand: streams can interfere with exception handling, and complex streams may be more difficult to reason about than equivalent non-stream code, so judgement is required.

### Javadoc

Javadoc comments (`/** … */`) are meant to explain the intended purpose of the element they are attached to. See the [style guide](https://google.github.io/styleguide/javaguide.html) for a canonical list of when Javadoc is required.

Some guidelines for writing good Javadoc:

*   Spend more time documenting classes that will be frequently used or updated. Documentation mitigates against the risk of future misinterpretation.
*   Block tags (`@param` and `@return`) are generally considered _optional_:
    *   Use `@param` block tags when detail is required to clearly convey the meaning of each function parameter.
    *   Use a `@return` block tag if the main text cannot clearly express the meaning of the returned value in a natural way.

### Methods

Some guidelines around Java methods:

*   Keep methods short and simple.
*   Avoid unwieldy parameter lists.
*   Beware of accepting or returning null.
*   _Prefer_ general types for parameters and specific types for return values.
*   Avoid output parameters.

See [EJ3 #49-56](https://github.com/david-sauvage/effective-java-summary#methods) for more tips.

### Null

Avoid nullable references where possible! Of course, this isn’t always possible, but null checks and `@Nullable` annotations should always be viewed with healthy skepticism.

#### Nullable annotations

@Nullable _may_ be used to indicate parameters or return values where null values are expected. When used, this annotation _should_ be in addition to (but not instead of!) Javadoc comments indicating where and when null values are expected.

@NotNull and @Nonnull can be noisy; they _should not_ be used, unless there is a particular reason that readers might otherwise expect an argument to be nullable.

#### Optional

Optional is... optional :)

`java.util.Optional` is often a good alternative to a null reference, especially as a method return type ([ref](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555)). It is not a cure-all, however, and its use is neither required nor disallowed in Terra.

Guidelines:

*   Optional _should not_ be used as a function parameter (see [this post](https://www.google.com/url?q=http://dolszewski.com/java/java-8-optional-use-cases/&sa=D&source=editors&ust=1614364854203000&usg=AOvVaw1U3iFSNy-mdUTCPKs7XmnW) for some discussion on the topic[ref](https://stackoverflow.com/questions/31922866/why-should-java-8s-optional-not-be-used-in-arguments)).
*   Usage within function bodies and as a return value is a _matter of preference_.
*   Optional usage patterns _should_ be consistent within a class (and ideally throughout a service repo).

### Visibility

A class _should_ expose only the minimum-required set of methods and classes for use by clients. This is especially critical for common library developers, but service-level code also benefits from package-level visibility control.

_Always_ use the @VisibleForTesting annotation on methods which are made public or package private just for testing purposes.
