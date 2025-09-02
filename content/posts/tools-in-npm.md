+++
date = '2025-09-02T15:15:20-04:00'
draft = false
title = 'Tools in NPM and Other Fancies'
+++

There are a few command-line tools that I use that are implemented as `npm` packages.

I don't want to install them globally (that is, to need privileges when
doing so) -- I like to affect as little of the machine as possible: it
could be a shared machine, and even if it's not, backing out changes
or starting again from scratch is a lot easier, and I get a bit of
vertigo anytime I find myself typing `sudo rm -rf ...`.

It all started off with
[**bash-language-server**](https://www.npmjs.com/package/bash-language-server),
since I needed to maintain a Swiss-army knife shell script for
developers. Given what I've seen in my past lives, I suppose some of
you out there have probably also seen or worked on tools written in `bash` that
just kept growing and developed a life of their own.

The Language Server Protocol is a big win, and it's particularly
suited to niche uses like shell scripting languages that would get
their own IDE or IDE mode, I suspect, do not to need maintain a lot of
state in the server. Note that apparently, `bash-language-server`
_could_ have been implemented in a [language other than
JavaScript](https://tree-sitter.github.io/tree-sitter/) -- it's just
that these days, JavaScript is firmly in the mainstream if you
consider its acceptance status as a language with which one is
expected to have to some proficiency, and the reach of `npm` as a way
to distribute packages.

In turn, I hook up [`bash-language-server` with
Emacs](https://www.npmjs.com/package/bash-language-server#emacs) so
that I can tame script complexity in the One True Text Editor, which
has excellent LSP support. It also turns that `bash-language-server`
is an excellent helper to maintain your hairy custom `bash` environment,
which is a nice bit of positive feedback.

Because this was the first `npm`-based tool I had ever used, all I did
was figure out how to install `bash-language-server` manually, and
didn't address that in my usual _modus operandi_, which is to commit
all but the most trival environmental setup to source control. The one
wrinkle, again, was that I didn't want to install packages locally, so
I had to do a little spelunking to find where `npm` was dropping
executables that should go on the `PATH.`

I've been putting my environment in source control since back when CVS
was the cool source control system. [^1] Really, once you start
building up a collection of aliases, scripts, and editor init files,
it's madness to maintain your environment by copying files around. And
along with the multi-machine problem comes the multi-platform
problem. Things still don't work the same everywhere, especially when
it comes to Unix commands that were re-implemented across Linux and
BSD (from which macOS inherits a lot of its userland, even in
2025). So what at first is a mere bag of files becomes actual _code_
to tweak and test (scripts with `if`!) and that means it starts taking
on the characteristics of software, which means You Need Source
Control.

There's often a judgement call about maintaining one's workflow over
time -- when do you bother writing tools and so on? For myself, I
don't like doing similar things twice.

Then the need to run [JFrog's `jf` command](https://jfrog.com/getcli/)
came along. Like `bash-language-server`, this is a command that is
distributed as a Node package. Now it was time to understand the very,
very, _very_ basic rudiments of Node packages (which I had already
seen as a bystander in my role as a build engineer).

# Approach

Simply put, I maintain a `package.json` that describes the tools I need.

For my approach, I also need to define an environment variable that
captures the directory where this `package.json` file is checked
in. This variable is used by the commands to maintain the package and
add it the `PATH`. Setting that should your shell profile -- your
favorite shell's profile (like `.bash_profile`) file or something it
includes (likely by `source`).

For example:

```bash
export _ME_NPM_PROJECT=~/Library/init/tools-in-npm
```

The `package.json` there looks like this for using `bash-language-server`
and `jf`:

```json
{
  "dependencies": {
      "bash-language-server": ">5.0.0",
      "jfrog-cli-v2-jf": ">2.0"
  }
}
```

Each tool is a dependency. The versions mentioned here were just what
was current when I started using these tools.

We Git-ignore the files that `npm` writes -- the only "truth" is in
the `package.json` file -- a `.gitignore` in `$_ME_NPM_PROJECT` should
do the trick:

```
# Ignore npm files
node_modules
package-lock.json
```

So, the only two files tracked in `$_ME_NPM_PROJECT` are `.gitignore`
and `package.json`.

Now, time to get back to getting those `npm` commands on your path
from your shell's profile. I have a bunch of different ways to get
`npm` itself depending on the operating system and whether it's
been installed by certain add-on package managers (such as MacPorts),
and end up with the `npm` to use in the variable `node_thing`

```bash
if [ -n "${node_thing}" ] ;then
    # Assume we need this for locally installed node
    # packages.
    if [ -e "${_ME_NPM_PROJECT}/node_modules/.bin" ] ; then
       PATH="${_ME_NPM_PROJECT}/node_modules/.bin:$PATH"
    fi
fi
```

(You could just set `node_thing` to `npm` and hope for the best. It's
only used to find `npm` and I make sure that `npm` is on the
`PATH`.)

Because I don't put this `.bin` directory on the `PATH` unless it exists,
the first time I'm in a new "world," I have to start a new top-level
shell after running the install command.

Installation and update are two separate commands which I define as
shell functions, so they should be defined in a per-shell file
such as `.bashrc`.

```bash
_me_npm_update() {
    (cd "${_ME_NPM_PROJECT}" && npm update --no-save)
}

_me_npm_install() {
    (cd "${_ME_NPM_PROJECT}" && npm install)
}
```

And when it's all set up:

```bash
💻  $ which bash-language-server
/Users/rpk/Library/init/tools-in-npm/node_modules/.bin/bash-language-server
💻  $ bash-language-server --version
5.6.0
```

Now, suppose you wanted one of those new-fangled agentic things.
You could add this, to your `package.json`:

```bash
      "@anthropic-ai/claude-code": ">1.0.0",
```

# Windows

How _would_ you do this on Windows? Outside of Cygwin or MSYS, using
only, say, CMD or PowerShell, I am not sure.

[^1]: Yes, I know that was a long time ago. Then my "brain" repository was migrated to Subversion, and finally to Git.
