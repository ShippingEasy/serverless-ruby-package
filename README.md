# serverless-ruby-package

Teaches the [serverless framework](https://serverless.com/framework/) how to
package ruby services and their gem dependencies.

By default, `serverless` will package _all_ files in your ruby service project.
That includes test files, `node_modules`, readmes, etc. You don't need any of
those files to execute your function - they just increase the size of the package
and slow down updates. In most cases, it also won't include any rubygems that
your function depends on, so the function will not work once it is deployed.

This plugin solves those problems in a couple ways. First, it excludes _all_
files by default, forcing you to whitelist the files needed by your function
(using the `package/includes` key in `serverless.yml`). Second, it automatically
packages the gems from your default bundler group -- skipping gems from
any custom groups, like `development` or `test`. Sometimes gems themselves will
install their own test files - those will also be excluded from the package.


## Usage

The plugin will make it easier to work with rubygems in your serverless project,
but it requires your project to follow some conventions:

1) Add the plugin

```
npm install --save-dev serverless-ruby-package
```

Add the plugin to your `serverless.yml` file:

```yaml
plugins:
  - serverless-ruby-package
```

2) You must use [bundler](https://bundler.io/) in _standalone_ mode, with the path `vendor/bundle`:

```
bundle install --standalone --path vendor/bundle
```

3) Add the following lines to the top of your service handler file:

```ruby
load "vendor/bundle/bundler/setup.rb"
```

4) The plugin will force the `serverless` packaging step to exclude _all_ files
by default. You need to explicitly add your service handler file to `serverless.yml`:

```yaml
package:
  excludeDevDependencies: false # only applies to node
  include:
    - handler.rb
```

(optional) If your service is implemented across multiple ruby files, it is
recommended that you keep your handler file in the root, and the rest of the
files in a `lib/` directory. You can make them available to your handler by
adding the following line to the top of your handler:

```ruby
# handler.rb
load "vendor/bundle/bundler/setup.rb"
$LOAD_PATH.unshift(File.expand_path("lib", __dir__))
```

You will also need to edit your `serverless.yml` `package` settings to include
those files:

```yaml
package:
  include:
    - handler.rb
    - lib/**
```

## Verify

Build your package to confirm it is being built as expected:

```
serverless package
```

That will create the package that is uploaded during a `serverless deploy`, but
keep it around so you can examine it. You can use the following command to verify
it only includes the files you expect:

```
unzip -l .serverless/<servicename>.zip
```


## Development

To work on this plugin, you should first run the following in your local directory:

```
./dev_setup.sh          # configures the demo service project needed by the tests

yarn test               # run the automated test suite

./integration_test.sh   # run serverless package on the demo project and observe the results
```
