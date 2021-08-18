# py-vercel [![NPM version](https://img.shields.io/npm/v/@potatohd/py-vercel.svg)](https://www.npmjs.com/package/@potatohd/py-vercel) [![License](https://img.shields.io/npm/l/@PotatoHD404/py-vercel)](https://github.com/PotatoHD404/py-vercel/blob/dev/LICENSE.md)

## *A Vercel builder for Python WSGI applications*

## Quickstart

If you have an existing WSGI app, getting this builder to work for you is a
piece of 🍰!

### 1. Add a Vercel configuration

Add a `vercel.json` file to the root of your application:

```json
{
    "builds": [{
        "src": "index.py",
        "use": "@potatohd/py-vercel",
        "config": { "maxLambdaSize": "15mb" }
    }]
}
```

This configuration is doing a few things in the `"builds"` part:

1. `"src": "index.py"`
   This tells Now that there is one entrypoint to build for. `index.py` is a
   file we'll create shortly.
2. `"use": "@potatohd/py-vercel"`
   Tell Now to use this builder when deploying your application
3. `"config": { "maxLambdaSize": "15mb" }`
   Bump up the maximum size of the built application to accommodate some larger
   python WSGI libraries (like Django or Flask). This may not be necessary for
   you.

### 2. Add a Now entrypoint

Add `index.py` to the root of your application. This entrypoint should make
available an object named `application` that is an instance of your WSGI
application. E.g.:

```python
# For a Dango app
from django_app.wsgi import application
# Replace `django_app` with the appropriate name to point towards your project's
# wsgi.py file
```

Look at your framework documentation for help getting access to the WSGI
application.

If the WSGI instance isn't named `application` you can set the
`wsgiApplicationName` configuration option to match your application's name (see
the configuration section below).

### 3. Deploy

That's it, you're ready to go:

```console
$ vercel
> Deploying python-wsgi-app
...
> Success! Deployment ready [57s]
```

## Requirements

### Linux requirements

Your project may optionally include a `setup.sh` or a `post-install.sh` file to declare any
dependencies, e.g.:

```sh
# setup.sh
yum install -y gcc mysql-devel

ln -s /usr/lib64/libmariadbclient.a /usr/lib64/libmariadb.a

pip install -t $srcDir $srcDir/django-storages
```

(`setup.sh` runs before python packages installation and `post-install.sh` after)

As you can see, there is default variable `$srcDir` which contains
path to your project folder. Also, yoy may notice that the server is on
special Amazon Linux 2 runtime. So keep that in mind when you are writing scripts.

Also be aware that on the runtime you must use the `-t` argument when installing
python packages.

### Python requirements

Your project may optionally include a `requirements.txt` file to declare any
dependencies, e.g.:

```text
# requirements.txt
Django >=2.2,<2.3
```

Be aware that the builder will install `Werkzeug` as a requirement of the
handler. This can cause issues if your project requires a different version of
`Werkzeug` than the handler.

## Configuration options

### `runtime`

Select the lambda runtime. Defaults to `python3.8`.

```json
{
    "builds": [{
        "config": { "runtime": "python3.8" }
    }]
}
```

### `wsgiApplicationName`

Select the WSGI application to run from your entrypoint. Defaults to
`application`.

```json
{
    "builds": [{
        "config": { "wsgiApplicationName": "application" }
    }]
}
```


### `production`

Select whether the application is in production mode or not. When set to false,
the debugger will appear on error. Defaults to
false.

```json
{
    "builds": [{
        "config": { "production": true }
    }]
}
```

## Additional considerations

### Routing

You'll likely want all requests arriving at your deployment url to be routed to
your application. You can do this by adding a route rewrite to the Now
configuration:

```json
{
    "builds": [{
        "src": "index.py",
        "use": "@potatohd/py-vercel"
    }],
    "routes" : [{
        "src" : "/(.*)", "dest":"/"
    }]
}
```

### Avoiding the `index.py` file

If having an extra file in your project is troublesome or seems unecessary, it's
also possible to configure Now to use your application directly, without passing
it through `index.py`.

If your WSGI application lives in `vercel_app/wsgi.py` and is named `application`,
then you can configure it as the entrypoint and adjust routes accordingly:

```json
{
    "builds": [{
        "src": "vercel_app/wsgi.py",
        "use": "@potatohd/py-vercel"
    }],
    "routes" : [{
        "src" : "/(.*)", "dest":"/vercel_app/wsgi.py"
    }]
}
```

### Lambda environment limitations

At the time of writing, Vercel runs on AWS Lambda. This has a number of
implications on what libraries will be available to you, notably:

- PostgreSQL, so psycopg2 won't work out of the box
- MySQL, so MySQL adapters won't work out of the box either
- Sqlite, so the built-in Sqlite adapter won't be available

## Attribution

This implementation draws upon work from:

- [vercel-python-wsgi](https://github.com/jayhale/vercel-python-wsgi) by
   [@jayhale](https://github.com/jayhale)
