# Contributing

Hi there! First off, we're thrilled 🤩 you want contribute to this project!

First time contributor to a GitHub project? If you could use some help getting
started, [take a look at this quick and easy guide][1]. 💜

## Briefly: Submitting a Pull Request (PR)

> See also: [CODE_OF_CONDUCT.md][2]

This repository uses a [fully automated][3] [continuous linting][4] (CL),
[integration testing][5] (CI), and [deployment][5] (CD)
[semantic-release][6]-based pipeline for integrating PRs and publishing
releases. The nice thing about a fully automated CL/CI/CD pipeline is that
anyone anywhere can make a contribution quickly and with minimal tedium all
around!

This repository makes extensive use of [debug][7] (through [rejoinder][8]).
Should you wish to view all possible debugging output, [export `DEBUG='*'`][9].
To get debugging output for just this project's components, [export
`DEBUG='@xunnamius/dummy-pkg-2:*'`][9].

The ideal contributor flow is as follows:

1.  [Fork][10] this repository and [clone it locally][11]
2.  Configure and install dependencies: `npm ci`
    - You use `npm ci` here instead of `npm install` to [prevent unnecessary
      updates to `package.json` and `package-lock.json`][12], but if it makes
      more sense to use `npm install` feel free to use that instead
    - If `.env.example` exists, consider copying it to `.env` and configuring
      sensible defaults
    - If you're using `npm@<=6`, you'll need to install any [peer
      dependencies][13] manually. If you're using `npm@>=7`, you may have to
      [forcefully][14] allow peer deps to be satisfied by custom forks of
      certain packages
3.  Before making any changes, ensure all unit tests are passing: `npm run test`
4.  _(optional but recommended)_ Create a new branch, usually off `main`
    - Example: `git checkout -b contrib-feature-1`
5.  Make your changes and commit. Thanks to CL, your work will be checked as you
    commit it; any problems will abort the commit attempt
6.  Push your commits to your fork and, when you're ready, [_fearlessly_ submit
    your PR][15]! Your changes will be tested in our CI pipeline
7.  Pat yourself on the back! Your hard work is well on its way to being
    reviewed and, if everything looks good, merged and released 🚀

Additionally, there are a few things you can do to increase the likelihood your
PR passes review:

