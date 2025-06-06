# Ruby Rules for Bazel

## Overview

This repository hosts [Ruby][1] language ruleset for [Bazel][2].

The ruleset is known to work with:

- Bazel 8 using WORKSPACE and Bzlmod *(tested on CI)*.
- Bazel 7 using WORKSPACE and Bzlmod *(no longer tested on CI)*.
- Bazel 6 using WORKSPACE and Bzlmod *(no longer tested on CI)*.

## Getting Started

### WORKSPACE

1. Install the ruleset following WORKSPACE instructions on the [latest release][13].
2. Download and install Ruby:

```bazel
# WORKSPACE
load("@rules_ruby//ruby:deps.bzl", "rb_register_toolchains")

rb_register_toolchains(
    version = "3.1.6",
    # alternatively, load version from .ruby-version file
    # version_file = "//:.ruby-version",
)
```

3. *(Optional)* Download and install Bundler dependencies:

```bazel
# WORKSPACE
load("@rules_ruby//ruby:deps.bzl", "rb_bundle_fetch")

rb_bundle_fetch(
    name = "bundle",
    gemfile = "//:Gemfile",
    gemfile_lock = "//:Gemfile.lock",
)
```

4. Start defining your library, binary and test targets in `BUILD` files.

### Bzlmod

1. Install ruleset following Bzlmod instructions on the [latest release][13].
2. Download and install Ruby:

```bazel
# MODULE.bazel
ruby = use_extension("@rules_ruby//ruby:extensions.bzl", "ruby")
ruby.toolchain(
    name = "ruby",
    version = "3.0.6",
    # alternatively, load version from .ruby-version file
    # version_file = "//:.ruby-version",
)
use_repo(ruby, "ruby")
```

3. _(Optional)_ Download and install Bundler dependencies:

```bazel
# MODULE.bazel
ruby.bundle_fetch(
    name = "bundle",
    gemfile = "//:Gemfile",
    gemfile_lock = "//:Gemfile.lock",
)
use_repo(ruby, "bundle", "ruby_toolchains")
```

4. Register Ruby toolchains:

```bazel
# MODULE.bazel
register_toolchains("@ruby_toolchains//:all")
```

4. Start defining your library, binary and test targets in `BUILD` files.

## Documentation

- See [repository rules][3] for the documentation of `WORKSPACE` rules.
- See [rules][4] for the documentation of `BUILD` rules.

## Examples

See [`examples`][14] directory for a comprehensive set of examples how to use the ruleset.

## Toolchains

The following toolchains are known to work and tested on CI.

| Ruby             | Linux | macOS | Windows |
|------------------|-------|-------|---------|
| MRI 3.5          | 🟩    | 🟩    | 🟥      |
| MRI 3.4          | 🟩    | 🟩    | 🟩      |
| MRI 3.3          | 🟩    | 🟩    | 🟩      |
| MRI 3.2          | 🟩    | 🟩    | 🟩      |
| JRuby 10.0       | 🟩    | 🟩    | 🟩      |
| TruffleRuby 24.0 | 🟩    | 🟩    | 🟥      |

The following toolchains were previously known to work but *no longer tested on CI*.

| Ruby             | Linux | macOS | Windows |
|------------------|-------|-------|---------|
| MRI 3.1          | 🟩    | 🟩    | 🟩      |
| MRI 3.0          | 🟩    | 🟩    | 🟩      |
| MRI 2.7          | 🟩    | 🟩    | 🟩      |
| JRuby 9.4        | 🟩    | 🟩    | 🟩      |
| JRuby 9.3        | 🟩    | 🟩    | 🟩      |
| TruffleRuby 23.0 | 🟩    | 🟩    | 🟥      |
| TruffleRuby 22.0 | 🟩    | 🟩    | 🟥      |

### MRI

On Linux and macOS, [ruby-build][5] is used to install MRI from sources.
Keep in mind, that it takes some time for compilation to complete.

On Windows, [RubyInstaller][6] is used to install MRI.

### JRuby

On all operating systems, JRuby is downloaded manually.
It uses Bazel runtime Java toolchain as JDK.
JRuby is currently the only toolchain that supports [Remote Build Execution][15].

### TruffleRuby

On Linux and macOS, [ruby-build][5] is used to install TruffleRuby.
Windows is not supported.

### Other

On Linux and macOS, you can potentially use any Ruby distribution that is supported by [ruby-build][5].
However, some are known not to work or work only partially (e.g. mRuby has no bundler support).

## Known Issues

* JRuby/TruffleRuby might need `HOME` variable exposed.
  See [`examples/gem/.bazelrc`][7] to learn how to do that.
  This is to be fixed in [`jruby/jruby#5661`][9] and [`oracle/truffleruby#2784`][10].
* JRuby might fail with `Errno::EACCES: Permission denied - NUL` error on Windows.
  You need to configure JDK to allow proper access.
  This is described in [`jruby/jruby#7182`][11].
* RuboCop < 1.55 crashes with `LoadError` on Windows.
  This is fixed in [`rubocop/rubocop#12062`][12].
* REPL doesn't work when used with `bazel test`.
  To work it around, use a debugger with remote client support such as [`ruby/debug`][8] .
  See [`examples/gem/.bazelrc`][7] to learn how to do that.
* Some gems contain files with spaces which cause Bazel error `link or target filename contains space`.
  To work it around, use [`--experimental_inprocess_symlink_creation`][16] Bazel flag.
  See [`bazelbuild/bazel#4327`][17] for more details.


[1]: https://www.ruby-lang.org
[2]: https://bazel.build
[3]: docs/repository_rules.md
[4]: docs/rules.md
[5]: https://github.com/rbenv/ruby-build
[6]: https://rubyinstaller.org
[7]: examples/gem/.bazelrc
[8]: https://github.com/ruby/debug
[9]: https://github.com/jruby/jruby/issues/5661
[10]: https://github.com/oracle/truffleruby/issues/2784
[11]: https://github.com/jruby/jruby/issues/7182#issuecomment-1112953015
[12]: https://github.com/rubocop/rubocop/pull/12062
[13]: https://github.com/bazel-contrib/rules_ruby/releases/tag/v0.3.0
[14]: examples/
[15]: https://bazel.build/remote/rbe
[16]: https://bazel.build/reference/command-line-reference#flag--experimental_inprocess_symlink_creation
[17]: https://github.com/bazelbuild/bazel/issues/4327
