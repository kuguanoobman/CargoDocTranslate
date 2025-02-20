{==+==}
## Configuration
{==+==}
## 配置
{==+==}

{==+==}
This document explains how Cargo’s configuration system works, as well as
available keys or configuration. For configuration of a package through its
manifest, see the [manifest format](manifest.md).
{==+==}
本文档解释了Cargo的配置系统如何工作，以及可用的键或配置。关于通过包的配置清单进行的配置，参阅[配置清单格式](manifest.md)。
{==+==}

{==+==}
### Hierarchical structure
{==+==}
### 层次结构
{==+==}

{==+==}
Cargo allows local configuration for a particular package as well as global
configuration. It looks for configuration files in the current directory and
all parent directories. If, for example, Cargo were invoked in
`/projects/foo/bar/baz`, then the following configuration files would be
probed for and unified in this order:
{==+==}
Cargo允许对某一特定包进行本地配置，也允许进行全局配置。它在当前目录和所有父目录下寻找配置文件。例如，如果在`/projects/foo/bar/baz`中调用Cargo，那么以下的配置文件会被探测到，并按照这个顺序统一起来。
{==+==}

{==+==}
* `/projects/foo/bar/baz/.cargo/config.toml`
* `/projects/foo/bar/.cargo/config.toml`
* `/projects/foo/.cargo/config.toml`
* `/projects/.cargo/config.toml`
* `/.cargo/config.toml`
* `$CARGO_HOME/config.toml` which defaults to:
    * Windows: `%USERPROFILE%\.cargo\config.toml`
    * Unix: `$HOME/.cargo/config.toml`
{==+==}
* `/projects/foo/bar/baz/.cargo/config.toml`
* `/projects/foo/bar/.cargo/config.toml`
* `/projects/foo/.cargo/config.toml`
* `/projects/.cargo/config.toml`
* `/.cargo/config.toml`
* `$CARGO_HOME/config.toml` 默认为:
    * Windows: `%USERPROFILE%\.cargo\config.toml`
    * Unix: `$HOME/.cargo/config.toml`
{==+==}

{==+==}
With this structure, you can specify configuration per-package, and even
possibly check it into version control. You can also specify personal defaults
with a configuration file in your home directory.
{==+==}
有了这种结构，你可以指定每个包的配置，甚至可以将其检查到版本控制中。你也可以在你的主目录下用一个配置文件指定个人默认值。
{==+==}

{==+==}
If a key is specified in multiple config files, the values will get merged
together. Numbers, strings, and booleans will use the value in the deeper
config directory taking precedence over ancestor directories, where the
home directory is the lowest priority. Arrays will be joined together.
{==+==}
如果一个键在多个配置文件中被指定，这些值将被合并在一起。数字、字符串和布尔值将使用更深层级配置目录中的值，其优先于祖先目录的，其主目录的优先级最低。数组将被连接在一起。
{==+==}

{==+==}
At present, when being invoked from a workspace, Cargo does not read config
files from crates within the workspace. i.e. if a workspace has two crates in
it, named `/projects/foo/bar/baz/mylib` and `/projects/foo/bar/baz/mybin`, and
there are Cargo configs at `/projects/foo/bar/baz/mylib/.cargo/config.toml`
and `/projects/foo/bar/baz/mybin/.cargo/config.toml`, Cargo does not read
those configuration files if it is invoked from the workspace root
(`/projects/foo/bar/baz/`).
{==+==}
目前，当从工作区调用时，Cargo不会从工作区的crates中读取配置文件，即如果一个工作区有两个crates，分别名为 `/projects/foo/bar/baz/mylib` 和 `/projects/foo/bar/baz/mybin` ，并且存在 `/projects/foo/bar/baz/mylib/.cargo/config.toml` 和 `/projects/foo/bar/baz/mybin/.cargo/config.toml` ，如果从工作区根(`/projects/foo/bar/baz/`)调用，Cargo就不会读取这些配置文件。
{==+==}

{==+==}
> **Note:** Cargo also reads config files without the `.toml` extension, such as
> `.cargo/config`. Support for the `.toml` extension was added in version 1.39
> and is the preferred form. If both files exist, Cargo will use the file
> without the extension.
{==+==}
> **注意:** Cargo 也可以读取没有 `.toml` 扩展名的配置文件，如`.cargo/config`。
> 对`.toml`扩展的支持是在1.39版本中加入的，是首选的形式。
> 如果两个文件都存在，Cargo 将使用没有扩展名的文件。
{==+==}

{==+==}
### Configuration format
{==+==}
### 配置格式
{==+==}

{==+==}
Configuration files are written in the [TOML format][toml] (like the
manifest), with simple key-value pairs inside of sections (tables). The
following is a quick overview of all settings, with detailed descriptions
found below.
{==+==}
配置文件是以 [TOML格式][toml] 编写的，在表内有简洁的键值对。以下是对所有设置的快速概述，详细说明见下文。
{==+==}