- **Do** [open an issue][16] and discuss your proposed changes (to prevent
  wasting your valuable time, e.g. _maybe we're already working on a fix!_), and
  [search][17] to see if there are any existing issues related to your concerns
- **Do** practice [atomic committing][18]
- **Do not** reduce code coverage ([codecov][19] checks are performed during CI)
- **Do** [follow convention][20] when coming up with your commit messages
- **Do not** circumvent CL, i.e. automated pre-commit linting, formatting, and
  unit testing
- **Do** ensure `README.md` and other documentation that isn't autogenerated is
  kept consistent with your changes
- **Do not** create a PR to introduce [_purely_ cosmetic commits][21]
  - Code de-duplication and other potential optimizations we **do not** consider
    _purely_ cosmetic 🙂
- **Do** keep your PR as narrow and focused as possible
  - If you ran `npm install` instead of `npm ci` and it updated `package.json`
    or `package-lock.json` and those updates have nothing to do with your PR
    (e.g. random nested deps were updated), **do not** stage changes to those
    files
  - If there are multiple related changes to be made but (1) they do not
    immediately depend on one another or (2) one implements extended/alternative
    functionality based on the other, consider submitting them as separate PRs
    instead 👍🏿

> Be aware: all contributions to this project, regardless of committer, origin,
> or context and immediately upon push to this repository, are [released][22] in
> accordance with [this project's license][23].

---

At this point, you're ready to create your PR and ✨ contribute ✨. What follows
is a description of this project's automated [CL][4]/[CI/CD][5] pipeline and NPM
scripts; **this is optional reading for external collaborators.** You're done!

---

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

## [The Build-Test-Deploy Pipeline][5]

Development in this repository adheres to [Trunk Based Development][24]
principles, specifically leveraging [_short-lived feature branches_][25] (SLFB)
and Continuous Linting (CL), Integration (CI), and Deployment (CD).

Broadly speaking, this pipeline consists of three "sub-pipelines" put together
front to back:

- First, the so-called "Continuous Linting" pipeline, which automatically runs
  formatting, linting, and unit testing locally on the developer's machine
  _before every commit_. This [tightens the developer feedback loop][26] and
  [saves money][27].

- Once one or more commits are pushed to remote, the [Continuous
  Integration][28] (CI) pipeline runs next, which runs unit tests, project-wide
  integration tests, and project-wide linting concurrently upon every triggering
  event (below).

- Finally, if the CI pipeline terminates successfully (and other conditions are
  met), the [Continuous Deployment][29] (CD) pipeline runs. It builds, formats,
  versions, and ships to production on every commit. Production releases only
  occur on the addition of features, fixes, build system changes, or breaking
  changes.

These pipelines are situated one after the other such that the CD pipeline
always fails to publish when the CI pipeline check fails. Further, the CL
pipeline will reject local commits that fail to pass unit testing before they
ever reach the CI pipeline.

### Pipeline Usage and Structure

- The existence of certain files is assumed, such as `package.json`

- `main` is the only permanent branch, all other branches are automatically
  deleted after being merged into `main`

  - The term "merged" as used here and elsewhere in this document connotes a
    [non-ff merge][30] operation. Note that [ff merge][30], [rebase][31], and
    [squash][32] operations can be used as well **except when merging between
    release branches (like `main` and `canary`); [only non-ff merge operations
    should be used to merge between release branches][33]; any other operation
    (_[including force pushing][34]_) risks damaging `semantic-release`'s
    version tracking metadata**!

  - Technically, there are also [maintenance branches][35], which are
    semi-permanent

  - For NPM package projects, this also means `latest` is the only permanent
    [dist-tag][36]

- Changes are committed directly to `main`, to a SLFB that is eventually merged
  into `main`, or through a PR that is eventually merged into `main` from an
  external repository

- `canary` is a special SLFB used to publish commits on the canary release
  channel before before they're merged into `main` (useful to combine multiple
  features as a single testable release)

  - For projects that are deployed (e.g. Vercel, web push, etc), `canary` may be
    used as a permanent preview branch in addition to the permanent `main`
    branch

- Pushing a commit to any branch, opening a PR against `main`/`canary`, or
  synchronizing a PR made against `main`/`canary` will trigger the CI pipeline

- Pushing a commit directly to `main` or `canary` will trigger the CI pipeline
  and, if all tests pass, also trigger the [semantic-release][37]-based CD
  pipeline where:

  - Commits pushed to `main` are released on the [default release channel][38]

  - Commits pushed to `canary` are released on the [prerelease channel][39]

  - Commits pushed to `N.x`/`N.x.x` and `N.N.x` branches are released on their
    respective [maintenance channels][35]

  - Commits pushed to other release branches will also generate a release
    depending on [custom configuration][40]

  - Commits pushed to branches that aren't the above will never cause the CD
    pipeline to generate a release even if all tests pass

- Force pushing to `main` and `canary` will always fail (unless temporarily
  allowed)

- Commits that include `BREAKING:`, `BREAKING CHANGE:`, or `BREAKING CHANGES:`
  in their message body will be treated as major release commits and will appear
  in [CHANGELOG.md][41] regardless of their type

  - Example:

    ```shell
    git commit -m "debug: this commit will cause a major version bump and
    will appear in the changelog, even though it's only a debug commit!!!

    BREAKING CHANGE: this feature replaces that feature
    BREAKING CHANGE: this other feature now also works differently"
    ```

- PRs only trigger the CI pipeline and _never_ the CD pipeline

- The CD pipeline never runs in forks of this repository, even when GitHub
  Actions are explicitly enabled (this can be overridden)

- All tags created through this pipeline are annotated and automatically signed.
  To support this and other features that require annotated tags, we use a
  [custom fork of semantic-release][42]

  - Hopefully [support for][43] [annotated tags][44] will be included upstream
    one day.

- The CD pipeline will not publish to NPM [so long as package.json contains
  `private: true`][45]

- Note that **[all reverts are treated as patches and immediately
  released][46]** no matter the type of the reverted commit. This means
  **commits that were reverted will appear in [CHANGELOG.md][41] even if they
  didn't trigger an earlier release**

  - This also means **reverting a commit that introduced a breaking change will
    only trigger a patch release** unless the revert commit itself also includes
    `BREAKING CHANGE:` in its message body

  - If a push includes only revert commits (and `BREAKING CHANGE:` or an
    alternative is not present in the top commit's message body), **the result
    is always a patch release!**

### Pipeline Trigger Events

The CI/CD pipeline is triggered by two [events][47]:

- `push` events that:
  - Are of non-tag refs (pushed tags are ignored by CI/CD)
  - Are of refs with names not starting with `dependabot/`, `snyk-`, or `no-ci/`
- `pull_request` events that:
  - Are of type `synchronize` or `opened`
  - Compare against branches `main` or `canary`

> For NPM packages, [the `cleanup` workflow][48] prunes [dist-tags][36]
> associated with deleted branches and is triggered by the `delete` event.

> The units that make up the pipeline can usually be triggered manually. For
> workflows, manual invocations are treated as `push`/`delete` events.

This is further described by the following flow chart of events:

    pushed `main` ==> [run CI] ==> tested ==> [run CD if CI passed] ==> released vx.y.z
    pushed `canary` ==> [run CI] ==> tested ==> [run CD if CI passed] ==> released vx.y.z-canary.N
    pushed any other SLFB* ==> [run CI] ==> tested
    PR opened against `main`/`canary` ==> [run CI] ==> tested
    PR synchronized against `main`/`canary` ==> [run CI] ==> tested

<small>\* Excluding branches with names starting with `dependabot/`, `snyk-`, or
`no-ci/`</small>

When the CI/CD pipeline is triggered, jobs are executed according to the
following chronology:

    gather metadata ==> [CI] security audit
                        [CI] linters
                        [CI] unit tests
                        [CI] integration tests
                        [CD] install, build, format, sort ==> [CD] release

Jobs in the same column are executed concurrently. A job failing in one column
prevents the pipeline from proceeding to the next column.

This pipeline supports four suites of integration tests: _node_, _externals_,
_client_ (for browsers/cli/etc), and _webpack_. The presence of these test
suites is picked up by `grep`-ing the output of `npm run list-tasks` to search
for the presence of the script keys `test-integration-node`,
`test-integration-externals`, `test-integration-client`, or
`test-integration-webpack` respectively.

This pipeline also supports an optional documentation build step via the
`build-docs` key. A warning will be generated for projects that lack this key.
Similarly, the pipeline will fail if there is a `build-externals` key without a
`test-integration-externals` key or vice-versa.

Note that internal PRs to `main`/`canary` made from pushing to internal branches
**whose names do not begin with `no-ci/`** will trigger two CI runs: one on the
`push` event generated by pushing to said branch and the other on the subsequent
`pull_request` event when the PR is opened (type: `opened`) or its merge commit
is updated (type: `synchronize`). If this is a problem (i.e. wasting money),
prepend `no-ci/` to the internal branch name or transition to a _clone-and-pull_
workflow instead of _branch-and-pull_.

### Pipeline Commit Commands

There are several commands that can affect the behavior of the pipeline. To use
them, include them as part of the top commit's message when pushing to remote.
When a single push consists of multiple commits, only the very top commit's
message is parsed for commands.

The following commands are recognized:

| Command     | Alias(es)   | Description                                      | Usage Example                                          |
| ----------- | ----------- | ------------------------------------------------ | ------------------------------------------------------ |
| `[skip ci]` | `[ci skip]` | Skip the [CI workflow][5] (implies `[skip cd]`)  | `git commit -m 'build: fix CI system [skip ci]'`       |
| `[skip cd]` | `[cd skip]` | Skip only the semantic-release-based CD pipeline | `git commit -m 'style: do-not-release-this [skip cd]'` |

## NPM Scripts

This repo ships with several [NPM scripts][49]. Use `npm run list-tasks` to see
which of the following scripts are available for this project.

> Using these scripts requires a linux-like development environment. None of the
> scripts are likely to work on non-POSIX environments. If you're on Windows,
> use [WSL][50].

### Developing

- `npm run dev` to start a development server or instance
- `npm run lint` to run a project-wide type check (handled by CL/CI)
- `npm run test` (or `npm test`, `npm run test-unit`) to run the unit tests
  (handled by CL/CI)
  - Also [gathers test coverage data][51] as HTML files (under `coverage/`)
  - Can also run `npm run test-integration` to run all the integration tests
- `npm run test-integration-node` to run integration tests on the last three LTS
  Node versions (handled by CI)
- `npm run test-integration-client` to run client (browser/cli/etc) integration
  tests with [puppeteer][52] (handled by CI)
- `npm run test-integration-webpack` to run tests verifying the distributable
  can be bundled with Webpack 4 and 5 (as ESM, CJS, or both) (handled by CI)
- `npm run test-integration-externals` to run tests on compiled external
  executables (under `external-scripts/bin/`) (handled by CI)

> Note: `npm test` and all the `npm run test*` commands accept an optional
> `JEST_CLI` environment variable, the contents of which will be passed directly
> to Jest. For example: `JEST_CLI='--verbose' npm test` to get verbose output.

#### Other Development Scripts

- `npm run test-repeat` to run the entire test suite 100 times
  - Good for spotting bad async code and heisenbugs
  - Uses `__test-repeat` NPM script under the hood
- `npm run generate` to transpile config files (under `config/`) from scratch
- `npm run regenerate` to quickly re-transpile config files (under `config/`)
- `npm run postinstall` to (re-)install [Husky Git hooks][53] if not in a CI
  environment (handled by NPM)

### Building and Deploying

- `npm run build` (alias: `npm run build-dist`) to compile `src/` into `dist/`
  (or `build/`), which is what ships to production (handled by CI/CD)
- `npm run format` to run source formatting over the codebase (handled by
  CL/CI/CD)
- `npm run start` to deploy a _local production mode_ instance
- `npm run deploy` to deploy to production (bring your own auth tokens) (handled
  by CD)

#### Other Build Scripts

- `npm run clean` to delete all build process artifacts (except `node_modules/`)
- `npm run build-changelog` to re-build the changelog (handled by CI/CD)
  - You can run this as `CHANGELOG_SKIP_TITLE=true npm run build-changelog` to
    skip prepending the header
- `npm run build-docs` to re-build the documentation (handled by CI/CD)
- `npm run build-externals` to compile `external-scripts/` into
  `external-scripts/bin/`
- `npm run build-stats` to gather statistics about Webpack (look for
  `bundle-stats.ignore.json`)

### NPX Scripts

> These commands might be installed as a project dependency but are expected to
> be run using [`npx X`][54] instead of `npm run X` regardless.

- `npx npm-force-resolutions` to forcefully patch security audit problems
- `npx semantic-release -d` to run the CD pipeline locally (in [dry-run
  mode][55])

[1]: https://www.dataschool.io/how-to-contribute-on-github
[2]: /.github/CODE_OF_CONDUCT.md
[3]: https://github.com/features/actions
[4]: https://github.com/Xunnamius/projector-lens-cli/tree/main/.husky
[5]: .github/workflows/build-test-deploy.yml
[6]: https://github.com/semantic-release/semantic-release#readme
[7]: https://www.npmjs.com/package/debug
[8]: https://www.npmjs.com/package/rejoinder
[9]: https://www.npmjs.com/package/debug#wildcards
[10]: https://github.com/Xunnamius/projector-lens-cli/fork
[11]:
  https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository
[12]: https://docs.npmjs.com/cli/v6/commands/npm-ci
[13]:
  https://docs.npmjs.com/cli/v6/configuring-npm/package-json#peerdependencies
[14]:
  https://docs.npmjs.com/cli/v7/commands/npm-install#configuration-options-affecting-dependency-resolution-and-tree-design
[15]: https://github.com/Xunnamius/projector-lens-cli/compare
[16]: https://github.com/Xunnamius/projector-lens-cli/issues/new/choose
[17]: https://github.com/Xunnamius/projector-lens-cli/issues?q=
[18]: https://www.codewithjason.com/atomic-commits-testing/
[19]: https://about.codecov.io/
[20]: https://www.conventionalcommits.org/en/v1.0.0/#summary
[21]: https://github.com/rails/rails/pull/13771#issuecomment-32746700
[22]:
  https://help.github.com/articles/github-terms-of-service/#6-contributions-under-repository-license
[23]: LICENSE
[24]: https://trunkbaseddevelopment.com/
[25]: https://trunkbaseddevelopment.com/#scaled-trunk-based-development
[26]:
  https://blog.nelhage.com/post/testing-and-feedback-loops/#invest-in-regression-testing
[27]: https://github.com/pricing
[28]: https://en.wikipedia.org/wiki/Continuous_integration
[29]: https://en.wikipedia.org/wiki/Continuous_deployment
[30]: https://git-scm.com/docs/git-merge#Documentation/git-merge.txt---no-ff
[31]: https://git-scm.com/docs/git-rebase
[32]: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#_squashing
[33]:
  https://github.com/semantic-release/git#merging-between-semantic-release-branches
[34]:
  https://semantic-release.gitbook.io/semantic-release/support/troubleshooting#release-not-found-release-branch-after-git-push-force
[35]:
  https://semantic-release.gitbook.io/semantic-release/usage/workflow-configuration#maintenance-branches
[36]: https://docs.npmjs.com/cli/v6/commands/npm-dist-tag#purpose
[37]: https://www.npmjs.com/package/semantic-release
[38]:
  https://semantic-release.gitbook.io/semantic-release/usage/workflow-configuration#release-branches
[39]:
  https://semantic-release.gitbook.io/semantic-release/usage/workflow-configuration#pre-release-branches
[40]: release.config.js
[41]: CHANGELOG.md
[42]: https://github.com/Xunnamius/semantic-release/tree/contrib-holistic
[43]: https://github.com/semantic-release/semantic-release/pull/1709
[44]: https://github.com/semantic-release/semantic-release/pull/1710
[45]: https://github.com/semantic-release/npm#options
[46]:
  https://github.com/semantic-release/commit-analyzer/blob/e8c560459d7ef8752180154ed0263ce262aa22a7/lib/default-release-rules.js#L8
[47]:
  https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows
[48]: .github/workflows/cleanup.yml
[49]: https://docs.npmjs.com/cli/v6/commands/npm-run-script
[50]: https://docs.microsoft.com/en-us/windows/wsl/install-win10
[51]: https://jestjs.io/docs/en/cli.html#--coverageboolean
[52]: https://github.com/puppeteer/puppeteer
[53]: https://github.com/typicode/husky
[54]: https://www.npmjs.com/package/npx
[55]:
  https://semantic-release.gitbook.io/semantic-release/usage/configuration#dryrun