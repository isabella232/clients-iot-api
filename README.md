# iot-api clients generator

This repo contains the informations and the tools needed to automatically
generate API clients for the `iot-api` service in [any language][0] supported by
OpenAPI generator.

The OpenAPI generator is orchestrated by a Python tool called [apigentools][1]
that let us keep the spec files and the configuration bits in one single repo
(this one) without duplicating the boilerplate on each git repo hosting the
actual clients.

## [IMPORTANT] Client generation process

The process to generate the clients explained in paragraphs below is fully
automated through GitHub actions. The workflow will start every time a tag in the form of `vX.Y` (e.g. `v2.1`) is pushed to this repo. If the workflow completes
successfully, a PR will be opened for each client in their respective git
repositories. See the
[actions page](https://github.com/bcmi-labs/clients-iot-apiactions) to
monitor the status of a workflow.

## Sample clients

Following clients have been successfully generated with the present workflow:

* [Go](https://github.com/bcmi-labs/iot-api-client-go)
* [Python](https://github.com/bcmi-labs/iot-api-client-py)
* [Javascript](https://github.com/bcmi-labs/iot-api-client-js)

## Generation process

Apart from the initial setup (the procedures needed to setup this git repo),
the ideal workflow consists in updating this repo every time the API service
changes, re-generate all the clients, push the generated code to the git repo
it belongs to and release the updated clients (this last step may vary
depending on the programming language).

The operations are detailed in the following paragraphs.

### Requirements

To be able to run the workflow locally in a developmnent environment, you'll
need the following:

* Python 3.6+
* OpenAPI generator 4.0+ (see [install instructions](https://openapi-generator.tech/docs/installation))
* Apigentools (`pip install apigentools`)

### Get an updated version of the API specification

In this case the specs are generated by Goa using Swagger and they can be found
at [http://api2.arduino.cc/iot/swagger.json][2]. To be more future proof
and to leverage the latest versions of the tools available, we're using the
version 3 of the OpenAPI for the generator, this means that the output from
Goa must be converted into a compatible spec [like the one in this repo][3].

Several conversion tools from OpenAPI v2 to v3 exist, and some of them can be
easily integrated in a CI, so major updates will be performed by either:

* have Goa produce a v3 OpenAPI spec
* dump the Goa spec, convert to v3 and update this repo

Minor updates might be done manually since v3 uses Yaml and the resulting spec
is human friendly.

### Validate the Spec

The generator can validate the content of the spec, this should likely go in a
CI step:

```sh
apigentools validate
```

### Render upstream templates

This step patches the original (upstream) templates **before** the generation
step, so that we have full control over the generated code. The step consists
of cloning the openapi-generator repo, applying one or more patches in the form
of patch files to it and copy the relevant templates in the folder `templates`:

```sh
apigentools templates -p template-patches openapi-git
```

There are other ways to provide upstream templates other than `openapi-git` that
might be useful to speed up development iterations, please refer to `apigentools`
docs for more details.

### Generate the clients

Once templates are patched and ready to be used, this step is as easy as running:

```sh
apigentools generate
```

The previous command is fine for local development but in case the generated code
has to be pushed upstream to each client's repository, run the previous command as
follows:

```sh
 apigentools generate --clone-repo
```

### Push generated code to each client's git repository

`apigentools` has a simple command that can be invoked for each client generated
that will push the resulting code to different repos, using different branches
so the code can be reviewed and merged through a regular PR:

```sh
apigentools push
```

A release process should take from here and shouldn't be part of this workflow.

## Customization

The code generator can be fine tuned using [templates][4]. By patching existing
templates for each language or adding new ones, any aspect of the resulting client
can be adjusted to fit custom needs: the readme, the docs, the tests, the model
classes themselves can be changed downstream, and changes will reflect on the
resulting client.

[0]: https://openapi-generator.tech/docs/generators
[1]: https://github.com/DataDog/apigentools
[2]: http://api2.arduino.cc/iot/swagger.json
[3]: clients-iot-api/spec/v2/swagger.yaml
[4]: https://openapi-generator.tech/docs/templating
