# pyproject.toml

Since `pyproject.toml` files are commonly used by python packages it can be desirable to create a lock
file directly from those dependencies to single-source a package's dependencies.

This makes use of some conda-forge infrastructure ([pypi-mapping][mapping]) to do a lookup of the PyPI
package name to a corresponding conda package name (e.g. `docker` -> `docker-py`).  In cases where there
is no lookup for the package it assumes that the PyPI name, and the conda name are the same.

## Features

### dependency resolution

=== "poetry"

    ```toml
    # pyproject.toml

    [tool.poetry.dependencies]
    requests = "^2.13.0"
    toml = ">=0.10"

    [tool.poetry.dev-dependencies]
    pytest = ">=5.1.0"

    [build-system]
    requires = ["poetry>=0.12"]
    build-backend = "poetry.masonry.api"
    ```

=== "pep621 (flit)"
    ```toml
    # pyproject.toml

    [project]
    dependencies = [
        "requests ^2.13.0",
        "toml >=0.10",
    ]
    [project.optional-dependencies]
    test = [
        "pytest >=5.1.0",
    ]
    ```

This will create a conda-lock specification with

**main**

    requests ^2.13.0"
    toml >=0.10

**dev**

    pytest >=5.1.0

### pure pip dependencies

If a dependency refers directly to a URL rather than a package name and version,
`conda-lock` will assume it is pip-installable, e.g.:

```toml
# pyproject.toml

[tool.poetry.dependencies]
python = "3.9"
pymage = {url = "https://github.com/MickaelRigault/pymage/archive/v1.0.tar.gz#sha256=11e99c4ea06b76ca7fb5b42d1d35d64139a4fa6f7f163a2f0f9cc3ea0b3c55eb"}
```

Similarly, if a dependency is explicitly marked with `source = "pypi"`, it will
be treated as a `pip` dependency, e.g.:

```toml
# pyproject.toml

[tool.poetry.dependencies]
python = "3.9"
ampel-ztf = {version = "^0.8.0-alpha.2", source = "pypi"}
```

In both these cases, the dependencies of `pip`-installable packages will also be
installed with `pip`, unless they were already requested by a `conda`
dependency.

### Extras

If your pyproject.toml file contains optional dependencies/extras these can be referred to by using the `--extras` flag

=== "poetry"

    ```toml
    # pyproject.toml

    [tool.poetry.dependencies]
    mandatory = "^1.0"
    psycopg2 = { version = "^2.7", optional = true }
    mysqlclient = { version = "^1.3", optional = true }

    [tool.poetry.extras]
    mysql = ["mysqlclient"]
    pgsql = ["psycopg2"]
    ```

=== "pep621 (flit)"

    ```toml
    # pyproject.toml

    [project]
    dependencies = [
        "mandatory ^1.0",
    ]

    [project.optional-dependencies]
    mysql = ["mysqlclient ^1.3"]
    pgsql = ["psycopg2 ^2.7"]
    ```


These can be referened as follows

```
conda-lock --extra mysql --extra pgsql -f pyproject.toml
```

When generating lockfiles that make use of extras it is recommended to make use of `--filename-template` covered [here](#file-naming).


## Extensions

As the `pyproject.toml` format is not really designed for conda there are a few extensions we support in the
toml file.  All extensions live in the `tool.conda-lock` section.

### Channels

```toml
# pyproject.toml

[tool.conda-lock]
channels = [
    'conda-forge', 'defaults'
]
```

### Platforms

Like in [environment.yml](/src_environment_yml#platform-specification), you can specify default platforms to target:

```toml
# pyproject.toml

[tool.conda-lock]
platforms = [
    'osx-arm64', 'linux-64'
]
```

### Extra conda dependencies

Since in a `pyproject.toml` all the definitions are python dependencies if you need
to specify some non-python dependencies as well this can be accomplished by adding
the following sections to the `pyproject.toml`

```toml
# pyproject.toml

[tool.conda-lock.dependencies]
sqlite = ">=3.34"
```


[mapping]: https://github.com/regro/cf-graph-countyfair/blob/master/mappings/pypi/grayskull_pypi_mapping.yaml