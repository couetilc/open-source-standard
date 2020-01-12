# Standard for Open Source Projects

  Learn how to develop and publish an open-source project using established
  standards that ensure software is of quality.

# What is Open Source Software

  Open-source software adheres to [10 principles](https://opensource.org/osd-annotated)
  as defined by the [Open Source Initiative](https://opensource.org/):

  1. Free Redistribution
  2. Source Code
  3. Derived Works
  4. Integrity of the Author's Source Code
  5. No Discrimination Against Persons or Groups
  6. No Discrimination Against Fields of Endeavor
  7. Distribution of License
  8. License Must Not Be Specific to a Product
  9. License Must Not Restrict Other Software
  10. License Must Be Technology-Neutral

# Evaluating Software Quality

  ISO 9126 is an international standard for the evaluation of software which
  established 6 quality characteristics used when evaluating software:

    - Functionality
      - (Suitability, Accurateness, Interoperability, Compliance, Security)
    - Reliability
      - (Maturity, Fault Tolerance, Recoverability)
    - Usability
      - (Understandability, Learnability, Operability)
    - Efficiency
      - (Time behavior, Resource behavior)
    - Maintainability
      - (Analyzability, Changeability, Stability, Testability)
    - Portability
      - (Adaptability, Installability, Conformance, Replaceability)

  These characteristics should be used as a guide when evaluating and describing
  software in technical documentation for an open-source project.

## Components of Open Source Projects

  0. licensing (https://opensource.org/licenses) (https://choosealicense.com/)
    - default to MIT License, an established standard for open source software
      allowing for permissive use of the software while requiring preservation
      of copyright and absolving liability for the software author.
  1. documentation
    - README
    - CONTRIBUTING (https://github.blog/2016-02-17-issue-and-pull-request-templates/)
    - [Setting up your project for healthy contributions](https://help.github.com/en/github/building-a-strong-community/setting-up-your-project-for-healthy-contributions)
  2. source control (git)
  3. package management
  4. formatting (linting/prettifying)
  5. build scripts
  6. tests
  7. benchmarks (e.g. package size, sort speed, test coverage)
  8. publishing/versioning

## Source Control

  Tracking changes to source code between versions helps developers enable
  collaboration and manage a codebase over time. Git is the most widely adopted
  version control system, is free and open-source, and integrates with many
  tools.

### Githooks

  Githooks are programs that trigger actions at certain points during git's
  execution. See https://git-scm.com/docs/githooks. When developing an open-
  source project, githooks are useful for enforcing code quality standards and
  removing friction from the development process. The following are recommended
  githooks for use during project development.

#### post-checkout

  Install javascript dependencies

  ```sh
  npm ci
  ```

#### post-merge

  Install javascript dependencies

  ```sh
  npm ci
  ```

#### pre-commit

  Check code base against style standards

  ```sh
  if !npm run checkLint; then
    echo 'Failed lint check'
    exit 1
  fi
  ```

#### pre-push

  Check code base style and run test suite

  ```sh
  if !npm run checkLint; then
    echo 'Failed lint check'
    exit 1
  fi

  if !npm run checkPretty; then
    echo 'Failed formatting check'
    exit 1
  fi

  if !npm run test; then
    echo 'Failed test suite'
    exit 1
  fi
  ```

#### prepare-commit-msg

  Prepend the branch name to the commit message

  ```sh
  # This way you can customize which branches should be skipped when
  # prepending commit message. 
  if [ -z "$BRANCHES_TO_SKIP" ]; then
    BRANCHES_TO_SKIP=(master develop test qa)
  fi

  BRANCH_NAME=$(git symbolic-ref --short HEAD)
  BRANCH_NAME="${BRANCH_NAME##*/}"

  BRANCH_EXCLUDED=$(printf "%s\n" "${BRANCHES_TO_SKIP[@]}" | grep -c "^$BRANCH_NAME$")
  BRANCH_IN_COMMIT=$(grep -c "\[$BRANCH_NAME\]" $1)

  if [ -n "$BRANCH_NAME" ] && ! [[ $BRANCH_EXCLUDED -eq 1 ]] && ! [[ $BRANCH_IN_COMMIT -ge 1 ]]; then 
    sed -i.bak -e "1s/^/[$BRANCH_NAME] /" $1
  fi
  ```

## Formatting

  A clear standard for written software style and formatting should be
  established to maintain quality and legibility of the code base.

  Tooling should be provided that minimizes effort spent maintaining the
  software's standards for style and formatting.

### Javascript

  The following sections configure style guidelines for javascript projects

  1. [Linting](#linting)
  2. [Prettifying](#prettifying)

#### Linting:

  [ESLint](https://eslint.org/) is the established standard for enforcing style
  in javascript projects. The software comes with a useful `--init` command
  which creates a project's linting configuration.

  **Instructions for installation and configuration of ESLint with _npm v5+_:**

  1. Install ESLint using `npm`, the node package manager.
  See https://eslint.org/docs/user-guide/getting-started

  ```sh
  npm install --save-dev eslint
  ```

  2. Configure the project's linting standards by answering the prompts from
  ESLint's `--init` command

  ```sh
  npx eslint --init
  ```

  3. Add a "lint" script to the project's package.json with the following
  contents

  ```sh
  eslint --cache "<source_code_directory>/*"
  ```

  4. Add a "checkLint" script to the project's package.json with the
  following contents

  ```sh
  eslint "<source_code_directory>/*"
  ```

  5. Ensure adherence to linting standards by adding lint checks to git's
  `pre-commit` and `pre-push` hook.

  **Add to pre-commit githook:**

  ```sh
  npm run lint
  ```

  **Add to pre-push githook**

  ```sh
  npm run checkLint
  ```

#### Prettifying:

  [Prettier](https://prettier.io/) is the established standard for automatic
  code formatting in javascript, with a focus on settling debates for the most
  common code style questions.

  **Instructions for installation and configuration of Prettier with _npm v5+_:**

  1. Install Prettier using `npm`, the node package manager.
  See https://prettier.io/docs/en/install.html

  ```sh
  npm install --save-dev --save-exact prettier
  ```

  2. Select style options from https://prettier.io/docs/en/options.html and
  configure rules according to https://prettier.io/docs/en/configuration.html

  3. Automatically format code when committing changes to git.
  See https://prettier.io/docs/en/precommit.html

  **Add to `pre-commit` githook:**

  ```sh
  #!/bin/sh
  FILES=$(git diff --cached --name-only --diff-filter=ACMR "*.js" "*.jsx" | sed 's| |\\ |g')
  [ -z "$FILES" ] && exit 0

  # Prettify all selected files
  echo "$FILES" | xargs ./node_modules/.bin/prettier --write

  # Add back the modified/prettified files to staging
  echo "$FILES" | xargs git add

  exit 0
  ```

  **Add to `post-commit` githook:**

  ```sh
  #!/bin/sh
  git update-index -g
  ```

  4. Add a "checkPretty" script to the project's package.json with the following
  contents

  ```sh
  prettier --check "<source_code_directory>/**/*.js"
  ```

  5. Ensure adherence to formatting standards by adding prettier checks to git's
  `pre-push` hook

  **Add to pre-push githook:**

  ```sh
  npm run checkPretty
  ```

## Maintaining Open Source Projects

  on each new branch/commit run:

  1. build scripts
  2. style checks
  3. tests
  4. benchmarks
  5. formatting? (should be already formatted?)
  6. publish (only if version changed)

  on each new PR:

  0. run all checks in branch/commit section
  1. contribution checks (e.g. if changelog was updated)
    - See [Setting up your project for healthy contributions](https://help.github.com/en/github/building-a-strong-community/setting-up-your-project-for-healthy-contributions)

## Documenting Open Source Projects

  When publishing open-source software, a user of the software should clearly
  understand the project's software quality from reading its documentation.
  See section ["Evaluating Software Quality"](#evaluating-software-quality).
