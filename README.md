
[![recorded video][youtube-preview]][youtube-video]

This is a quick demo and presentation about go1.11 modules based on my (very
limited) experience. This talk was given at the
[Boulder Gophers][boulder-gophers] meetup.

# Before We Get Started

- Poll the audience: Who has ran vgo, who has used modules?
- I'm not an expert at this stuff. I haven't used it on any projects yet but
  I think what exists in Go 1.11 is an improvement on the current dependency
  management situation.

# What is a Module

- A module is a tree of packages that share a common prefix and are versioned
  together.
- This is typically a repository but you can have multiple modules per repo if
  needed.
- Modules in 1.11 includes changes to the Go toolchain.

# Import Compatability Rule

- Modules follow the "Import Compatability Rule":

> If an old package and a new package have the same import path, the new
> package must be backwards compatible with the old package.

- Then how do you introduce breaking changes?
- Major versions above v1 are included in import path. So v1 and v2 of a
  module have different import paths. For example
   - import "foo"    // pre-1.x.x
   - import "foo"    // 1.x.x
   - import "foo/v2" // 2.x.x
   - import "foo/v3" // 3.x.x
- This means your final program can use multiple major versions of a module.

# Using Modules

- A module is defined by a `go.mod` file being in the ancestry of the
  directory.
- This file contains the module path as well as all dependencies and versions.

In order to use modules you can either:

- Put your code outside of GOPATH and create a go.mod file
- Set `GO111MODULE=on`

# Demo

```
docker run -it --rm -v $PWD:/my-module/cmd golang:1.11rc1

# most of the new functionality is under go mod
go mod

# lots of good information under go help
go help modules

# to create a new module
go mod init github.com/milehighgophers/mod-demo
cat go.mod

# build, test and list will update the go.mod file
go build -o demo github.com/milehighgophers/mod-demo/cmd
cat go.mod

# there is also a checksum file created
ls -la
cat go.sum

# you can use that to verify the bits you are using to build are what you
# expect
sed -i'' 's/h1:/h1:a/' go.sum
cat go.sum
go mod verify
rm go.sum

# to view all dependencies for this modules
go list -m all

# to test all packages for this module
go test all

# vendor is going away! yay! however it is not gone entirely.
# vendor is limited to the top-level module being built only, to populate this
# directory use go mod vendor
go mod vendor
ls -la vendor
rm -rf vendor

# go mod why tells you why a package was included
go mod why golang.org/x/text/language

# go mod tidy updates go.mod requirements with a view of all packages and
# build configurations for that module
go mod tidy

# to build without downloading deps (useful for CI)
go build -mod=readonly
```

# Minimum Version Selection

- With MVS each module defines the minimum version of their dependencies that
  they require.
- The algorithm will find a set of dependencies that satisfies all the module's
  requirements.
- There is always a solution to the MVS, if that solution builds and is
  correct is another question.
- The latest version is used by default when you add the dependency.
- New versions are not automatically incorporated simply because they have
  been published, they have to be listed in a go.mod.
- MVS doesn't require a NP-complete SAT solver so it is much simpler to reason
  about and faster to run.
- You can use replace and exclude to control for incompatible module versions.

# Community and Controversy

- Russ's solution departs from the way this problem is solved in other popular
  languages.
- The dep folks are justifiably a little sour about the whole thing.
- There have been lots of criticisms about how decisions were made, about MVS,
  and that the work put into dep was not incorporated.
- This is anecdotal, but most folks I talk to enjoy both vgo and modules. I
  have only heard complaints about dep. It is slow and, personally, working on
  kubernetes, I have ran into issues where I had to abandon its use for older
  tools like Godep.
- The more I learn about it the more I think modules are a home run for Go.
  Go is meant to be a language that scales to massive projects. MVS fits well
  with that design goal.
- Only time will tell.

[youtube-video]: https://www.youtube.com/watch?v=8-X48nI7LK4
[youtube-preview]: https://github.com/jasonkeene/go-modules-demo/raw/master/preview.jpg
[boulder-gophers]: https://www.meetup.com/Boulder-Gophers/events/pvwcrpyxlbvb/
