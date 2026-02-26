# Documentation template for Phoebus projects

## Instructions

- Clone this repository to get the template
- In your project, make sure to move the old documentation out of the way
- Copy every file from the template to the root directory of your project, including hidden files, excluding the README.md
- Change the project name in `docs/pyproject.toml`
- Change the project name in `docs/conf.py`
- Change the project name and source code repository URL in `docs/index.md`
- Choose your profiles, remove the unneeded ones by:
  - Removing the directory
  - Removing the toctree in `docs/index.md`
- Re-integrate the old documentation into the new structure
- Remove any unused sections from the documentation, as they can hide what's actually written

## Read the Docs setup

- Create a project on Read the Docs, connected to this repository
- Make sur to enable Pull Request builds on the settings page