{==+==}
```toml
paths = ["/path/to/override"] # path dependency overrides
{==+==}
paths = ["/path/to/override"] # 路径依赖覆盖
{==+==}

{==+==}
[alias]     # command aliases
b = "build"
c = "check"
t = "test"
r = "run"
rr = "run --release"
recursive_example = "rr --example recursions"
space_example = ["run", "--release", "--", "\"command list\""]
{==+==}
[alias]     # 命令别名
b = "build"
c = "check"
t = "test"
r = "run"
rr = "run --release"
recursive_example = "rr --example recursions"
space_example = ["run", "--release", "--", "\"command list\""]
{==+==}

{==+==}
[build]
jobs = 1                      # number of parallel jobs, defaults to # of CPUs
rustc = "rustc"               # the rust compiler tool
rustc-wrapper = "…"           # run this wrapper instead of `rustc`
rustc-workspace-wrapper = "…" # run this wrapper instead of `rustc` for workspace members
rustdoc = "rustdoc"           # the doc generator tool
target = "triple"             # build for the target triple (ignored by `cargo install`)
target-dir = "target"         # path of where to place all generated artifacts
rustflags = ["…", "…"]        # custom flags to pass to all compiler invocations
rustdocflags = ["…", "…"]     # custom flags to pass to rustdoc
incremental = true            # whether or not to enable incremental compilation
dep-info-basedir = "…"        # path for the base directory for targets in depfiles
{==+==}
[build]
jobs = 1                      # 并行任务数, 默认为 CPU 数
rustc = "rustc"               # rust  编译器工具
rustc-wrapper = "…"           # 运行这个包装器，而不是 `rustc`
rustc-workspace-wrapper = "…" # 运行这个包装器，而不是 `rustc` ，对于工作区成员
rustdoc = "rustdoc"           # 文档生成工具
target = "triple"             # 构建 target triple (被 `cargo install` 忽略)
target-dir = "target"         # 放置所有生成制品的路径
rustflags = ["…", "…"]        # 要传递给所有编译器调用的自定义标志
rustdocflags = ["…", "…"]     # 传递给rustdoc的自定义标志
incremental = true            # 是否启用增量编译
dep-info-basedir = "…"        # 开发文件中目标的基础目录路径
{==+==}

{==+==}
[doc]
browser = "chromium"          # browser to use with `cargo doc --open`,
                              # overrides the `BROWSER` environment variable
{==+==}
[doc]
browser = "chromium"          # 与 `cargo doc --open` 一起使用的浏览器，覆盖 `BROWSER` 环境变量。
{==+==}

{==+==}
[env]
# Set ENV_VAR_NAME=value for any process run by Cargo
ENV_VAR_NAME = "value"
# Set even if already present in environment
ENV_VAR_NAME_2 = { value = "value", force = true }
# Value is relative to .cargo directory containing `config.toml`, make absolute
ENV_VAR_NAME_3 = { value = "relative/path", relative = true }
{==+==}
[env]
# 为Cargo运行的任何进程设置ENV_VAR_NAME=value
ENV_VAR_NAME = "value"
# 即使环境中已经存在，也将设置
ENV_VAR_NAME_2 = { value = "value", force = true }
# 该值是相对于包含 `config.toml` 的.cargo目录而言的，设置为绝对的。
ENV_VAR_NAME_3 = { value = "relative/path", relative = true }
{==+==}

{==+==}
[future-incompat-report]
frequency = 'always' # when to display a notification about a future incompat report
{==+==}
[future-incompat-report]
frequency = 'always' # 何时显示关于未来不兼容报告的通知
{==+==}

{==+==}
[cargo-new]
vcs = "none"              # VCS to use ('git', 'hg', 'pijul', 'fossil', 'none')
{==+==}
[cargo-new]
vcs = "none"              # VCS 便用 ('git', 'hg', 'pijul', 'fossil', 'none')
{==+==}

{==+==}
[http]
debug = false               # HTTP debugging
proxy = "host:port"         # HTTP proxy in libcurl format
ssl-version = "tlsv1.3"     # TLS version to use
ssl-version.max = "tlsv1.3" # maximum TLS version
ssl-version.min = "tlsv1.1" # minimum TLS version
timeout = 30                # timeout for each HTTP request, in seconds
low-speed-limit = 10        # network timeout threshold (bytes/sec)
cainfo = "cert.pem"         # path to Certificate Authority (CA) bundle
check-revoke = true         # check for SSL certificate revocation
multiplexing = true         # HTTP/2 multiplexing
user-agent = "…"            # the user-agent header
{==+==}
[http]
debug = false               # HTTP 调试
proxy = "host:port"         # libcurl格式的HTTP代理
ssl-version = "tlsv1.3"     # 要使用的TLS版本
ssl-version.max = "tlsv1.3" # TLS 最大版本
ssl-version.min = "tlsv1.1" # TLS 最小版本
timeout = 30                # 每个HTTP请求的超时，单位为秒
low-speed-limit = 10        # 网络超时阈值(字节/秒)。
cainfo = "cert.pem"         # CA证书绑定的路径
check-revoke = true         # 检查SSL证书是否被撤销
multiplexing = true         # HTTP/2 多路复用
user-agent = "…"            # user-agent 头
{==+==}

{==+==}
[install]
root = "/some/path"         # `cargo install` destination directory
{==+==}
[install]
root = "/some/path"         # `cargo install` 目标目录
{==+==}

{==+==}
[net]
retry = 2                   # network retries
git-fetch-with-cli = true   # use the `git` executable for git operations
offline = true              # do not access the network
{==+==}
[net]
retry = 2                   # 网络重试
git-fetch-with-cli = true   # 使用 `git` 可执行文件进行git操作
offline = true              # 不接入网络
{==+==}

{==+==}
[patch.<registry>]
# Same keys as for [patch] in Cargo.toml
{==+==}
[patch.<registry>]
# 与Cargo.toml中 [patch] 的键相同。
{==+==}

{==+==}
[profile.<name>]         # Modify profile settings via config.
opt-level = 0            # Optimization level.
debug = true             # Include debug info.
split-debuginfo = '...'  # Debug info splitting behavior.
debug-assertions = true  # Enables debug assertions.
overflow-checks = true   # Enables runtime integer overflow checks.
lto = false              # Sets link-time optimization.
panic = 'unwind'         # The panic strategy.
incremental = true       # Incremental compilation.
codegen-units = 16       # Number of code generation units.
rpath = false            # Sets the rpath linking option.
[profile.<name>.build-override]  # Overrides build-script settings.
# Same keys for a normal profile.
[profile.<name>.package.<name>]  # Override profile for a package.
# Same keys for a normal profile (minus `panic`, `lto`, and `rpath`).
{==+==}
[profile.<name>]         # 通过配置来修改配置文件设置。
opt-level = 0            # 优化级别。
debug = true             # 包括调试信息。
split-debuginfo = '...'  # 调试信息拆分行为。
debug-assertions = true  # 启用调试断言。
overflow-checks = true   # 启用运行时的整数溢出检查。
lto = false              # 设置链接时优化。
panic = 'unwind'         # 恐慌策略。
incremental = true       # 增量编译。
codegen-units = 16       # 代码生成单元的数目。
rpath = false            # 设置rpath链接选项。
[profile.<name>.build-override]  # 覆盖build-script的设置。
# profile下的键是一样的。
[profile.<name>.package.<name>]  # 覆盖包的配置文件。
# profile下的键是一样的 (减去 `panic`, `lto`, 和 `rpath`)。
{==+==}

{==+==}
[registries.<name>]  # registries other than crates.io
index = "…"          # URL of the registry index
token = "…"          # authentication token for the registry
{==+==}
[registries.<name>]  # crates.io外的其他注册中心
index = "…"          # 注册中心的URL索引
token = "…"          # 注册中心的认证令牌
{==+==}

{==+==}
[registry]
default = "…"        # name of the default registry
token = "…"          # authentication token for crates.io
{==+==}
[registry]
default = "…"        # 默认注册中心的名称
token = "…"          # crates.io的认证令牌
{==+==}

{==+==}
[source.<name>]      # source definition and replacement
replace-with = "…"   # replace this source with the given named source
directory = "…"      # path to a directory source
registry = "…"       # URL to a registry source
local-registry = "…" # path to a local registry source
git = "…"            # URL of a git repository source
branch = "…"         # branch name for the git repository
tag = "…"            # tag name for the git repository
rev = "…"            # revision for the git repository
{==+==}
[source.<name>]      # 源的定义和替换
replace-with = "…"   # 用给定命名的源替换这个源
directory = "…"      # 目录源的路径
registry = "…"       # 注册中心源的 URL
local-registry = "…" # 本地注册源的路径
git = "…"            # git 源的 URL
branch = "…"         # git仓库的分支名称
tag = "…"            # git仓库的tag名称
rev = "…"            # git仓库的修订
{==+==}

{==+==}
[target.<triple>]
linker = "…"            # linker to use
runner = "…"            # wrapper to run executables
rustflags = ["…", "…"]  # custom flags for `rustc`
{==+==}
[target.<triple>]
linker = "…"            # 要使用的链接器
runner = "…"            # 运行可执行文件的包装器
rustflags = ["…", "…"]  # `rustc` 的自定义标记
{==+==}

{==+==}
[target.<cfg>]
runner = "…"            # wrapper to run executables
rustflags = ["…", "…"]  # custom flags for `rustc`
{==+==}
[target.<cfg>]
runner = "…"            # 运行可执行文件的包装器
rustflags = ["…", "…"]  # `rustc` 的自定义标记
{==+==}

{==+==}
[target.<triple>.<links>] # `links` build script override
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = ["-L", "/some/path"]
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
{==+==}
[target.<triple>.<links>] # `links` 构建脚本覆盖
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = ["-L", "/some/path"]
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
{==+==}

{==+==}
[term]
quiet = false          # whether cargo output is quiet
verbose = false        # whether cargo provides verbose output
color = 'auto'         # whether cargo colorizes output
progress.when = 'auto' # whether cargo shows progress bar
progress.width = 80    # width of progress bar
```
{==+==}
[term]
quiet = false          # cargo是否输出静默
verbose = false        # cargo是否提供详细输出
color = 'auto'         # cargo是否着色输出
progress.when = 'auto' # cargo是否显示进度条
progress.width = 80    # 进度条的宽度
```
{==+==}

{==+==}
### Environment variables
{==+==}
### 环境变量
{==+==}

{==+==}
Cargo can also be configured through environment variables in addition to the
TOML configuration files. For each configuration key of the form `foo.bar` the
environment variable `CARGO_FOO_BAR` can also be used to define the value.
Keys are converted to uppercase, dots and dashes are converted to underscores.
For example the `target.x86_64-unknown-linux-gnu.runner` key can also be
defined by the `CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_RUNNER` environment
variable.
{==+==}
除了TOML配置文件，Cargo还可以通过环境变量进行配置。对于每个形如 `foo.bar` 的配置键，也可以用环境变量 `CARGO_FOO_BAR` 定义值。
键被转换为大写字母，点和破折号被转换为下划线。例如，`target.x86_64-unknown-linux-gnu.runner` 键也可以由 `CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_RUNNER` 环境变量定义。
{==+==}

{==+==}
Environment variables will take precedence over TOML configuration files.
Currently only integer, boolean, string and some array values are supported to
be defined by environment variables. [Descriptions below](#configuration-keys)
indicate which keys support environment variables and otherwise they are not
supported due to [technicial issues](https://github.com/rust-lang/cargo/issues/5416).
{==+==}
环境变量将优先于TOML配置文件。目前只有整数、布尔值、字符串和一些数组值支持由环境变量定义。
[下面描述](#configuration-keys)表明哪些键支持环境变量，否则由于[技术问题](https://github.com/rust-lang/cargo/issues/5416)而不受支持。
{==+==}

{==+==}
In addition to the system above, Cargo recognizes a few other specific
[environment variables][env].
{==+==}
除了上述系统外，Cargo还能识别一些其他特定的[环境变量][env]。
{==+==}

{==+==}
### Command-line overrides
{==+==}
### 命令行覆盖
{==+==}

{==+==}
Cargo also accepts arbitrary configuration overrides through the
`--config` command-line option. The argument should be in TOML syntax of
`KEY=VALUE`:
{==+==}
Cargo也可以通过 `--config` 命令行选项接受任意的配置覆盖。参数应是TOML语法中的 "KEY=VALUE" 。
{==+==}

{==+==}
```console
cargo --config net.git-fetch-with-cli=true fetch
```
{==+==}
```console
cargo --config net.git-fetch-with-cli=true fetch
```
{==+==}

{==+==}
The `--config` option may be specified multiple times, in which case the
values are merged in left-to-right order, using the same merging logic
that is used when multiple configuration files apply. Configuration
values specified this way take precedence over environment variables,
which take precedence over configuration files.
{==+==}
`--config` 选项可以被多次指定，在这种情况下，这些值将按照从左到右的顺序合并，使用与适用多个配置文件时相同的合并逻辑。
以这种方式指定的配置值优先于环境变量，环境变量优先于配置文件。
{==+==}

{==+==}
Some examples of what it looks like using Bourne shell syntax:
{==+==}
一些看起像什么Bourne shell语法使用的例子:
{==+==}

{==+==}
```console
# Most shells will require escaping.
cargo --config http.proxy=\"http://example.com\" …
{==+==}
```console
# 多数 shells 请求将转义。
cargo --config http.proxy=\"http://example.com\" …
{==+==}

{==+==}
# Spaces may be used.
cargo --config "net.git-fetch-with-cli = true" …
{==+==}
# 可以使用空格。
cargo --config "net.git-fetch-with-cli = true" …
{==+==}

{==+==}
# TOML array example. Single quotes make it easier to read and write.
cargo --config 'build.rustdocflags = ["--html-in-header", "header.html"]' …
{==+==}
# TOML数组例子。单引号使其更容易读写。
cargo --config 'build.rustdocflags = ["--html-in-header", "header.html"]' …
{==+==}

{==+==}
# Example of a complex TOML key.
cargo --config "target.'cfg(all(target_arch = \"arm\", target_os = \"none\"))'.runner = 'my-runner'" …
{==+==}
# 一个复杂的TOML键的例子。
cargo --config "target.'cfg(all(target_arch = \"arm\", target_os = \"none\"))'.runner = 'my-runner'" …
{==+==}

{==+==}
# Example of overriding a profile setting.
cargo --config profile.dev.package.image.opt-level=3 …
```
{==+==}
# 覆盖一个配置文件设置的例子。
cargo --config profile.dev.package.image.opt-level=3 …
```
{==+==}

{==+==}
The `--config` option can also be used to pass paths to extra
configuration files that Cargo should use for a specific invocation.
Options from configuration files loaded this way follow the same
precedence rules as other options specified directly with `--config`.
{==+==}
 `--config` 选项也可以用来传递额外的配置文件的路径，Cargo应该在特定的调用中使用这些文件。
通过这种方式加载的配置文件中的选项与直接用 `--config` 指定的其他选项遵循同样的优先级规则。
{==+==}

{==+==}
### Config-relative paths
{==+==}
### Config相对路径
{==+==}

{==+==}
Paths in config files may be absolute, relative, or a bare name without any path separators.
Paths for executables without a path separator will use the `PATH` environment variable to search for the executable.
Paths for non-executables will be relative to where the config value is defined.
{==+==}
配置文件中的路径可以是绝对的、相对的，或者是没有任何路径分隔符的无修饰名称。
没有路径分隔符的可执行文件的路径将使用 `PATH` 环境变量来搜索可执行文件。
非可执行文件的路径将相对于配置值定义位置。
{==+==}

{==+==}
In particular, rules are:
{==+==}
具体来说，规则是:
{==+==}

{==+==}
* For environment variables, paths are relative to the current working directory.
* For config values loaded directly from the [`--config KEY=VALUE`](#command-line-overrides) option,
  paths are relative to the current working directory.
* For config files, paths are relative to the parent directory of the directory where the config files were defined,
  no matter those files are from either the [hierarchical probing](#hierarchical-structure)
  or the [`--config <path>`](#command-line-overrides) option.
{==+==}
* 对于环境变量，路径是相对于当前工作目录的。
* 对于直接从 [`--config KEY=VALUE`](#command-line-overrides) 选项加载的配置值，路径是相对于当前工作目录的。
* 对于配置文件，路径是相对于定义配置文件的目录的父目录而言的，无论这些文件是来自 [层级检索](#hierarchical-structure) 还是 [`--config <path>`](#command-line-overrides) 选项。
{==+==}

{==+==}
> **Note:** To maintain consistency with existing `.cargo/config.toml` probing behavior,
> it is by design that a path in a config file passed via `--config <path>`
> is also relative to two levels up from the config file itself.
>
> To avoid unexpected results, the rule of thumb is putting your extra config files
> at the same level of discovered `.cargo/config.toml` in your project.
> For instance, given a project `/my/project`,
> it is recommended to put config files under `/my/project/.cargo`
> or a new directory at the same level, such as `/my/project/.config`.
{==+==}
> **注意:** 为了与现有的 `.cargo/config.toml` 检索行为保持一致，在设计上，通过 `--config <path>` 传递的配置文件中的路径也是相对于配置文件本身两级。
> 为了避免意外的结果，经验方法是把你的额外配置文件放在项目中发现的 `.cargo/config.toml` 的同一级别。
> 例如，给定项目 `/my/project` ，建议把配置文件放在 `/my/project/.cargo` 下，或者在同一级别上新建一个目录，如 `/my/project/.config` 。
{==+==}

{==+==}
```toml
# Relative path examples.
{==+==}
```toml
# 相对路径例子。
{==+==}

{==+==}
[target.x86_64-unknown-linux-gnu]
runner = "foo"  # Searches `PATH` for `foo`.
{==+==}
[target.x86_64-unknown-linux-gnu]
runner = "foo"  # 在 `PATH` 中搜索 `foo` .
{==+==}

{==+==}
[source.vendored-sources]
# Directory is relative to the parent where `.cargo/config.toml` is located.
# For example, `/my/project/.cargo/config.toml` would result in `/my/project/vendor`.
directory = "vendor"
```
{==+==}
[source.vendored-sources]
# 目录是相对于 `.cargo/config.toml` 所在的父目录而言的。
# 比如, `/my/project/.cargo/config.toml` 会在 `/my/project/vendor` 。
directory = "vendor"
```
{==+==}

{==+==}
### Executable paths with arguments
{==+==}
### 带参数的可执行路径
{==+==}

{==+==}
Some Cargo commands invoke external programs, which can be configured as a path
and some number of arguments.
{==+==}
一些Cargo命令调用外部程序，可以将其配置为路径和一些参数。
{==+==}

{==+==}
The value may be an array of strings like `['/path/to/program', 'somearg']` or
a space-separated string like `'/path/to/program somearg'`. If the path to the
executable contains a space, the list form must be used.
{==+==}
该值可以是一个字符串数组，如 `['/path/to/program', 'somearg']` 或一个空格分隔的字符串，如 `'/path/to/program somearg'` 。如果可执行文件的路径包含一个空格，则必须使用列表形式。
{==+==}

{==+==}
If Cargo is passing other arguments to the program such as a path to open or
run, they will be passed after the last specified argument in the value of an
option of this format. If the specified program does not have path separators,
Cargo will search `PATH` for its executable.
{==+==}
如果Cargo向程序传递其他参数，比如打开或运行的路径，它们将在这种格式的选项值中最后指定的参数之后传递。如果指定的程序没有路径分隔符，Cargo会在 `PATH` 中搜索其可执行文件。
{==+==}

{==+==}
### Credentials
{==+==}
### 证书
{==+==}

{==+==}
Configuration values with sensitive information are stored in the
`$CARGO_HOME/credentials.toml` file. This file is automatically created and updated
by [`cargo login`]. It follows the same format as Cargo config files.
{==+==}
带有敏感信息的配置值存储在 `$CARGO_HOME/credentials.toml` 文件中。这个文件由 [`cargo login`] 自动创建和更新。它遵循与Cargo配置文件相同的格式。
{==+==}

{==+==}
```toml
[registry]
token = "…"   # Access token for crates.io

