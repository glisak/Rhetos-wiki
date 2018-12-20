This roadmap presents high-level priorities, goals and plans for further development of the platform. Each point represents a themed group which will be broken into one or more projects during implementation.

## 1. Better IDE experience

* Embedded C# code IntelliSense
* Better Rhetos DSL IntelliSense
* Seamless build workflow

We want to significantly improve overall experience of developing Rhetos based applications. Specifically, primary focus is to provide full IntelliSense and static code analysis features for Rhetos DSL code as well as embedded C# code used in it.

We would also like developers to be able to seamlessly run (and debug) their Rhetos based applications from within IDE in the same way they can run a regular C# application.

## 2. Learning resources

* Code samples
* Tutorials
* Issues database
* Full application samples

We are not satisfied with currently available resources for learning Rhetos. Absence of basic *Getting started* tutorials for all Rhetos concepts makes it difficult to start using the platform. We will focus on build an extensive repository of small and useful documentation and code fragments, each addressing a specific Rhetos paradigm/practice.

Instead, we will focus on the above elements and incrementally build an extensive repository of small and useful documentation and code fragments - each addressing a specific Rhetos paradigm/practice.

Motivating as many Rhetos community members to contribute to this point is critical to its success.

## 3. Complex web API methods

The existing REST service exposes only simple objects by default. For example, if developer needs to read or write a master-detail object in a single request, a custom code must be developed, sometimes with custom data serialization within already serialized web requests.

Our goal is to allow developers to simply create view-models or DTOs, including complex hierarchical objects, using only declarative code, without need for developing middleware service or custom web methods.

## 4. Migration to .NET Core

With .NET Core gaining so much momentum, especially in the open-source space, we feel it is a natural next step for Rhetos platform.

We want to fully migrate Rhetos and all its components to .NET Core platform. Expected benefits are: better performance, modern tooling and cross-platform support.

## Timeline

This roadmap provides only high-level priorities and plans and does not include a specific timeline. As development on these points gets planned we will post further information.

## Related resources

* [Release notes](https://github.com/Rhetos/Rhetos/blob/master/ChangeLog.md) for previous releases
* Short-term [Milestones](https://github.com/Rhetos/Rhetos/milestones)
