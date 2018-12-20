Contributions are very welcome. The easiest way is to contribute is to **fork** the repository, make changes and then make a **pull request** from your fork.

GitHub provides a detailed documentation on this process ([Fork a repo](https://help.github.com/articles/fork-a-repo/)) and working with contributions in general ([Collaborating with issues and pull requests](https://help.github.com/categories/collaborating-with-issues-and-pull-requests/)). The first time you make a pull request, you may be asked to sign a Contributor Agreement.

## Wiki articles

Our documentation is stored on [wiki](https://github.com/Rhetos/Rhetos/wiki) pages. Contributions are done by creating a fork from the [Rhetos-wiki](https://github.com/Rhetos/Rhetos-wiki) repository, and submitting the pull requests there (see the GitHub notes above).

How to write documentation:

* The wiki pages are written in [markdown format](https://guides.github.com/features/mastering-markdown/). Use an offline text editor with spell checker and markdown lint tool to make sure there are no issues with the test formatting. For example, use **VS Code** with plugins: "Spell Right", "markdownlint" and "Markdown All in One" to write the documents.
* Skip the first-level heading in the article, because the GitHub wiki adds it automatically.

Please see the specific guidelines for [Writing tutorial articles](Writing-tutorial-articles).

## Questions, bug reports and features requests

Our [Issues list](https://github.com/Rhetos/Rhetos/issues) is a place to ask questions, discuss the existing issues, report a bug or request a new feature.

When creating a new issue, you will be asked to choose the issue type: question, bug or feature.

## Framework and plugins development

When deciding on what to change and how to implement the change, use our [Issues list](https://github.com/Rhetos/Rhetos/issues) for ideas, community requests and discussion on the implementation details. Additionally, [Roadmap](Rhetos-platform-roadmap) contains a valuable information on our long-term goals and the general direction of development.

Before submitting a pull request:

* Please run the unit tests before submitting the pull request.
  Rhetos contains two types of unit tests: standard unit tests (in Rhetos.sln), and integration tests (in CommonConceptsTest.sln) that can be run only after a successful deployment.
* Check if you code conforms to the [Rhetos coding standard](Rhetos-coding-standard).
* Some changes that could break the backward compatibility should be implemented with a legacy-support option:
  [Backward compatible feature implementation](Backward-compatible-feature-implementation-in-Rhetos-and-CommonConcepts)
