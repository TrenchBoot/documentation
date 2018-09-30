# Contributing to TrenchBoot

We love your input! We want to make contributing to this project as easy and
transparent as possible, whether it's:

- Reporting a bug
- Discussing the current state of the code
- Submitting a fix
- Proposing new features
- Becoming a maintainer

## Project Structure

The functionality being developed under TrenchBoot are cross-cutting
capabilities that span multiple open source projects.  The role of TrenchBoot
is to function as both a development as well as a cross-project integration
project. As such it maintains a set of repository clones of upstream
project(s) that TrenchBoot conducts development within. 

### Upstream Repository

For each upstream repoository within TrenchBoot will have at least one maintainer.
The maintainer(s) will have merge permissions and responsible for maintaining
adherence to upstream practices. When necessary, they are responsible for
maintaining out-of-tree capabilities developed by TrenchBoot.

#### Upstream Focus

One of the primary objectives for TrenchBoot is to deliver interoperable launch
integrity capabilities to existing open source projects involved with the
system boot cycle. Under this objective, all development against an upstream
project must comply with upstream coding style and strive to minimize
disruption/breakage of upstream capabilities.

#### Out of Tree Maintenance

There may be a case that a TrenchBoot capability may encounter a slow adoption
by an upstream project which results in multiple upstream releases without the
capability merged. As such the capability in question must be kept in sync with
upstream changes. While this situation is not desired, it is likely to occur
and will be the responsibility of the respective TrenchBoot maintainer(s).

### TrenchBoot Original Repositories

While the focus is on delivering capabilities into upstream projects, it is
possible that a new or derivative code base may result to fulfill a role in a
cross-cutting capability. The maintainer(s) for these code bases will be
responsible for establishing coding styles and licensing that ensure compliance
with all other TrenchBoot relevant projects.

## Development

We use github to host the project, to include tracking issues and feature
requests, as well as accept pull requests.

### Development Flow Overview

Pull requests are the best way to propose changes to the project (we use
[Github Flow](https://guides.github.com/introduction/flow/index.html)). We
actively welcome your pull requests:

1. Fork the repo and create your branch from `master`.
2. If you've added code that should be tested, add tests.
3. If you've changed APIs, update the documentation.
4. Ensure the test suite passes.
5. Make sure your code lints.
6. Issue that pull request!

### Contribution Licensing

TrenchBoot is a cross-community integration project consisting of project
repositories along with upstream project repositories. All contributions to a
repository are made under the license that covers all work in that repository.

### Issue Tracking

Report issues, e.g. bugs, feature requests, etc., using Github's
[issues](https://github.com/briandk/transcriptase-atom/issues)

**Great Issues** tend to have:

- A quick summary and/or background
- For bug reports include:
  - Steps to reproduce for bugs
    - Be specific!
    - Give sample code if you can.
  - What you expected would happen
  - What actually happens
  - Notes (possibly including why you think this might be happening, or stuff
    you tried that didn't work)
- For feature requests include:
  - An outline of how the feature would work
  - Any dependecies the feature would require
  - The benefit the feature will provide

### Coding Style

Markdown documents should be formatted to 80 columns to the extent possible.
Exception to the 80 column is when column limitation breaks markdown rendering.

Contributions targeting upstream project repositories will follow the upstream
project's coding style rules.

## License
[![License: CC BY 4.0](https://i.creativecommons.org/l/by/4.0/88x31.png)
](https://creativecommons.org/licenses/by/4.0/) This work is licensed under a
[Creative Commons Attribution 4.0 International
License](ttp://creativecommons.org/licenses/by/4.0/).  By contributing you
agree that your contributions will be licensed under the Creative Commons
Attribution 4.0 International License. Feel free to contact the maintainers if
that's a concern.

## Contacting Maintainers

TrenchBoot is maintained by:
 - Daniel P. Smith <dpsmith@apertussolutions.com>

# References
This document was adapted from the this open-source [contribution
template](https://gist.github.com/briandk/3d2e8b3ec8daf5a27a62/)
