# gowebsite-gomodules-ja
Go 公式が公開している Go Modules に関するドキュメントを日本語訳するプロジェクトです。

## Getting started
### [Tutorial: Create a module](https://github.com/golang/website/blob/master/_content/doc/tutorial/create-module.html)
A tutorial of short topics introducing functions, error handling, arrays, maps, unit testing, and compiling.

## Developing modules
### [Developing and publishing modules](https://github.com/golang/website/blob/master/_content/doc/modules/developing.md)
You can collect related packages into modules, then publish the modules for other developers to use. This topic gives an overview of developing and publishing modules.

### [Module release and versioning workflow](https://github.com/golang/website/blob/master/_content/doc/modules/release-workflow.md)
When you develop modules for use by other developers, you can follow a workflow that helps ensure a reliable, consistent experience for developers using the module. This topic describes the high-level steps in that workflow.

### [Managing module source](https://github.com/golang/website/blob/master/_content/doc/modules/managing-source.md)
When you're developing modules to publish for others to use, you can help ensure that your modules are easier for other developers to use by following the repository conventions described in this topic.

### [Developing a major version update](https://github.com/golang/website/blob/master/_content/doc/modules/major-version.md)
A major version update can be very disruptive to your module's users because it includes breaking changes and represents a new module. Learn more in this topic.

### [Publishing a module](https://github.com/golang/website/blob/master/_content/doc/modules/publishing.md)
When you want to make a module available for other developers, you publish it so that it's visible to Go tools. Once you've published the module, developers importing its packages will be able to resolve a dependency on the module by running commands such as go get.

### [Module version numbering](https://github.com/golang/website/blob/master/_content/doc/modules/version-numbers.md)
A module's developer uses each part of a module's version number to signal the version’s stability and backward compatibility. For each new release, a module's release version number specifically reflects the nature of the module's changes since the preceding release.

## References
### [Go Modules Reference](https://github.com/golang/website/blob/master/_content/doc/mod.md)
A detailed reference manual for Go's dependency management system.

## [go.mod file reference](https://github.com/golang/website/blob/master/_content/doc/modules/gomod-ref.md)
Reference for the directives included in a go.mod file.
