# How to add an ament task

Adding an ament linter to the vscode-ament-task-provider repository involves updating adding the linter to available tasks, and adding a problem matcher.

## Add to available tasks

### Update src/amentTaskProvider.ts:

This file contains a function that retrieves the available ament tasks. You need to add the new linter to the linters array. For example, if you want to add a linter called my_linter, add the following line to the function:

```ts
const linters: string[] = ['cpplint', 'cppcheck', 'lint_cmake', 'flake8', 'pep257', 'xmllint', 'my_linter'];
```

### Update package.json

This file contains a section that defines the ament task definitions. You need to add the new linter to the `taskDefinitions` array. For example, if you want to add a linter called my_linter, add the following JSON object to the array:

```jsonc
    "contributes": {
        "taskDefinitions": [
            {
                "type": "ament",
                "required": [
                    "task"
                ],
                "properties": {
                    "task": {
                        "type": "string",
                        "description": "The ament linter",
                        "examples": [
                            "cpplint",
                            "cppcheck",
                            "lint_cmake",
                            "flake8",
                            "pep257",
                            "xmllint",
                            "my_linter"
                        ]
                    },
```

## Add problem matcher

### Update package.json

This file contains a section that defines how to parse the output of the linter and identify errors. You need to add a new problem matcher for the new linter.

Here's an example of a problem matcher for a linter called my_linter:

```jsonc
{
  "name": "ament_my_linter",
  "label": "Ament my_linter problem matcher",
  "owner": "ament",
  "source": "ament_my_linter",
  "applyTo": "allDocuments",
  "fileLocation": "absolute",
  "severity": "error",
  "pattern": [
    {
      "regexp": "^(.+):(\\d+):\\s+(.+)\\[(.+)\\]$",
      "file": 1,
      "line": 2,
      "message": 3,
      "code": 4
    }
  ]
}
```

You can follow [this guide](https://www.allisonthackston.com/articles/vscode-tasks-problemmatcher.html) on how to set up a problem matcher

### Update tests

Finally, you need to add some unit tests to make sure the new linter is working correctly. You need to add a new test file or modify an existing one to include tests for the new linter.

Tests check that the problem matcher definition matches the output of the linter. To automate this, there is a generation script and helper functions.

#### update tests/data

Create two new files in the `tests/data` directory, one for a passing lint check and one for a failing lint check. The filenames should be in the format `<linter>_pass.<extension>` and `<linter>_fail.<extension>`, where <linter> is the name of the linter being added and <extension> is the file extension used by the linter (e.g. cpp for C++ files).

#### Update tests/data/gen.sh

Update the `gen.sh` script in the `tests/data` directory to generate the new lint check files. Add a new section for the new linter, and call the fixture_gen function with the appropriate arguments.

```sh
# generate cpplint fixtures
echo "generating cpplint"
fixture_gen ament_cpplint cpplint_pass.cpp cpplint_pass.txt
fixture_gen ament_cpplint cpplint_fail.cpp cpplint_fail.txt

# generate new linter fixtures
echo "generating new_linter"
fixture_gen ament_new_linter new_linter_pass.<extension> new_linter_pass.txt
fixture_gen ament_new_linter new_linter_fail.<extension> new_linter_fail.txt
```

#### Add to fixtures

Add the new linter to the fixtures object in tests/fixtures.ts. This object contains the paths to the passing and failing lint check files for each linter.

```ts
const cpplintOutputSuccess = syncReadFile('./data/cpplint_ok.txt');
const cpplintOutputFail = syncReadFile('./data/cpplint_fail.txt');
// add new linter here
```

```ts
export const fixtures = {
    cpplintOutputSuccess,
    cpplintOutputFail,
    // add new linter here
```

#### Add to problem-matcher.tests.ts

Update the `problem-matcher.test.ts` file to include tests for the new linter.

```ts
describe('ament_new_linter problemMatcher', () => {
  const matcherName = 'ament_new_linter';
  const matcherDef = () => findProblemMatcher(matcherName);

  it('exists in package.json', () => {
    expect(matcherDef(), `problemMatcher with name ${matcherName}`).to.be.ok;
  });

  describe('given ament_new_linter output with a failure', () => {
    const lines = () => blobToLines(fixtures.new_linter.fail);

    it('has a sequence matching problemMatcher.pattern sequence', () => {
      expect(lines()).to.haveAnEntry.matchFirstRegexpOf(matcherDef().pattern);
    });
  });

  describe('given ament_new_linter output with no failures', () => {
    const lines = () => blobToLines(fixtures.new_linter.pass);

    it('does not have a sequence matching problemMatcher.pattern', () => {
      expect(lines()).to.not.haveAnEntry.matchFirstRegexpOf(matcherDef().pattern);
    });
  });
});
```
