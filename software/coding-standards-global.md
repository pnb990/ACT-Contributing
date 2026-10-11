<!--
SPDX-FileCopyrightText: 2024 - 2026 Benoit Rolandeau <benoit.rolandeau@allcircuits.com>
SPDX-FileCopyrightText: 2026 Pierre-Noel Bouteville <pierre-noel.bouteville@allcircuits.com>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
-->

# Global coding standards

Language-agnostic rules that apply to every ACT software repository. Each rule has a stable id
(`RGxx`) so it can be cited in code review and linked from other repositories. Treat these ids as a
public API: their wording may change, but a code (`RG8`) never moves to a different rule and is
never reused.

These global rules are the base layer. A language standard (C, C++, Qt, Flutter) or a project
standard may tighten a value; the most specific one wins: `project > language > global`. See
[coding-standards.md](coding-standards.md) for the override mechanism.

**Every rule below is blocking: a merge request that violates one is not merged until it is fixed.**

## Table of contents

- [Language and formatting](#language-and-formatting)
- [Version control and workflow](#version-control-and-workflow)
- [Building and delivery](#building-and-delivery)
- [Testing](#testing)
- [Design and code quality](#design-and-code-quality)
- [Licensing](#licensing)

## Language and formatting

### RG1 - No more than 1000 lines

Keep a source file under 1000 lines. Past that, split it by responsibility - a file that long is
almost always doing several jobs.

### RG2 - No more than 100 characters per line

Keep lines under 100 characters. A language or project standard may set a different limit (80 in C,
for example); the most specific one wins.

### RG3 - Indent with 4 spaces

Indent with spaces, never tabs. One indentation level is 4 spaces.

### RG4 - Write code in English

Write code in English: identifiers, symbols, everything the compiler sees.

### RG5 - Write code documentation in English

Write in-code documentation (comments, doc blocks) in English.

### RG21 - Encode files in UTF-8

Encode every source file in UTF-8.

### RG32 - Declare the code formatter version in the file header

Auto-formatters such as `clang-format` change their output from one major version to the next, and
a workstation usually has the version its distribution ships. A project that does not follow - a
legacy one, or one that cannot reformat its whole tree right now - is then reformatted differently
by each developer, which pollutes the diffs and hides the real changes.

Every source file in a language the project formatter handles declares, in its header right after
the SPDX block ([RG22](#rg22---carry-spdx-licensing-on-every-file)), the **major** version of the
formatter its layout complies with, with the exact tag `clang-format-version: <major>`:

```c
// SPDX-FileCopyrightText: 2026 Jane Doe <jane.doe@allcircuits.com>
//
// SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
//
// clang-format-version: 19
```

- Format the file with the declared version only. On Debian the packages are versioned
  (`clang-format-19`), so several versions can live side by side.
- Migrating a file to a newer version updates the tag **in the same commit** as the reformatting,
  and that commit contains nothing else.
- Files that must never be formatted (generated code, third-party trees) are excluded at project
  level, in the formatter and CI configuration. There is no per-file opt-out.
- CI ([RG27](#rg27---build-and-lint-the-project-with-ci)) fails when a file lacks the tag, or when
  `clang-format-<major> --dry-run --Werror` with the declared version would change it.

During a migration several versions coexist under one `.clang-format`, so keep that configuration
readable by the **oldest** version declared in the tree, and state on its first lines the version
it is written for. A project mixing C and C++ has one configuration per source tree. To adopt this
rule on an existing project, add the tag with the current version to every file in a single
mechanical commit, before any reformatting.

## Version control and workflow

### RG6 - Store code in a version control system

Store any code you develop in a version control system (Git, SVN, etc.), whether on your own
platform (GitHub, GitLab, Bitbucket) or the client's.

### RG7 - Keep at least one stable branch

Every repository has a stable branch. A branch is stable when, at any time, you can recover a fully
functional application, library, or firmware from it. Bugs found on it are logged in the tracker
and triaged: fix now, or plan for a later cycle. Name it `master`, `main`, or `stable` as you wish.

### RG8 - Work in development branches

Develop in a working branch, never directly on the stable branch. Branch names follow the pattern:

```text
tmp/xxxx-branch-name
```

- `xxxx`: the id of the associated task in your tracker (Redmine, Jira, GitHub, GitLab, etc.)
- `branch-name`: a short summary of the task's purpose

For example:

```text
tmp/845-add-main-app-logo
```

### RG10 - Tie code to a requested feature

Write only code that serves a feature requested by the customer, the ordering party, or agreed
together. Forgetting a feature and adding an unrequested one are equally wrong.

Speculative features cost more than they save: when the need becomes real it rarely matches what you
guessed, so you refactor anyway. If that speculative code sits at the base of the dependency tree,
the refactor is expensive and bug-prone. On a shared library or open-source project, discuss a
feature's relevance and priority before building it.

### RG11 - Keep the task tracker up to date

Because code is written against features tracked as tasks or issues, keep each task's status current
as you progress: started, in review, merged into the stable branch, and so on.

### RG26 - Peer review code before it enters a stable branch

Code reaches a stable branch (`master`, `main`, `stable`) only through peer code review.

A proof-of-concept project, one not meant for production, may skip the review; its code still goes
through development branches ([RG8](#rg8---work-in-development-branches)).

## Building and delivery

### RG9 - Keep the code building

Code that compiles must build successfully before it is merged into the stable branch.

Distinguish work in progress from merge candidates. A development branch may hold code that does not
build or run - say so in the commit message. Code ready to merge into the stable branch must build.

### RG12 - Merge only working code

Code merged into the stable branch works as expected. This is partly subjective and open to
discussion with reviewers.

A review reads the code, so a reviewer cannot always tell whether it runs. When there are no unit
tests and the feature or code is tricky, the reviewer should run it to confirm it works, and check
that what was built matches what was requested.

### RG27 - Build and lint the project with CI

Build and lint the project with continuous integration tools (Jenkins, GitLab CI, GitHub Actions,
etc.). CI proves the code builds on a machine other than the author's - developers install many
tools that can make software work by accident - and enforces the rules that can be checked
automatically.

### RG28 - Produce deliverables with CI/CD

When you release the result of a build, produce the deliverable with CI/CD tools, not on your own
machine. A developer's machine has an uncontrolled toolchain, so its output is not reproducible; and
if you are away or your machine breaks, a colleague still needs to be able to deliver. Applies when
the source goes through a compiler.

### RG29 - Provide a README with a presentation and quick start

Every repository has a `README.md` at its root, with a short presentation of the project and a quick
start covering how to install the dev environment, build the library or application, and deploy it.
It also links to the coding standards the project follows: these standards and, if any, the
project's own.

### RG30 - Pin CI and container dependencies by digest

In CI/CD definitions (GitHub Actions, GitLab CI, etc.) and Dockerfiles, reference external actions
and container images by their immutable digest - a commit SHA for an action, a `sha256` digest for
an image - never by a tag alone. A tag can be moved or deleted: pinning protects against
supply-chain attacks, keeps builds reproducible, and avoids extra registry requests and rate limits.

Keep the human-readable version next to the digest, so readers know what it is and can bump it: in a
comment for an action, as the tag in the `name:tag@sha256:...` form for an image.

```yaml
- uses: actions/checkout@df4cb1c069e1874edd31b4311f1884172cec0e10 # v6.0.3
```

```dockerfile
FROM debian:trixie@sha256:9cc080028c43b27d2074d63a5f9caf7166d731494965616c1a6d2827a004585c
```

## Testing

Rules in this section apply when unit tests are to be developed for the project.

### RG13 - Cover requested features with unit tests

Cover the features requested by the customer, the ordering party, or agreed together with unit
tests.

### RG14 - Stub third-party dependencies in unit tests

When unit tests depend on third-party elements provided by the customer or ordering party, develop
software stubs for them. Stubbing isolates your code, so a detected bug points clearly at your side
rather than the third party's.

### RG15 - Make unit tests automatable

Write unit tests so they can be automated.

### RG16 - Pass all unit tests before merging

All unit tests pass before the code is merged into the stable branch.

## Design and code quality

### RG19 - Keep the code maintainable

Write code clear enough for another developer to maintain. This is subjective and best settled
during design, or during peer review - though by review it is almost too late.

### RG20 - Design for foreseeable evolution

Design code so it can evolve toward the customer's future and already-stated needs. This is
subjective and depends on how well the client can project themselves, but medium-term direction is
usually known, so account for it during design while staying close to the primary need.

For example, if the client asks for shoes and expects to run in them later, sneakers are a better
base design than sandals or dress shoes.

### RG24 - No dead code without a justification

No dead code: neither commented-out code nor code that is never called. Dead code raises the reading
cost and misleads readers about how the feature actually works.

Keep dead code only for a stated reason - a class not wired up yet, an example kept on purpose - and
write that reason in a comment right before it. The reviewer judges whether the justification holds.

### RG25 - Use the TODO and FIXME comment style

Write `TODO` or `FIXME` in all caps, followed by who wrote the comment - a username, a name or
initials, whatever lets a reader find that person - then the message, optionally followed by a link
to the issue.

```c
// TODO(username): message.
// FIXME(TM): message, https://URL-to-issue.
```

A TODO without an author cannot be followed up: nobody knows whom to ask what was left undone.

### RG31 - Handle every error, or stop visibly

When a program reaches a state it does not know how to handle, it stops the activity that led
there, makes the failure visible and recordable, and preserves what is needed to diagnose it. It
never carries on in a state it can no longer describe, and never relies on an outside mechanism to
notice the failure for it.

An error is handled when the code knows which state it is in afterwards. Catching a failure to turn
it into a documented error and passing it up is handling it. The following are not, and are
forbidden:

- a catch-all that does nothing, or only logs and lets execution continue: `catch (...) { }`,
  `catch (e) { }` and their equivalent in every language;
- an error return value, status or code that no caller reads;
- an asynchronous failure nobody is attached to: a `Future` or `Promise` with no error handler, a
  stream subscribed without `onError`;
- an empty wait loop or a silent return standing in for the failure handling;
- a retry that hides a permanent failure;
- counting on a watchdog, supervisor, orchestrator or automatic restart to detect the failure. Such
  a mechanism is often not deployed, and when it acts, the state that would have explained the
  failure is already gone.

What "visible and recordable" means depends on the product, and the language standards say with
which mechanism. On a desktop application or a service: a log entry carrying the context and the
call stack, and a controlled termination of the operation. On a microcontroller with no console:
the dead end of [RC19](coding-standards-c.md#rc19---panic-handler) - safe state, identifying
pattern, reset.

### RG33 - One exit point per function

Give each function a single `return`, at its end. Carry the outcome in a result variable and reach
the end through structured control flow, instead of scattering early returns. A function then has
one place where its result is decided and one place to put a breakpoint, a trace or a cleanup.

```c
int pwm_start(uint32_t freq_hz)
{
    int result = -1;

    if (pwm_is_configured())
    {
        result = pwm_timer_start(freq_hz);
    }

    return result;
}
```

When a single exit genuinely makes a function harder to read, readability wins - but that is the
exception, and the reviewer judges it. In a language with exceptions, raising one is the error exit
and does not count as a second `return`.

### RG34 - No goto

Never use `goto`, including the C "goto cleanup" error-handling idiom. Guard the steps that follow
a failure with nested blocks or with the result variable of
[RG33](#rg33---one-exit-point-per-function), and release resources at the single exit.

```c
int log_save(const char *path)
{
    int result = -1;
    FILE *file = fopen(path, "w");

    if (file != NULL)
    {
        result = log_write(file);
        (void)fclose(file);
    }

    return result;
}
```

## Licensing

### RG17 - Check third-party licenses for compatibility

The licenses of third-party libraries you use must be compatible with your own license(s), the
context you develop in (company, open source, etc.), and, for a customer project, how the customer
intends to distribute it.

### RG18 - Use only software you have the right to use

Do not use a library, program, or source you do not have the right to use in the context you use it
in.

### RG22 - Carry SPDX licensing on every file

Every file's copyright and license are declared in SPDX form:

```text
SPDX-FileCopyrightText: YYYY First name Name <email>

SPDX-License-Identifier: LicenseRef-ALLCircuits-ACT-1.1
```

Declare it in one of three ways, in order of preference:

- an inline header, commented in the file's own syntax (`//` for C/C++, `#` for Python and shell,
  etc.);
- a sibling `<file>.license` holding the same header, for a binary or non-commentable file (for
  example `icon.png.license`);
- a `REUSE.toml` entry covering a folder or a set of files, for a complex or third-party tree (use
  it sparingly).

Update the license version when needed. Reviewers make sure outdated headers are updated.

### RG23 - Keep a LICENSES directory

Keep a `LICENSES/` folder at the repository root with one text file per license in use, including at
least `LicenseRef-ALLCircuits-ACT-X.Z.txt`. For example, add `MIT.txt` when the MIT license is used.
