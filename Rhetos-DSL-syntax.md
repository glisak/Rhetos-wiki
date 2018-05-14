## Declarative programming

The Rhetos DSL is a declarative programming language: The DSL scripts are interpreted as a set of declared features (statements), not a list of imperative commands.

This means that the following changes are **irrelevant** to the generated application and can be ignored:

* The ordering of the statements in the scripts
* Multiple declaration of the same feature
* The arrangement of the statements over multiple DSL scripts

## Literals

Each statement in a DSL script has the following format:

1. starts with the concept's **keyword**,
2. followed by the parameter values,
3. ends with a semicolon (`;`) or an inner block of statements (`{` ... `}`).

Each parameter is **a string** or **a reference** to another statement (feature).

Example:

<pre>
<b>Module</b> Demo
{
    <b>Entity</b> School
    {
        <b>ShortString</b> Name;
        <b>Reference</b> BelongTo Demo.Region;
        <b>ShortString</b> Code
        {
            <b>Unique</b>;
            <b>RegExMatch</b> '\d{5}' 'Must have 5 digits.';
        }
    }

    <b>Entity</b> Region;
}
</pre>

String literals:

* Simple strings such as object names (**identifiers**) do not need parenthesis.
* Strings with spaces and special characters (code snippets, messages) must be enclosed in either **single or double** quotation marks.
  * Coding standard: The single quotes are usually used for C# code snippet and the double quotes are used for SQL code snippets,
  to avoid escaping quoted strings in the respective languages.
* Quotation marks within a string are escaped by typing the quotation mark twice.
* Any string parameter can be placed in an **external file** and referenced from the DSL scripts by the relative path in angle quotes.
  * In the example above: `RegExMatch '\d{5}' <RegexMessage.txt>;`
  * Coding standard: This is recommended for larger SQL queries.

References:

* A reference to another statement is a list of the statement's identifiers (the feature's name), separated by a dot. In the example above, `Demo.Region` after `BelongTo` is a reference to the second entity.

Comments:

* Single-line comments begin with a double slash (`//`).

## Flat vs. embedded statements

Rhetos reads the DSL scripts as a set of somewhat independent statements (with references between them).
Embedding a block of statements within another statement is just an optional syntactic sugar.

The following script is semantically identical to the example above:

```
Module Demo;
Entity Demo.School;
ShortString Demo.School.Name;
Reference Demo.School.BelongTo Demo.Region;
ShortString Demo.School.Code;
Unique Demo.School.Code;
RegExMatch Demo.School.Code '\d{5}' 'Must have 5 digits.';
Entity Demo.Region;
```

Embedding one statement into another is a helper for setting **the first argument** of the statement to reference the parent statement. Notice changes in the `ShortString` placement and syntax in the following three equivalent examples:

```
Module Demo { Entity School { ShortString Name; } }

Module Demo { Entity School; ShortString School.Name; }

Module Demo { Entity School; } ShortString Demo.School.Name;
```

## See also

* [Rhetos coding standard](https://github.com/Rhetos/Rhetos/wiki/Rhetos-coding-standard)
