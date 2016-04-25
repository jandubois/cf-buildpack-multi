# cf-multi-buildpack #

This repo contains a proof-of-concept fork of the [Heroku multi buildpack](https://github.com/heroku/heroku-buildpack-multi) with additional support to delegate to CF admin buildpacks.

Admin buildpacks are mapped into the staging container, but it isn't possible to locate them (by name). The cloud controller jealousy guards the mapping between buildpack name and key/guid.

This buildpack assumes that the admin buildpack locations will be exposed via an environment variable:

```json
$ echo $CF_BUILDPACKS | jq
[
  {
    "name": "multi_buildpack",
    "path": "/tmp/buildpacks/9b70be7b15e98cd140611ca4aef0e22b"
  },
  {
    "name": "staticfile_buildpack",
    "path": "/tmp/buildpacks/50c096378c1d0f231c496280a4c6c2c5"
  },
  {
    "name": "ruby_buildpack",
    "path": "/tmp/buildpacks/d0e98d8a90f127f3a5d7dab0ea069b55"
  },
  {
    "name": "nodejs_buildpack",
    "path": "/tmp/buildpacks/236f3e3113e8a6667a82f1f5c52b41cd"
  },
  {
    "name": "go_buildpack",
    "path": "/tmp/buildpacks/3b68bef6015ac2e3074a8abf6d0ed97c"
  },
  {
    "name": "python_buildpack",
    "path": "/tmp/buildpacks/339046ccef25c47851c0040c270ab5c7"
  },
  {
    "name": "php_buildpack",
    "path": "/tmp/buildpacks/9bf25f7222afb2d4f98163f640b1ddb4"
  },
  {
    "name": "binary_buildpack",
    "path": "/tmp/buildpacks/1856820bac3215acac7e4d5f4a6f8fba"
  },
  {
    "name": "cf_iis_buildpack",
    "path": "/tmp/buildpacks/c1da16a409729293bb429c882028491b"
  }
]
```

Since this is currently not implemented, this buildpack provides a small helper script [cf_buildpacks](https://github.com/jandubois/cf-buildpack-multi/blob/master/cf_buildpacks) that reads the settings directly from the CCDB.

Staging then needs to happen via multiple commands:

```bash
cf push myapp --no-start
cf set-env myapp CF_BUILDPACKS $(cf_buildpacks)
cf start myapp
```

The required change to the buildpack is quite small if you [ignore whitespace differences](https://github.com/jandubois/cf-buildpack-multi/commit/d24dbc0?w=1).

If `cf-buildpack-multi` is deployed as an admin buildpack itself, then it should be moved to the top position to make sure it is detected first.

## Sample deployment ##

```bash
$ cat .buildpacks
nodejs_buildpack
php_buildpack

$ cf push multi --no-start
[...]

$ cf set-env multi CF_BUILDPACKS "$(./cf_buildpacks)"
Setting env variable 'CF_BUILDPACKS' to '[{"name":"multi_buildpack","path":"/tmp/buildpacks/9b70be7b15e98cd140611ca4aef0e22b"},{"name":"staticfile_buildpack","path":"/tmp/buildpacks/50c096378c1d0f231c496280a4c6c2c5"},{"name":"ruby_buildpack","path":"/tmp/buildpacks/d0e98d8a90f127f3a5d7dab0ea069b55"},{"name":"nodejs_buildpack","path":"/tmp/buildpacks/236f3e3113e8a6667a82f1f5c52b41cd"},{"name":"go_buildpack","path":"/tmp/buildpacks/3b68bef6015ac2e3074a8abf6d0ed97c"},{"name":"python_buildpack","path":"/tmp/buildpacks/339046ccef25c47851c0040c270ab5c7"},{"name":"php_buildpack","path":"/tmp/buildpacks/9bf25f7222afb2d4f98163f640b1ddb4"},{"name":"binary_buildpack","path":"/tmp/buildpacks/1856820bac3215acac7e4d5f4a6f8fba"},{"name":"cf_iis_buildpack","path":"/tmp/buildpacks/c1da16a409729293bb429c882028491b"}]' for app multi in org hpe / space myspace as admin...OK
TIP: Use 'cf restage' to ensure your env variable changes take effect

$ cf start multi
Starting app multi in org hpe / space myspace as admin...
[...]
Staging...
=====> Locating Admin Buildpack: nodejs_buildpack
=====> Detected Framework: node.js 1.5.8
-------> Buildpack version 1.5.8
[...]
=====> Locating Admin Buildpack: php_buildpack
=====> Detected Framework: php 4.3.7
[...]
Using release configuration from last framework (php 4.3.7).
Exit status 0
Staging complete
```

## Original Heroku README ##

This a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) that
allows one to multiple other buildpacks in a single deploy process. This helps support:

1. Running multiple language buildpacks such as JS for assets and Ruby for your application
2. Running a daemon process such as [pgbouncer](https://github.com/heroku/heroku-buildpack-pgbouncer) with your application
3. Pulling in system dependencies.

### Usage ###

To use this buildpack you'll first need to set it as your custom buildpack:

    $ heroku buildpacks:set https://github.com/heroku/heroku-buildpack-multi.git

From here you will need to create a `.buildpacks` file which contains (in order) the buildpacks you wish to run when you deploy:

    $ cat .buildpacks
    https://github.com/heroku/heroku-buildpack-nodejs.git#0198c71daa8
    https://github.com/heroku/heroku-buildpack-ruby.git#v86

### License ###

MIT

### FAQ ###

