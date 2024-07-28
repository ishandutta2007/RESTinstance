# RESTinstance

[Robot Framework](http://robotframework.org) library for RESTful JSON APIs

[Keyword Documentation](https://asyrjasalo.github.io/RESTinstance)

## Advantages

1.  **RESTinstance relies on Robot Framework's language-agnostic, clean
    and minimal syntax, for API tests.** It is neither tied to any
    particular programming language nor development framework. Using
    RESTinstance requires little, if any, programming knowledge. It
    builts on long-term technologies with well established communities,
    such as HTTP, JSON (Schema), Swagger/OpenAPI and Robot Framework.
2.  **It validates JSON using JSON Schema, guiding you to write API
    tests to base on properties** rather than on specific values (e.g.
    "email must be valid" vs "email is foo@bar.com"). This approach
    reduces test maintenance when the values responded by the API are
    prone to change. Although values are not required, you can still
    test them whenever they make sense (e.g. GET response body from one
    endpoint, then POST some of its values to another endpoint and
    verify the results).
3.  **It generates JSON Schema for requests and responses automatically,
    and the schema gets more accurate by your tests.** Output the schema
    to a file and reuse it as expectations to test the other methods, as
    most of them respond similarly with only minor differences. Or
    extend the schema further to a full Swagger spec (version 2.0,
    OpenAPI 3.0 also planned), which RESTinstance can test requests and
    responses against. All this leads to reusability, getting great test
    coverage with minimum number of keystrokes and very clean tests.


## Installation

You can install and upgrade
[from PyPi](https://pypi.org/project/RESTinstance):

    pip install --upgrade RESTinstance

These also install [Robot Framework](https://pypi.org/project/robotframework)
if you do not have it already.

## Usage

See [the keyword documentation](https://asyrjasalo.github.io/RESTinstance).

### Quick start

1.  Create two new empty directories, `atest` and `results`.
2.  Create a new file `atest/YOURNAME.robot` with the content:

``` robotframework
*** Settings ***
Library         REST    https://jsonplaceholder.typicode.com
Documentation   Test data can be read from variables and files.
...             Both JSON and Python type systems are supported for inputs.
...             Every request creates a so-called instance. Can be `Output`.
...             Most keywords are effective only for the last instance.
...             Initial schemas are autogenerated for request and response.
...             You can make them more detailed by using assertion keywords.
...             The assertion keywords correspond to the JSON types.
...             They take in either path to the property or a JSONPath query.
...             Using (enum) values in tests optional. Only type is required.
...             All the JSON Schema validation keywords are also supported.
...             Thus, there is no need to write any own validation logic.
...             Not a long path from schemas to full Swagger/OpenAPI specs.
...             The persistence of the created instances is the test suite.
...             Use keyword `Rest instances` to output the created instances.


*** Variables ***
${json}         { "id": 11, "name": "Gil Alexander" }
&{dict}         name=Julie Langford


*** Test Cases ***
GET an existing user, notice how the schema gets more accurate
    GET         /users/1                  # this creates a new instance
    Output schema   response body
    Object      response body             # values are fully optional
    Integer     response body id          1
    String      response body name        Leanne Graham
    [Teardown]  Output schema             # note the updated response schema

GET existing users, use JSONPath for very short but powerful queries
    GET         /users?_limit=5           # further assertions are to this
    Array       response body
    Integer     $[0].id                   1           # first id is 1
    String      $[0]..lat                 -37.3159    # any matching child
    Integer     $..id                     maximum=5   # multiple matches
    [Teardown]  Output  $[*].email        # outputs all emails as an array

POST with valid params to create a new user, can be output to a file
    POST        /users                    ${json}
    Integer     response status           201
    [Teardown]  Output  response body     ${OUTPUTDIR}/new_user.demo.json

PUT with valid params to update the existing user, values matter here
    PUT         /users/2                  { "isCoding": true }
    Boolean     response body isCoding    true
    PUT         /users/2                  { "sleep": null }
    Null        response body sleep
    PUT         /users/2                  { "pockets": "", "money": 0.02 }
    String      response body pockets     ${EMPTY}
    Number      response body money       0.02
    Missing     response body moving      # fails if property moving exists

PATCH with valid params, reusing response properties as a new payload
    &{res}=     GET   /users/3
    String      $.name                    Clementine Bauch
    PATCH       /users/4                  { "name": "${res.body['name']}" }
    String      $.name                    Clementine Bauch
    PATCH       /users/5                  ${dict}
    String      $.name                    ${dict.name}

DELETE the existing successfully, save the history of all requests
    DELETE      /users/6                  # status can be any of the below
    Integer     response status           200    202     204
    Rest instances  ${OUTPUTDIR}/all.demo.json  # all the instances so far
```

3.  Make JSON API testing great again:
```
robot --outputdir results atest/
```


## Contributing

Please create an issue and then create a pull request.


### Local development

The actual tasks are defined in `noxfile.py`:

    pipx install nox

To list all possible tasks:

    nox -l

Tasks defined in `RESTinstance/noxfile.py`:

    * test -> Run development tests for the package.
    - testenv -> Run development server for acceptance tests.
    * atest -> Run acceptance tests for the project.
    - docs -> Regenerate documentation for the project.
    - black -> Reformat/unify/"blacken" Python source code in-place.
    - prospector -> Run various static analysis tools for the package.
    - build -> Build sdist and wheel dists.
    - release_testpypi -> Publish dist/* to TestPyPI.
    - install_testpypi -> Install the latest (pre-)release from TestPyPI.
    - release -> Tag, build and publish a new release to PyPI.
    - install -> Install the latest release from PyPI.
    - clean -> Remove all .venv's, build files and caches in the directory.

Tasks marked with `*` are selected by default.

That is, to run both `test`s and `atest`s:

    nox

Session `nox -s atest` assumes you have started `testapi/` on
[mountebank](https://www.mbtest.org):

    nox -s testenv

After started, you can debug requests and responses by tests in web browser at
[localhost:2525](http://localhost:2525/imposters).

Both `nox -s test` and `nox -s atest` allow passing arguments to `pytest`
and `robot`, respectively:

    nox -s test -- test/<test_modulename>.py
    nox -s atest -- atest/<atest_suitedir>/<atest_suitefile>.robot

Update documentation:

    nox -s docs

### Building and tagging a new version

Remove all sessions (`.venv/`s) as well as temporary files in your working copy:

    nox -s clean

Build:

    nox -s clean build

We use [zest.releaser](https://github.com/zestsoftware/zest.releaser) for
versioning, tagging and building (universal) `bdist_wheel`s.

It uses [twine](https://pypi.org/project/twine/) underneath to upload to PyPIs
securely over HTTPS, which can't be done with `python setup.py` commands.

### Releasing to PyPIs

This workflow is preferred for distributing a new (pre-)release to TestPyPI:

    nox -s test atest docs clean build release_testpypi install_testpypi

If that installed well, all will be fine to let the final release to PyPI:

    nox -s release

To install the latest release from PyPI, and in a dedicated venv of course:

    nox -s install

### pre-commit hooks

We want our static analysis checks ran before code even ends up in a commit.

Thus both `nox` and `nox -s test` commands bootstrap
[pre-commit](https://pre-commit.com/) hooks in your git working copy.

The actual hooks are configured in `.pre-commit-commit.yaml`.


## Credits

RESTinstance is under
[LGPL-3.0 license](https://github.com/asyrjasalo/RESTinstance/blob/master/LICENSE.txt)
and is originally written by [Anssi Syrjäsalo](https://github.com/asyrjasalo).

It was first presented at the first [RoboCon](https://robocon.io), 2018.

Contributors:

- Eficode Ltd.

We use following Python excellence under the hood:

- [Flex](https://github.com/pipermerriam/flex), by Piper Merriam, for
Swagger 2.0 validation
- [GenSON](https://github.com/wolverdude/GenSON), by Jon "wolverdude"
Wolverton, for JSON Schema generator
- [jsonpath-ng](https://github.com/h2non/jsonpath-ng), by Tomas
Aparicio and Kenneth Knowles, for handling JSONPath queries
- [jsonschema](https://github.com/Julian/jsonschema), by Julian
Berman, for JSON Schema validator
- [pygments](http://pygments.org), by Georg Brandl et al., for JSON
syntax coloring, in terminal <span class="title-ref">Output</span>
- [requests](https://github.com/requests/requests), by Kenneth Reitz
et al., for making HTTP requests

See
[requirements.txt](https://github.com/asyrjasalo/RESTinstance/blob/master/requirements.txt)
for all the direct run time dependencies.