[registries.<name>]
token = "…"   # Access token for the named registry
```
{==+==}
```toml
[registry]
token = "…"   # crates.io 访问 token

[registries.<name>]
token = "…"   # 具名注册中心访问 token
```
{==+==}

{==+==}
Tokens are used by some Cargo commands such as [`cargo publish`] for
authenticating with remote registries. Care should be taken to protect the
tokens and to keep them secret.
{==+==}
一些Cargo命令使用令牌，例如[`cargo publish`]，用于与远程注册中心进行认证。应该注意保护令牌，使其私密。
{==+==}

{==+==}
As with most other config values, tokens may be specified with environment
variables. The token for [crates.io] may be specified with the
`CARGO_REGISTRY_TOKEN` environment variable. Tokens for other registries may
be specified with environment variables of the form
`CARGO_REGISTRIES_<name>_TOKEN` where `<name>` is the name of the registry in
all capital letters.
{==+==}
与其他大多数配置值一样，令牌可以用环境变量来指定。
[crates.io]的令牌可以用 `CARGO_REGISTRY_TOKEN` 环境变量来指定。
其他注册中心的令牌可以用 `CARGO_REGISTRIES_<name>_TOKEN` 形式的环境变量来指定，其中 `<name>` 是大写的注册中心的名称。
{==+==}

{==+==}
### Configuration keys
{==+==}
### 配置键
{==+==}

{==+==}
This section documents all configuration keys. The description for keys with
variable parts are annotated with angled brackets like `target.<triple>` where
the `<triple>` part can be any target triple like
`target.x86_64-pc-windows-msvc`.
{==+==}
本节记录了所有的配置键。带有可变部分的键的描述用角括号标注，如 `target.<triple>` ，其中 `<triple>` 部分可以是任何目标三元组，如 `target.x86_64-pc-windows-msvc` 。
{==+==}

{==+==}
#### `paths`
* Type: array of strings (paths)
* Default: none
* Environment: not supported
{==+==}
#### `paths`
* Type: 字符串数组 (paths)
* Default: none
* Environment: 不受支持
{==+==}

{==+==}
An array of paths to local packages which are to be used as overrides for
dependencies. For more information see the [Overriding Dependencies
guide](overriding-dependencies.md#paths-overrides).
{==+==}
一个本地包的路径数组，这些包将被用作依赖覆盖。
更多信息请参阅 [覆盖依赖指南](overriding-dependencies.md#paths-overrides) 。
{==+==}

{==+==}
#### `[alias]`
* Type: string or array of strings
* Default: see below
* Environment: `CARGO_ALIAS_<name>`
{==+==}
#### `[alias]`
* Type: 字符串或字符串数组
* Default: 参阅下面
* Environment: `CARGO_ALIAS_<name>`
{==+==}

{==+==}
The `[alias]` table defines CLI command aliases. For example, running `cargo
b` is an alias for running `cargo build`. Each key in the table is the
subcommand, and the value is the actual command to run. The value may be an
array of strings, where the first element is the command and the following are
arguments. It may also be a string, which will be split on spaces into
subcommand and arguments. The following aliases are built-in to Cargo:
{==+==}
`[alias]` 表定义了CLI命令的别名。例如，运行 `cargo b` 是运行 `cargo build` 的别名。表中的每个键是子命令，而值是实际要运行的命令。
值可以是一个字符串数组，其中第一个元素是命令，后面的元素是参数。它也可以是一个字符串，它将在空格处被分割成子命令和参数。以下是Cargo内置的别名。
{==+==}

{==+==}
```toml
[alias]
b = "build"
c = "check"
d = "doc"
t = "test"
r = "run"
rm = "remove"
```
{==+==}
```toml
[alias]
b = "build"
c = "check"
d = "doc"
t = "test"
r = "run"
rm = "remove"
```
{==+==}

{==+==}
Aliases are not allowed to redefine existing built-in commands.
{==+==}
别名不允许重新定义现有的内置命令。
{==+==}

{==+==}
Aliases are recursive:
{==+==}
别名可递归:
{==+==}

{==+==}
```toml
[alias]
rr = "run --release"
recursive_example = "rr --example recursions"
```
{==+==}
```toml
[alias]
rr = "run --release"
recursive_example = "rr --example recursions"
```
{==+==}

{==+==}
#### `[build]`

The `[build]` table controls build-time operations and compiler settings.
{==+==}
#### `[build]`

`[build]` 表控制构建时的操作和编译器设置。
{==+==}

{==+==}
##### `build.jobs`
* Type: integer
* Default: number of logical CPUs
* Environment: `CARGO_BUILD_JOBS`
{==+==}
##### `build.jobs`
* Type: 整数
* Default: 逻辑CPU的数量
* Environment: `CARGO_BUILD_JOBS`
{==+==}

{==+==}
Sets the maximum number of compiler processes to run in parallel. If negative,
it sets the maximum number of compiler processes to the number of logical CPUs
plus provided value. Should not be 0.
{==+==}
设定并行运行的最大编译器进程数。如果是负数，它将编译器进程的最大数量设置为逻辑CPU的数量加上所提供的值。不应该是0。
{==+==}

{==+==}
Can be overridden with the `--jobs` CLI option.
{==+==}
可以用 `--jobs` CLI选项覆盖。
{==+==}

{==+==}
##### `build.rustc`
* Type: string (program path)
* Default: "rustc"
* Environment: `CARGO_BUILD_RUSTC` or `RUSTC`
{==+==}
##### `build.rustc`
* Type: 字符串 (程序路径)
* Default: "rustc"
* Environment: `CARGO_BUILD_RUSTC` 或 `RUSTC`
{==+==}

{==+==}
Sets the executable to use for `rustc`.
{==+==}
设置用于 `rustc` 的可执行文件。
{==+==}

{==+==}
##### `build.rustc-wrapper`
* Type: string (program path)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WRAPPER` or `RUSTC_WRAPPER`
{==+==}
##### `build.rustc-wrapper`
* Type: 字符串 (程序路径)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WRAPPER` 或 `RUSTC_WRAPPER`
{==+==}

{==+==}
Sets a wrapper to execute instead of `rustc`. The first argument passed to the
wrapper is the path to the actual executable to use
(i.e., `build.rustc`, if that is set, or `"rustc"` otherwise).
{==+==}
设置包装器来代替 `rustc` 执行。传递给包装器的第一个参数是要使用的实际可执行文件的路径(即 `build.rustc` ，如果它被设置了，或者 `"rustc"` 以其它方式)。
{==+==}

{==+==}
##### `build.rustc-workspace-wrapper`
* Type: string (program path)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WORKSPACE_WRAPPER` or `RUSTC_WORKSPACE_WRAPPER`
{==+==}
##### `build.rustc-workspace-wrapper`
* Type: 字符串 (程序路径)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WORKSPACE_WRAPPER` 或 `RUSTC_WORKSPACE_WRAPPER`
{==+==}

{==+==}
Sets a wrapper to execute instead of `rustc`, for workspace members only.
The first argument passed to the wrapper is the path to the actual
executable to use (i.e., `build.rustc`, if that is set, or `"rustc"` otherwise).
It affects the filename hash so that artifacts produced by the wrapper are cached separately.
{==+==}
设置包装器来代替 `rustc` 执行，仅适用于工作区成员。
传递给包装器的第一个参数是要使用的实际可执行文件的路径(即`build.rustc`，如果它被设置了，或 `"rustc"` 以其它方式)。
它影响文件名hash，以便包装器产生的制品被单独缓存。
{==+==}

{==+==}
##### `build.rustdoc`
* Type: string (program path)
* Default: "rustdoc"
* Environment: `CARGO_BUILD_RUSTDOC` or `RUSTDOC`
{==+==}
##### `build.rustdoc`
* Type: 字符串 (程序路径)
* Default: "rustdoc"
* Environment: `CARGO_BUILD_RUSTDOC` 或 `RUSTDOC`
{==+==}

{==+==}
Sets the executable to use for `rustdoc`.
{==+==}
设置用于 `rustdoc` 的可执行文件。
{==+==}

{==+==}
##### `build.target`
* Type: string or array of strings
* Default: host platform
* Environment: `CARGO_BUILD_TARGET`
{==+==}
##### `build.target`
* Type: 字符串或字符串数组
* Default: 主机平台
* Environment: `CARGO_BUILD_TARGET`
{==+==}

{==+==}
The default target platform triples to compile to.
{==+==}
默认的目标平台三元组编译。
{==+==}

{==+==}
This allows passing either a string or an array of strings. Each string value
is a target platform triple. The selected build targets will be built for each
of the selected architectures.
{==+==}
这允许传递字符串或字符串数组。每个字符串值是目标平台三元组。将为每个选定的架构构建目标。
{==+==}

{==+==}
The string value may also be a relative path to a `.json` target spec file.
{==+==}
该字符串值也可以是 `.json` 目标规格文件的相对路径。
{==+==}

{==+==}
Can be overridden with the `--target` CLI option.
{==+==}
可以用 `--target` CLI选项覆盖。
{==+==}

{==+==}
```toml
[build]
target = ["x86_64-unknown-linux-gnu", "i686-unknown-linux-gnu"]
```
{==+==}
```toml
[build]
target = ["x86_64-unknown-linux-gnu", "i686-unknown-linux-gnu"]
```
{==+==}

{==+==}
##### `build.target-dir`
* Type: string (path)
* Default: "target"
* Environment: `CARGO_BUILD_TARGET_DIR` or `CARGO_TARGET_DIR`
{==+==}
##### `build.target-dir`
* Type: 字符路 (path)
* Default: "target"
* Environment: `CARGO_BUILD_TARGET_DIR` 或 `CARGO_TARGET_DIR`
{==+==}

{==+==}
The path to where all compiler output is placed. The default if not specified
is a directory named `target` located at the root of the workspace.
{==+==}
放置所有编译器输出的路径。如果没有指定，默认是位于工作区根的名为 `target` 的目录。
{==+==}

{==+==}
Can be overridden with the `--target-dir` CLI option.
{==+==}
可以用 `--target-dir` CLI选项覆盖。
{==+==}

{==+==}
##### `build.rustflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_BUILD_RUSTFLAGS` or `CARGO_ENCODED_RUSTFLAGS` or `RUSTFLAGS`
{==+==}
##### `build.rustflags`
* Type: 字符串或字符串数组
* Default: none
* Environment: `CARGO_BUILD_RUSTFLAGS` 或 `CARGO_ENCODED_RUSTFLAGS` 或 `RUSTFLAGS`
{==+==}

{==+==}
Extra command-line flags to pass to `rustc`. The value may be an array of
strings or a space-separated string.
{==+==}
额外的命令行标志，传递给 `rustc` 。该值可以是字符串数组或以空格分隔的字符串。
{==+==}

{==+==}
There are four mutually exclusive sources of extra flags. They are checked in
order, with the first one being used:
{==+==}
有四个互相排斥的额外标志源，依次检查，首先使用第一个:
{==+==}

{==+==}
1. `CARGO_ENCODED_RUSTFLAGS` environment variable.
2. `RUSTFLAGS` environment variable.
3. All matching `target.<triple>.rustflags` and `target.<cfg>.rustflags`
   config entries joined together.
4. `build.rustflags` config value.
{==+==}
1. `CARGO_ENCODED_RUSTFLAGS` 环境变量。
2. `RUSTFLAGS`  环境变量。
3. 所有匹配的 `target.<triple>.rustflags` 和 `target.<cfg>.rustflags` 配置项连接在一起。
4. `build.rustflags` 配置值。
{==+==}

{==+==}
Additional flags may also be passed with the [`cargo rustc`] command.
{==+==}
额外的标志也可以通过 [`cargo rustc`] 命令传递。
{==+==}

{==+==}
If the `--target` flag (or [`build.target`](#buildtarget)) is used, then the
flags will only be passed to the compiler for the target. Things being built
for the host, such as build scripts or proc macros, will not receive the args.
Without `--target`, the flags will be passed to all compiler invocations
(including build scripts and proc macros) because dependencies are shared. If
you have args that you do not want to pass to build scripts or proc macros and
are building for the host, pass `--target` with the host triple.
{==+==}
如果使用 `--target` 标志(或[`build.target`](#buildtarget))，那么这些标志将只传递给目标的编译器。为主机构建的东西，如构建脚本或proc macros，将不会收到这些args。
如果不使用 `--target` ，标志将被传递给所有编译器调用(包括构建脚本和proc macros)，因为依赖是共享的。
如果你有不想传递给构建脚本或proc macros的args，并且是为主机构建的，请将 `--target` 与主机三元组一起传递。
{==+==}

{==+==}
It is not recommended to pass in flags that Cargo itself usually manages. For
example, the flags driven by [profiles](profiles.md) are best handled by setting the
appropriate profile setting.
{==+==}
我们不建议传入Cargo本身通常管理的标志。
例如，由[profiles](profiles.md)决定的标志，最好通过设置适当的配置文件来处理。
{==+==}

{==+==}
> **Caution**: Due to the low-level nature of passing flags directly to the
> compiler, this may cause a conflict with future versions of Cargo which may
> issue the same or similar flags on its own which may interfere with the
> flags you specify. This is an area where Cargo may not always be backwards
> compatible.
{==+==}
> **警告**: 由于直接向编译器传递标志的低阶特性，这可能会与未来版本的Cargo产生冲突，后者可能会自行发布相同或类似的标志，这可能会干扰你指定的标志。
> 这是Cargo可能总是不会向后兼容的领域。
{==+==}

{==+==}
##### `build.rustdocflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_BUILD_RUSTDOCFLAGS` or `CARGO_ENCODED_RUSTDOCFLAGS` or `RUSTDOCFLAGS`
{==+==}
##### `build.rustdocflags`
* Type: 字符串或字符串数组
* Default: none
* Environment: `CARGO_BUILD_RUSTDOCFLAGS` 或 `CARGO_ENCODED_RUSTDOCFLAGS` 或 `RUSTDOCFLAGS`
{==+==}

{==+==}
Extra command-line flags to pass to `rustdoc`. The value may be an array of
strings or a space-separated string.
{==+==}
额外的命令行标志，传递给 `rustdoc` 。该值可以是字符串数组或以空格分隔的字符串。
{==+==}

{==+==}
There are three mutually exclusive sources of extra flags. They are checked in
order, with the first one being used:
{==+==}
有三个相互排斥的额外标志源。依次检查它们，首先使用第一个。
{==+==}

{==+==}
1. `CARGO_ENCODED_RUSTDOCFLAGS` environment variable.
2. `RUSTDOCFLAGS` environment variable.
3. `build.rustdocflags` config value.
{==+==}
1. `CARGO_ENCODED_RUSTDOCFLAGS` 环境变量。
2. `RUSTDOCFLAGS` 环境变量。
3. `build.rustdocflags` 配置值。
{==+==}

{==+==}
Additional flags may also be passed with the [`cargo rustdoc`] command.
{==+==}
附加标志也可以通过 [`cargo rustdoc`] 命令传递。
{==+==}

{==+==}
##### `build.incremental`
* Type: bool
* Default: from profile
* Environment: `CARGO_BUILD_INCREMENTAL` or `CARGO_INCREMENTAL`
{==+==}
##### `build.incremental`
* Type: bool
* Default: 来自配置文件
* Environment: `CARGO_BUILD_INCREMENTAL` 或 `CARGO_INCREMENTAL`
{==+==}

{==+==}
Whether or not to perform [incremental compilation]. The default if not set is
to use the value from the [profile](profiles.md#incremental). Otherwise this overrides the setting of
all profiles.
{==+==}
是否执行[增量编译]。如果没有设置，默认是使用 [profile](profiles.md#incremental) 中的值。否则这将覆盖所有配置文件的设置。
{==+==}

{==+==}
The `CARGO_INCREMENTAL` environment variable can be set to `1` to force enable
incremental compilation for all profiles, or `0` to disable it. This env var
overrides the config setting.
{==+==}
 `CARGO_INCREMENTAL` 环境变量可以设置为 `1` 以强制启用所有配置文件的增量编译，或 `0` 以禁用它。这个环境变量覆盖配置设置。
{==+==}

{==+==}
##### `build.dep-info-basedir`
* Type: string (path)
* Default: none
* Environment: `CARGO_BUILD_DEP_INFO_BASEDIR`
{==+==}

##### `build.dep-info-basedir`
* Type: 字符串 (path)
* Default: none
* Environment: `CARGO_BUILD_DEP_INFO_BASEDIR`
{==+==}

{==+==}
Strips the given path prefix from [dep
info](../guide/build-cache.md#dep-info-files) file paths. This config setting
is intended to convert absolute paths to relative paths for tools that require
relative paths.
{==+==}
从[dep info](../guide/build-cache.md#dep-info-files)文件路径中分割指定的路径前缀。这个配置的目的是为需要相对路径的工具，将绝对路径转换成相对路径。
{==+==}

{==+==}
The setting itself is a config-relative path. So, for example, a value of
`"."` would strip all paths starting with the parent directory of the `.cargo`
directory.
{==+==}
这个设置自身是与配置相对的路径。举例来说，`"."` 的值将分割所有以 `.cargo` 目录的父目录为起点的路径。
{==+==}

{==+==}
##### `build.pipelining`

This option is deprecated and unused. Cargo always has pipelining enabled.
{==+==}
##### `build.pipelining`

这个选项已被废弃，未使用。Cargo总是启用流水线。
{==+==}

{==+==}
#### `[doc]`

The `[doc]` table defines options for the [`cargo doc`] command.
{==+==}
#### `[doc]`

`[doc]` 表定义了 [`cargo doc`] 命令的选项。
{==+==}

{==+==}
##### `doc.browser`
{==+==}
##### `doc.browser`
{==+==}

{==+==}
* Type: string or array of strings ([program path with args])
* Default: `BROWSER` environment variable, or, if that is missing,
  opening the link in a system specific way
{==+==}
* Type: 字符串或字符串数组 ([带参数的程序路径])
* Default: `BROWSER` 环境变量，如果缺少该变量，则以系统特定的方式打开链接
{==+==}

{==+==}
This option sets the browser to be used by [`cargo doc`], overriding the
`BROWSER` environment variable when opening documentation with the `--open`
option.
{==+==}
这个选项设置 [`cargo doc`] 使用的浏览器，在用 `--open` 选项打开文档时，覆盖 `BROWSER` 环境变量。
{==+==}

{==+==}
#### `[cargo-new]`

The `[cargo-new]` table defines defaults for the [`cargo new`] command.
{==+==}
#### `[cargo-new]`

`[cargo-new]` 表定义了 [`cargo new`] 命令的默认值。
{==+==}

{==+==}
##### `cargo-new.name`

This option is deprecated and unused.
{==+==}
##### `cargo-new.name`

该选项已被废弃，未被使用。
{==+==}

{==+==}
##### `cargo-new.email`

This option is deprecated and unused.
{==+==}
##### `cargo-new.email`

该选项已被废弃，未被使用。
{==+==}

{==+==}
##### `cargo-new.vcs`
* Type: string
* Default: "git" or "none"
* Environment: `CARGO_CARGO_NEW_VCS`
{==+==}
##### `cargo-new.vcs`
* Type: 字符串
* Default: "git" 或 "none"
* Environment: `CARGO_CARGO_NEW_VCS`
{==+==}

{==+==}
Specifies the source control system to use for initializing a new repository.
Valid values are `git`, `hg` (for Mercurial), `pijul`, `fossil` or `none` to
disable this behavior. Defaults to `git`, or `none` if already inside a VCS
repository. Can be overridden with the `--vcs` CLI option.
{==+==}
指定初始化新版本库时使用的源码控制系统。
有效值是 `git` 、 `hg` (用于Mercurial) 、 `pijul`  、 `fossil` ，或 `none` 以禁用此行为。默认为 `git` ，如果已经在一个VCS仓库内，则默认为 `none` 。可以用 `--vcs` CLI选项来覆盖。
{==+==}

{==+==}
### `[env]`

The `[env]` section allows you to set additional environment variables for
build scripts, rustc invocations, `cargo run` and `cargo build`.
{==+==}
### `[env]`

`[env]` 部分允许你为构建脚本、rustc调用、 `cargo run` 和 `cargo build` 设置附加的环境变量。
{==+==}

{==+==}
```toml
[env]
OPENSSL_DIR = "/opt/openssl"
```
{==+==}
```toml
[env]
OPENSSL_DIR = "/opt/openssl"
```
{==+==}

{==+==}
By default, the variables specified will not override values that already exist
in the environment. This behavior can be changed by setting the `force` flag.
{==+==}
默认情况下，指定的变量将不会覆盖环境中已经存在的值。可以通过设置 `force` 标志来改变这种行为。
{==+==}

{==+==}
Setting the `relative` flag evaluates the value as a config-relative path that
is relative to the parent directory of the `.cargo` directory that contains the
`config.toml` file. The value of the environment variable will be the full
absolute path.
{==+==}
设置 `relative` 标志会将该值评估为相对配置路径，它是相对于包含`config.toml` 文件的 `.cargo` 目录的父目录而言的。环境变量的值将是完整的绝对路径。
{==+==}

{==+==}
```toml
[env]
TMPDIR = { value = "/home/tmp", force = true }
OPENSSL_DIR = { value = "vendor/openssl", relative = true }
```
{==+==}
```toml
[env]
TMPDIR = { value = "/home/tmp", force = true }
OPENSSL_DIR = { value = "vendor/openssl", relative = true }
```
{==+==}

{==+==}
### `[future-incompat-report]`

The `[future-incompat-report]` table controls setting for [future incompat reporting](future-incompat-report.md)
{==+==}
### `[future-incompat-report]`

`[future-incompat-report]` 表控制 [未来不匹配的报告](future-incompat-report.md) 的设置。
{==+==}

{==+==}
#### `future-incompat-report.frequency`
* Type: string
* Default: "always"
* Environment: `CARGO_FUTURE_INCOMPAT_REPORT_FREQUENCY`
{==+==}
#### `future-incompat-report.frequency`
* Type: 字符串
* Default: "always"
* Environment: `CARGO_FUTURE_INCOMPAT_REPORT_FREQUENCY`
{==+==}

{==+==}
Controls how often we display a notification to the terminal when a future incompat report is available. Possible values:
{==+==}
控制当未来不兼容报告可用时，多长时间向终端显示一次通知。可能的值为:
{==+==}

{==+==}
* `always` (default): Always display a notification when a command (e.g. `cargo build`) produces a future incompat report
* `never`: Never display a notification
{==+==}
* `always` (默认): 当命令(例如 `cargo build`)产生未来不兼容报告时，总是显示通知。
* `never`: 永不显示通知
{==+==}

{==+==}
#### `[http]`

The `[http]` table defines settings for HTTP behavior. This includes fetching
crate dependencies and accessing remote git repositories.
{==+==}
#### `[http]`

`[http]` 表定义了HTTP行为设置。包括获取crate的依赖和访问远程git仓库。
{==+==}

{==+==}
##### `http.debug`
* Type: boolean
* Default: false
* Environment: `CARGO_HTTP_DEBUG`
{==+==}
##### `http.debug`
* Type: boolean
* Default: false
* Environment: `CARGO_HTTP_DEBUG`
{==+==}

{==+==}
If `true`, enables debugging of HTTP requests. The debug information can be
seen by setting the `CARGO_LOG=cargo::ops::registry=debug` environment
variable (or use `trace` for even more information).
{==+==}
如果 `true` ，启用对HTTP请求的调试。调试信息可以通过设置 `CARGO_LOG=cargo::ops::registry=debug` 环境变量查看(或者使用 `trace` 获取更多信息) 。
{==+==}

{==+==}
Be wary when posting logs from this output in a public location. The output
may include headers with authentication tokens which you don't want to leak!
Be sure to review logs before posting them.
{==+==}
当把这个输出的日志发布到公开位置时，要小心谨慎。该输出可能包括带有认证令牌的头信息，你不希望泄漏这些信息。在发布日志之前，一定要审查日志。
{==+==}

{==+==}
##### `http.proxy`
* Type: string
* Default: none
* Environment: `CARGO_HTTP_PROXY` or `HTTPS_PROXY` or `https_proxy` or `http_proxy`
{==+==}
##### `http.proxy`
* Type: 字符串
* Default: none
* Environment: `CARGO_HTTP_PROXY` 或 `HTTPS_PROXY` 或 `https_proxy` 或 `http_proxy`
{==+==}

{==+==}
Sets an HTTP and HTTPS proxy to use. The format is in [libcurl format] as in
`[protocol://]host[:port]`. If not set, Cargo will also check the `http.proxy`
setting in your global git configuration. If none of those are set, the
`HTTPS_PROXY` or `https_proxy` environment variables set the proxy for HTTPS
requests, and `http_proxy` sets it for HTTP requests.
{==+==}
设置一个要使用的HTTP和HTTPS代理。其格式为 [libcurl格式] ，如 `[protocol://]host[:port]` 。
如果没有设置，Cargo也会检查你全局git配置中的 `http.proxy` 设置。
如果这些都没有设置， `HTTPS_PROXY` 或 `https_proxy` 环境变量为HTTPS请求设置代理， `http_proxy` 为HTTP请求设置。
{==+==}

{==+==}
##### `http.timeout`
* Type: integer
* Default: 30
* Environment: `CARGO_HTTP_TIMEOUT` or `HTTP_TIMEOUT`
{==+==}
##### `http.timeout`
* Type: 整数
* Default: 30
* Environment: `CARGO_HTTP_TIMEOUT` 或 `HTTP_TIMEOUT`
{==+==}

{==+==}
Sets the timeout for each HTTP request, in seconds.
{==+==}
设置每个HTTP请求的超时，单位为秒。
{==+==}

{==+==}
##### `http.cainfo`
* Type: string (path)
* Default: none
* Environment: `CARGO_HTTP_CAINFO`
{==+==}
##### `http.cainfo`
* Type: 字符串 (path)
* Default: none
* Environment: `CARGO_HTTP_CAINFO`
{==+==}

{==+==}
Path to a Certificate Authority (CA) bundle file, used to verify TLS
certificates. If not specified, Cargo attempts to use the system certificates.
{==+==}
CA证书绑定文件的路径，用于验证TLS证书。如果没有指定，Cargo会尝试使用系统证书。
{==+==}

{==+==}
##### `http.check-revoke`
* Type: boolean
* Default: true (Windows) false (all others)
* Environment: `CARGO_HTTP_CHECK_REVOKE`
{==+==}
##### `http.check-revoke`
* Type: boolean
* Default: true (Windows) false (all others)
* Environment: `CARGO_HTTP_CHECK_REVOKE`
{==+==}

{==+==}
This determines whether or not TLS certificate revocation checks should be
performed. This only works on Windows.
{==+==}
这决定了是否应该进行TLS证书撤销检查。仅适用于Windows。
{==+==}

{==+==}
##### `http.ssl-version`
* Type: string or min/max table
* Default: none
* Environment: `CARGO_HTTP_SSL_VERSION`
{==+==}
##### `http.ssl-version`
* Type: string 或 min/max table
* Default: none
* Environment: `CARGO_HTTP_SSL_VERSION`
{==+==}

{==+==}
This sets the minimum TLS version to use. It takes a string, with one of the
possible values of "default", "tlsv1", "tlsv1.0", "tlsv1.1", "tlsv1.2", or
"tlsv1.3".
{==+==}
此设置要使用的最小TLS版本。它需要一个字符串，可能的值是 "default"、"tlsv1"、"tlsv1.0"、"tlsv1.1"、"tlsv1.2"、"tlsv1.3" 之一。
{==+==}

{==+==}
This may alternatively take a table with two keys, `min` and `max`, which each
take a string value of the same kind that specifies the minimum and maximum
range of TLS versions to use.
{==+==}
这可能是有两个键的表，`min` 和 `max` ，每个键都有相同的字符串值，指定要使用的TLS版本的最小和最大范围。
{==+==}

{==+==}
The default is a minimum version of "tlsv1.0" and a max of the newest version
supported on your platform, typically "tlsv1.3".
{==+==}
默认情况下，最小版本为 "tlsv1.0" ，最大版本为你的平台上支持的最新版本，通常为 "tlsv1.3" 。
{==+==}

{==+==}
##### `http.low-speed-limit`
* Type: integer
* Default: 10
* Environment: `CARGO_HTTP_LOW_SPEED_LIMIT`
{==+==}
##### `http.low-speed-limit`
* Type: 整数
* Default: 10
* Environment: `CARGO_HTTP_LOW_SPEED_LIMIT`
{==+==}

{==+==}
This setting controls timeout behavior for slow connections. If the average
transfer speed in bytes per second is below the given value for
[`http.timeout`](#httptimeout) seconds (default 30 seconds), then the
connection is considered too slow and Cargo will abort and retry.
{==+==}
此设置控制过慢连接超时行为。
如果以字节/秒为单位的平均传输速度低于[`http.timeout`](#httptimeout)秒的给定值(默认为30秒)，那么该连接被认为太慢，Cargo将中止并重试。
{==+==}

{==+==}
##### `http.multiplexing`
* Type: boolean
* Default: true
* Environment: `CARGO_HTTP_MULTIPLEXING`
{==+==}
##### `http.multiplexing`
* Type: boolean
* Default: true
* Environment: `CARGO_HTTP_MULTIPLEXING`
{==+==}

{==+==}
When `true`, Cargo will attempt to use the HTTP2 protocol with multiplexing.
This allows multiple requests to use the same connection, usually improving
performance when fetching multiple files. If `false`, Cargo will use HTTP 1.1
without pipelining.
{==+==}
当 `true` 时，Cargo将尝试使用多路复用的HTTP2协议。
这允许多个请求使用同一个连接，通常可以提高获取多文件的性能。
如果 `false` ，Cargo将使用HTTP1.1，没有管道。
{==+==}

{==+==}
##### `http.user-agent`
* Type: string
* Default: Cargo's version
* Environment: `CARGO_HTTP_USER_AGENT`
{==+==}
##### `http.user-agent`
* Type: 字符串
* Default: Cargo的版本
* Environment: `CARGO_HTTP_USER_AGENT`
{==+==}

{==+==}
Specifies a custom user-agent header to use. The default if not specified is a
string that includes Cargo's version.
{==+==}
指定要使用的自定义user-agent header。如果没有指定，默认是一个包括Cargo版本的字符串。
{==+==}

{==+==}
#### `[install]`

The `[install]` table defines defaults for the [`cargo install`] command.
{==+==}
#### `[install]`

`[install]` 表定义了 [`cargo install`] 命令的默认值。
{==+==}

{==+==}
##### `install.root`
* Type: string (path)
* Default: Cargo's home directory
* Environment: `CARGO_INSTALL_ROOT`
{==+==}
##### `install.root`
* Type: 字符串 (path)
* Default: Cargo的主目录
* Environment: `CARGO_INSTALL_ROOT`
{==+==}

{==+==}
Sets the path to the root directory for installing executables for [`cargo
install`]. Executables go into a `bin` directory underneath the root.
{==+==}
为 [`cargo install`] 设置安装可执行文件的根目录的路径。可执行文件会进入根目录下的 `bin` 目录。
{==+==}

{==+==}
To track information of installed executables, some extra files, such as
`.crates.toml` and `.crates2.json`, are also created under this root.
{==+==}
为了跟踪已安装的可执行文件的信息，一些附加文件，如 `.crates.toml` 和 `.crates2.json` 也会在这个根下创建。
{==+==}

{==+==}
The default if not specified is Cargo's home directory (default `.cargo` in
your home directory).
{==+==}
如果没有指定，默认是Cargo的主目录 (默认为你的主目录中 `.cargo` ) 。
{==+==}

{==+==}
Can be overridden with the `--root` command-line option.
{==+==}
可以用 `--root` 命令行选项来覆盖。
{==+==}

{==+==}
#### `[net]`

The `[net]` table controls networking configuration.
{==+==}
#### `[net]`

`[net]` 表控制网络配置。
{==+==}

{==+==}
##### `net.retry`
* Type: integer
* Default: 2
* Environment: `CARGO_NET_RETRY`
{==+==}
##### `net.retry`
* Type: 整数
* Default: 2
* Environment: `CARGO_NET_RETRY`
{==+==}

{==+==}
Number of times to retry possibly spurious network errors.
{==+==}
可能是虚假网络的错误重试次数。
{==+==}

{==+==}
##### `net.git-fetch-with-cli`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_GIT_FETCH_WITH_CLI`
{==+==}
##### `net.git-fetch-with-cli`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_GIT_FETCH_WITH_CLI`
{==+==}

{==+==}
If this is `true`, then Cargo will use the `git` executable to fetch registry
indexes and git dependencies. If `false`, then it uses a built-in `git`
library.
{==+==}
如果这是 `true` ，那么Cargo将使用 `git` 可执行程序来获取注册中心索引和git依赖。如果是 `false` ，那么它将使用内置的 `git` 库。
{==+==}

{==+==}
Setting this to `true` can be helpful if you have special authentication
requirements that Cargo does not support. See [Git
Authentication](../appendix/git-authentication.md) for more information about
setting up git authentication.
{==+==}
如果你有特殊的认证要求，而Cargo不支持的话，将此设置为 `true` 会有帮助。
参阅 [Git Authentication](../appendix/git-authentication.md)以了解更多关于设置git认证的信息。
{==+==}

{==+==}
##### `net.offline`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_OFFLINE`
{==+==}
##### `net.offline`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_OFFLINE`
{==+==}

{==+==}
If this is `true`, then Cargo will avoid accessing the network, and attempt to
proceed with locally cached data. If `false`, Cargo will access the network as
needed, and generate an error if it encounters a network error.
{==+==}
如果是 `true` ，那么Cargo将避免访问网络，并尝试使用本地缓存的数据。
如果是 `false` ，Cargo将根据需要访问网络，如果遇到网络错误，将产生一个错误。
{==+==}

{==+==}
Can be overridden with the `--offline` command-line option.
{==+==}
可以用 `--offline` 命令行选项来覆盖。
{==+==}

{==+==}
#### `[patch]`
{==+==}
#### `[patch]`
{==+==}

{==+==}
Just as you can override dependencies using [`[patch]` in
`Cargo.toml`](overriding-dependencies.md#the-patch-section), you can
override them in the cargo configuration file to apply those patches to
any affected build. The format is identical to the one used in
`Cargo.toml`.
{==+==}
就像你可以用[`[patch]` in `Cargo.toml`](overriding-dependencies.md#the-patch-section)覆盖依赖一样，
你可以在cargo配置文件中覆盖它们，将这些补丁应用于任何受影响的构建。其格式与 `Cargo.toml` 中的格式相同。
{==+==}

{==+==}
Since `.cargo/config.toml` files are not usually checked into source
control, you should prefer patching using `Cargo.toml` where possible to
ensure that other developers can compile your crate in their own
environments. Patching through cargo configuration files is generally
only appropriate when the patch section is automatically generated by an
external build tool.
{==+==}
由于 `.cargo/config.toml` 文件通常不会被检查到源代码控制中，你应该尽可能选择使用 `Cargo.toml` 来打补丁，以确保其他开发者可以在自己的环境中编译你的crate。
一般来说，只有当补丁部分由外部构建工具自动生成时，通过cargo配置文件进行修补才是合适的。
{==+==}

{==+==}
If a given dependency is patched both in a cargo configuration file and
a `Cargo.toml` file, the patch in the configuration file is used. If
multiple configuration files patch the same dependency, standard cargo
configuration merging is used, which prefers the value defined closest
to the current directory, with `$HOME/.cargo/config.toml` taking the
lowest precedence.
{==+==}
如果特定的依赖在cargo配置文件和 `Cargo.toml` 文件中都有补丁，则使用配置文件中的补丁。
如果多个配置文件对同一个依赖打了补丁，则使用标准的cargo配置合并，它更倾向于使用离当前目录最近的定义值。
`$HOME/.cargo/config.toml` 的优先级最低。
{==+==}

{==+==}
Relative `path` dependencies in such a `[patch]` section are resolved
relative to the configuration file they appear in.
{==+==}
在这样的 `[patch]` 部分中的相对 `path` 依赖是相对于它们出现配置文件的。
{==+==}

{==+==}
#### `[profile]`

The `[profile]` table can be used to globally change profile settings, and
override settings specified in `Cargo.toml`. It has the same syntax and
options as profiles specified in `Cargo.toml`. See the [Profiles chapter] for
details about the options.
{==+==}
#### `[profile]`

`[profile]` 表可以用来全局改变配置文件的设置，并覆盖 `Cargo.toml` 中指定的设置。
它的语法和选项与 `Cargo.toml` 中指定的配置文件相同。
关于选项的详细信息，请参阅 [配置文件章节] 。
{==+==}

{==+==}
[Profiles chapter]: profiles.md

##### `[profile.<name>.build-override]`
* Environment: `CARGO_PROFILE_<name>_BUILD_OVERRIDE_<key>`
{==+==}
[Profiles chapter]: profiles.md

##### `[profile.<name>.build-override]`
* Environment: `CARGO_PROFILE_<name>_BUILD_OVERRIDE_<key>`
{==+==}

{==+==}
The build-override table overrides settings for build scripts, proc macros,
and their dependencies. It has the same keys as a normal profile. See the
[overrides section](profiles.md#overrides) for more details.
{==+==}
构建覆盖表覆盖构建脚本、进程宏和其依赖的设置。
它的键值与普通配置文件相同。
参见 [覆盖部分](profiles.md#overrides) 以了解更多细节。
{==+==}

{==+==}
##### `[profile.<name>.package.<name>]`
* Environment: not supported
{==+==}

##### `[profile.<name>.package.<name>]`
* Environment: 不支持
{==+==}

{==+==}
The package table overrides settings for specific packages. It has the same
keys as a normal profile, minus the `panic`, `lto`, and `rpath` settings. See
the [overrides section](profiles.md#overrides) for more details.
{==+==}
包表覆盖特定包的设置。
它和普通的配置文件有相同的键，除 `panic` 、 `lto` 和 `rpath` 设置。
参阅 [overrides section](profiles.md#overrides) 以了解更多细节。
{==+==}

{==+==}
##### `profile.<name>.codegen-units`
* Type: integer
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_CODEGEN_UNITS`
{==+==}
##### `profile.<name>.codegen-units`
* Type: 整数
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_CODEGEN_UNITS`
{==+==}

{==+==}
See [codegen-units](profiles.md#codegen-units).
{==+==}
参阅 [codegen-units](profiles.md#codegen-units) 。
{==+==}

{==+==}
##### `profile.<name>.debug`
* Type: integer or boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_DEBUG`
{==+==}
##### `profile.<name>.debug`
* Type: 整数或布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_DEBUG`
{==+==}

{==+==}
See [debug](profiles.md#debug).
{==+==}
参阅 [debug](profiles.md#debug)。
{==+==}

{==+==}
##### `profile.<name>.split-debuginfo`
* Type: string
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_SPLIT_DEBUGINFO`
{==+==}
##### `profile.<name>.split-debuginfo`
* Type: 字符串
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_SPLIT_DEBUGINFO`
{==+==}

{==+==}
See [split-debuginfo](profiles.md#split-debuginfo).
{==+==}
参阅 [split-debuginfo](profiles.md#split-debuginfo)。
{==+==}

{==+==}
##### `profile.<name>.debug-assertions`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_DEBUG_ASSERTIONS`
{==+==}
##### `profile.<name>.debug-assertions`
* Type: 布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_DEBUG_ASSERTIONS`
{==+==}

{==+==}
See [debug-assertions](profiles.md#debug-assertions).
{==+==}
参阅 [debug-assertions](profiles.md#debug-assertions)。
{==+==}

{==+==}
##### `profile.<name>.incremental`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_INCREMENTAL`
{==+==}
##### `profile.<name>.incremental`
* Type: 布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_INCREMENTAL`
{==+==}

{==+==}
See [incremental](profiles.md#incremental).
{==+==}
参阅 [incremental](profiles.md#incremental).
{==+==}

{==+==}
##### `profile.<name>.lto`
* Type: string or boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_LTO`
{==+==}
##### `profile.<name>.lto`
* Type: 字符串或布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_LTO`
{==+==}

{==+==}
See [lto](profiles.md#lto).
{==+==}
参阅 [lto](profiles.md#lto) 。
{==+==}

{==+==}
##### `profile.<name>.overflow-checks`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_OVERFLOW_CHECKS`
{==+==}
##### `profile.<name>.overflow-checks`
* Type: 布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_OVERFLOW_CHECKS`
{==+==}

{==+==}
See [overflow-checks](profiles.md#overflow-checks).
{==+==}
参阅 [overflow-checks](profiles.md#overflow-checks) 。
{==+==}

{==+==}
##### `profile.<name>.opt-level`
* Type: integer or string
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_OPT_LEVEL`
{==+==}
##### `profile.<name>.opt-level`
* Type: 字符串或布尔
* Default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_OPT_LEVEL`
{==+==}

{==+==}
See [opt-level](profiles.md#opt-level).
{==+==}
参阅 [opt-level](profiles.md#opt-level) 。
{==+==}

{==+==}
##### `profile.<name>.panic`
* Type: string
* default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_PANIC`
{==+==}
##### `profile.<name>.panic`
* Type: 字符串
* default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_PANIC`
{==+==}

{==+==}
See [panic](profiles.md#panic).
{==+==}
参阅 [panic](profiles.md#panic) 。
{==+==}

{==+==}
##### `profile.<name>.rpath`
* Type: boolean
* default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_RPATH`
{==+==}
##### `profile.<name>.rpath`
* Type: 布尔
* default: 参阅配置文件文档。
* Environment: `CARGO_PROFILE_<name>_RPATH`
{==+==}

{==+==}
See [rpath](profiles.md#rpath).
{==+==}
参阅 [rpath](profiles.md#rpath) 。
{==+==}

{==+==}
#### `[registries]`

The `[registries]` table is used for specifying additional [registries]. It
consists of a sub-table for each named registry.
{==+==}
#### `[registries]`

`[注册表]` 表用于指定附加的[注册中心]。
它由各个命名的注册中心的子表组成。
{==+==}

{==+==}
##### `registries.<name>.index`
* Type: string (url)
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_INDEX`
{==+==}
##### `registries.<name>.index`
* Type: 字符串 (url)
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_INDEX`
{==+==}

{==+==}
Specifies the URL of the git index for the registry.
{==+==}
指定注册中心git索引的URL。
{==+==}

{==+==}
##### `registries.<name>.token`
* Type: string
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_TOKEN`
{==+==}
##### `registries.<name>.token`
* Type: 字符串
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_TOKEN`
{==+==}

{==+==}
Specifies the authentication token for the given registry. This value should
only appear in the [credentials](#credentials) file. This is used for registry
commands like [`cargo publish`] that require authentication.
{==+==}
指定给定注册中心的认证令牌。
这个值应该只出现在 [credentials](#credentials) 文件中。
这用于需要认证的注册中心命令，如 [`cargo publish`] 。
{==+==}

{==+==}
Can be overridden with the `--token` command-line option.
{==+==}
可以用 `--token` 命令行选项来覆盖。
{==+==}

{==+==}
#### `[registry]`

The `[registry]` table controls the default registry used when one is not
specified.
{==+==}
#### `[registry]`

`[注册中心]` 表控制在没有指定的情况下使用的默认注册中心。
{==+==}

{==+==}
##### `registry.index`

This value is no longer accepted and should not be used.
{==+==}
##### `registry.index`

不再接受此值，不应使用。
{==+==}

{==+==}
##### `registry.default`
* Type: string
* Default: `"crates-io"`
* Environment: `CARGO_REGISTRY_DEFAULT`
{==+==}
##### `registry.default`
* Type: 字符串
* Default: `"crates-io"`
* Environment: `CARGO_REGISTRY_DEFAULT`
{==+==}

{==+==}
The name of the registry (from the [`registries` table](#registries)) to use
by default for registry commands like [`cargo publish`].
{==+==}
注册中心的名称(来自[ `registries` 表](#registries))，默认用于 [`cargo publish`] 等注册中心命令。
{==+==}

{==+==}
Can be overridden with the `--registry` command-line option.
{==+==}
可以用 `--registry` 命令行选项来覆盖。
{==+==}

{==+==}
##### `registry.token`
* Type: string
* Default: none
* Environment: `CARGO_REGISTRY_TOKEN`
{==+==}
##### `registry.token`
* Type: 字符串
* Default: none
* Environment: `CARGO_REGISTRY_TOKEN`
{==+==}

{==+==}
Specifies the authentication token for [crates.io]. This value should only
appear in the [credentials](#credentials) file. This is used for registry
commands like [`cargo publish`] that require authentication.
{==+==}
指定 [crates.io] 的认证令牌。
这个值应该只出现在 [credentials](#credentials) 文件中。
这用于像 [`cargo publish`] 这样的需要认证的注册中心命令。
{==+==}

{==+==}
Can be overridden with the `--token` command-line option.
{==+==}
可以用 `--token` 命令行选项来覆盖。
{==+==}

{==+==}
#### `[source]`

The `[source]` table defines the registry sources available. See [Source
Replacement] for more information. It consists of a sub-table for each named
source. A source should only define one kind (directory, registry,
local-registry, or git).
{==+==}
#### `[source]`

`[source]` 表定义了可用的注册中心源。
更多信息见[源替换]。
它由每个命名的源的一个子表组成。
源应该只定义为一种(目录、注册中心、本地注册或git)。
{==+==}

{==+==}
##### `source.<name>.replace-with`
* Type: string
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.replace-with`
* Type: 字符串
* Default: none
* Environment: 不支持
{==+==}

{==+==}
如果设置，用给定的命名源或命名注册中心替换这个源。
{==+==}

{==+==}

{==+==}
##### `source.<name>.directory`
* Type: string (path)
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.directory`
* Type: 字符串 (path)
* Default: none
* Environment: 不支持
{==+==}

{==+==}
设置目录路径，作为目录源使用。
{==+==}

{==+==}

{==+==}
##### `source.<name>.registry`
* Type: string (url)
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.registry`
* Type: 字符串 (url)
* Default: none
* Environment: 不支持
{==+==}

{==+==}
Sets the URL to use for a registry source.
{==+==}
设置用于注册中心源的URL。
{==+==}

{==+==}
##### `source.<name>.local-registry`
* Type: string (path)
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.local-registry`
* Type: 字符串 (path)
* Default: none
* Environment: 不支持
{==+==}

{==+==}
Sets the path to a directory to use as a local registry source.
{==+==}
设置目录路径，用作本地注册源。
{==+==}

{==+==}
##### `source.<name>.git`
* Type: string (url)
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.git`
* Type: 字符串 (url)
* Default: none
* Environment: 不支持
{==+==}

{==+==}
Sets the URL to use for a git repository source.
{==+==}
设置用于git仓库源的URL。
{==+==}

{==+==}
##### `source.<name>.branch`
* Type: string
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.branch`
* Type: 字符串
* Default: none
* Environment: 不支持
{==+==}

{==+==}
Sets the branch name to use for a git repository.
{==+==}
设置用于git仓库的分支名称。
{==+==}

{==+==}
##### `source.<name>.tag`
* Type: string
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.tag`
* Type: 字符串
* Default: none
* Environment: 不支持
{==+==}
{==+==}

{==+==}
Sets the tag name to use for a git repository.
{==+==}
设置用于git仓库的tag名称。
{==+==}

{==+==}
If none of `branch`, `tag`, or `rev` is set, defaults to the `master` branch.
{==+==}
如果没有设置 `branch` 、 `tag` 或 `rev` ，则默认为 `master` 分支。
{==+==}

{==+==}
##### `source.<name>.rev`
* Type: string
* Default: none
* Environment: not supported
{==+==}
##### `source.<name>.rev`
* Type: 字符串
* Default: none
* Environment: 不支持
{==+==}

{==+==}
Sets the [revision] to use for a git repository.
{==+==}
设置用于git仓库的[修订]。
{==+==}

{==+==}
#### `[target]`

The `[target]` table is used for specifying settings for specific platform
targets. It consists of a sub-table which is either a platform triple or a
[`cfg()` expression]. The given values will be used if the target platform
matches either the `<triple>` value or the `<cfg>` expression.
{==+==}
#### `[target]`

`[target]` 表用于指定特定平台目标的设置。
它由子表组成，这个子表是平台三元组或 [`cfg()` 表达式] 。
如果目标平台与 `<triple>` 值或 `<cfg>` 表达式匹配，将使用给定的值。
{==+==}

{==+==}
```toml
[target.thumbv7m-none-eabi]
linker = "arm-none-eabi-gcc"
runner = "my-emulator"
rustflags = ["…", "…"]

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "my-arm-wrapper"
rustflags = ["…", "…"]
```
{==+==}
```toml
[target.thumbv7m-none-eabi]
linker = "arm-none-eabi-gcc"
runner = "my-emulator"
rustflags = ["…", "…"]

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "my-arm-wrapper"
rustflags = ["…", "…"]
```
{==+==}

{==+==}
`cfg` values come from those built-in to the compiler (run `rustc --print=cfg`
to view), values set by [build scripts], and extra `--cfg` flags passed to
`rustc` (such as those defined in `RUSTFLAGS`). Do not try to match on
`debug_assertions` or Cargo features like `feature="foo"`.
{==+==}
`cfg` 值来自编译器内置的值 (运行 `rustc --print=cfg` 查看)，由[构建脚本]设置的值，以及传递给 `rustc` 的附加 `--cfg` 标志(例如在 `RUSTFLAGS` 中定义的标志)。
不要在 `debug_assertions` 或 Cargo 特性(如 `feature="foo"` )上尝试match。
{==+==}

{==+==}
If using a target spec JSON file, the `<triple>` value is the filename stem.
For example `--target foo/bar.json` would match `[target.bar]`.
{==+==}
如果使用目标规格的JSON文件，`<triple>` 值是文件名。
例如， `--target foo/bar.json` 将匹配 `[target.bar]` 。
{==+==}

{==+==}
##### `target.<triple>.ar`

This option is deprecated and unused.
{==+==}
##### `target.<triple>.ar`

该选项已被废弃，未被使用。
{==+==}

{==+==}
##### `target.<triple>.linker`
* Type: string (program path)
* Default: none
* Environment: `CARGO_TARGET_<triple>_LINKER`
{==+==}
##### `target.<triple>.linker`
* Type: 字符串 (程序路径)
* Default: none
* Environment: `CARGO_TARGET_<triple>_LINKER`
{==+==}

{==+==}
Specifies the linker which is passed to `rustc` (via [`-C linker`]) when the
`<triple>` is being compiled for. By default, the linker is not overridden.
{==+==}
指定在编译 `<triple>` 时传递给 `rustc` (通过[`-C linker`]) 的链接器。
默认情况下，链接器不可覆盖。
{==+==}

{==+==}
##### `target.<triple>.runner`
* Type: string or array of strings ([program path with args])
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUNNER`
{==+==}
##### `target.<triple>.runner`
* Type: 字符串或者字符串数组 ([带参数的程序路径])
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUNNER`
{==+==}

{==+==}
If a runner is provided, executables for the target `<triple>` will be
executed by invoking the specified runner with the actual executable passed as
an argument. This applies to [`cargo run`], [`cargo test`] and [`cargo bench`]
commands. By default, compiled executables are executed directly.
{==+==}
如果提供了运行器，目标 `<triple>` 的可执行文件将通过调用指定的运行器和作为参数传递的实际可执行文件来执行。
这适用于 [`cargo run`], [`cargo test`] 和 [`cargo bench`] 命令。
默认情况下，编译后的可执行文件被直接执行。
{==+==}

{==+==}
##### `target.<cfg>.runner`
{==+==}
##### `target.<cfg>.runner`
{==+==}

{==+==}
This is similar to the [target runner](#targettriplerunner), but using
a [`cfg()` expression]. If both a `<triple>` and `<cfg>` runner match,
the `<triple>` will take precedence. It is an error if more than one
`<cfg>` runner matches the current target.
{==+==}
这类似于 [target runner](#targettriplerunner) ，但使用 [`cfg()` 表达式] 。
如果同时有 `<triple>` 和 `<cfg>` 运行器匹配， `<triple>` 将优先使用。
如果有一个以上的 `<cfg>` 运行器与当前目标匹配，则错误。
{==+==}

{==+==}
##### `target.<triple>.rustflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUSTFLAGS`
{==+==}
##### `target.<triple>.rustflags`
* Type: 字符串或字符串数组
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUSTFLAGS`
{==+==}

{==+==}
Passes a set of custom flags to the compiler for this `<triple>`. The value
may be an array of strings or a space-separated string.
{==+==}
为这个 `<triple>` 向编译器传递一组自定义标志。
该值可以是字符串数组或以空格分隔的字符串。
{==+==}

{==+==}
See [`build.rustflags`](#buildrustflags) for more details on the different
ways to specific extra flags.
{==+==}
参阅 [`build.rustflags`](#buildrustflags) ，以了解更多关于特定附加标志的不同方式的细节。
{==+==}

{==+==}
##### `target.<cfg>.rustflags`
{==+==}
##### `target.<cfg>.rustflags`
{==+==}

{==+==}
This is similar to the [target rustflags](#targettriplerustflags), but
using a [`cfg()` expression]. If several `<cfg>` and `<triple>` entries
match the current target, the flags are joined together.
{==+==}
这类似于 [target rustflags](#targettriplerustflags) ，但使用 [`cfg()` 表达式] 。
如果有几个 `<cfg>` 和 `<triple>` 条目与当前的目标相匹配，这些标志就会被连在一起。
{==+==}

{==+==}
##### `target.<triple>.<links>`
{==+==}
##### `target.<triple>.<links>`
{==+==}

{==+==}
The links sub-table provides a way to [override a build script]. When
specified, the build script for the given `links` library will not be
run, and the given values will be used instead.
{==+==}
链接子表提供了一种[覆盖构建脚本]的方法。
当指定时，给定的 `links` 库的构建脚本将不会运行，而会使用给定的值。
{==+==}

{==+==}
```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
```
{==+==}
```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
```
{==+==}

{==+==}
#### `[term]`

The `[term]` table controls terminal output and interaction.
{==+==}
#### `[term]`

`[term]` 表控制终端输出和交互。
{==+==}

{==+==}
##### `term.quiet`
* Type: boolean
* Default: false
* Environment: `CARGO_TERM_QUIET`
{==+==}
##### `term.quiet`
* Type: 布尔
* Default: false
* Environment: `CARGO_TERM_QUIET`
{==+==}

{==+==}
控制Cargo是否显示日志信息。
{==+==}

{==+==}

{==+==}
Specifying the `--quiet` flag will override and force quiet output.
Specifying the `--verbose` flag will override and disable quiet output.
{==+==}
指定 `--quiet` 标志将覆盖和强制静默输出。
指定 `--verbose` 标志将覆盖和禁用静默输出。
{==+==}

{==+==}
##### `term.verbose`
* Type: boolean
* Default: false
* Environment: `CARGO_TERM_VERBOSE`
{==+==}
##### `term.verbose`
* Type: 布尔
* Default: false
* Environment: `CARGO_TERM_VERBOSE`
{==+==}

{==+==}
Controls whether or not extra detailed messages are displayed by Cargo.
{==+==}
控制Cargo是否显示附加的详细信息。
{==+==}

{==+==}
Specifying the `--quiet` flag will override and disable verbose output.
Specifying the `--verbose` flag will override and force verbose output.
{==+==}
指定 `--quiet` 标志将覆盖并禁用粗略输出。
指定 `--verbose` 标志将覆盖并强制进行粗略输出。
{==+==}

{==+==}
##### `term.color`
* Type: string
* Default: "auto"
* Environment: `CARGO_TERM_COLOR`
{==+==}
##### `term.color`
* Type: 字符串
* Default: "auto"
* Environment: `CARGO_TERM_COLOR`
{==+==}

{==+==}
Controls whether or not colored output is used in the terminal. Possible values:
{==+==}
控制是否在终端中使用彩色输出。可能的值:
{==+==}

{==+==}
* `auto` (default): Automatically detect if color support is available on the
  terminal.
* `always`: Always display colors.
* `never`: Never display colors.
{==+==}
* `auto` (默认): 自动检测终端是否有颜色支持。
* `always`: 允许显示颜色。
* `never`: 不允许显示颜色。
{==+==}

{==+==}
Can be overridden with the `--color` command-line option.
{==+==}
可以用 `--color` 命令行选项来覆盖。
{==+==}

{==+==}
##### `term.progress.when`
* Type: string
* Default: "auto"
* Environment: `CARGO_TERM_PROGRESS_WHEN`
{==+==}
##### `term.progress.when`
* Type: 字符串
* Default: "auto"
* Environment: `CARGO_TERM_PROGRESS_WHEN`
{==+==}

{==+==}
Controls whether or not progress bar is shown in the terminal. Possible values:
{==+==}
控制是否在终端中显示进度条。可能的值:
{==+==}

{==+==}
* `auto` (default): Intelligently guess whether to show progress bar.
* `always`: Always show progress bar.
* `never`: Never show progress bar.
{==+==}
* `auto` (默认): 智能地猜测是否显示进度条。
* `always`: 允许显示进度条。
* `never`: 不允许显示进度条。
{==+==}

{==+==}
##### `term.progress.width`
* Type: integer
* Default: none
* Environment: `CARGO_TERM_PROGRESS_WIDTH`
{==+==}
##### `term.progress.width`
* Type: 整数
* Default: none
* Environment: `CARGO_TERM_PROGRESS_WIDTH`
{==+==}

{==+==}
Sets the width for progress bar.
{==+==}
设置进度条的宽度。
{==+==}

{==+==}
[`cargo bench`]: ../commands/cargo-bench.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo new`]: ../commands/cargo-new.md
[`cargo publish`]: ../commands/cargo-publish.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo rustc`]: ../commands/cargo-rustc.md
[`cargo test`]: ../commands/cargo-test.md
[`cargo rustdoc`]: ../commands/cargo-rustdoc.md
[`cargo install`]: ../commands/cargo-install.md
[env]: environment-variables.md
[`cfg()` expression]: ../../reference/conditional-compilation.html
[build scripts]: build-scripts.md
[`-C linker`]: ../../rustc/codegen-options/index.md#linker
[override a build script]: build-scripts.md#overriding-build-scripts
[toml]: https://toml.io/
[incremental compilation]: profiles.md#incremental
[program path with args]: #executable-paths-with-arguments
[libcurl format]: https://everything.curl.dev/libcurl/proxies#proxy-types
[source replacement]: source-replacement.md
[revision]: https://git-scm.com/docs/gitrevisions
[registries]: registries.md
[crates.io]: https://crates.io/
{==+==}
[`cargo bench`]: ../commands/cargo-bench.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo new`]: ../commands/cargo-new.md
[`cargo publish`]: ../commands/cargo-publish.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo rustc`]: ../commands/cargo-rustc.md
[`cargo test`]: ../commands/cargo-test.md
[`cargo rustdoc`]: ../commands/cargo-rustdoc.md
[`cargo install`]: ../commands/cargo-install.md
[env]: environment-variables.md
[`cfg()` 表达式]: ../../reference/conditional-compilation.html
[build scripts]: build-scripts.md
[`-C linker`]: ../../rustc/codegen-options/index.md#linker
[override a build script]: build-scripts.md#overriding-build-scripts
[toml]: https://toml.io/
[incremental compilation]: profiles.md#incremental
[带参数的程序路径]: #executable-paths-with-arguments
[libcurl format]: https://everything.curl.dev/libcurl/proxies#proxy-types
[源替换]: source-replacement.md
[修订]: https://git-scm.com/docs/gitrevisions
[registries]: registries.md
[crates.io]: https://crates.io/
{==+==}