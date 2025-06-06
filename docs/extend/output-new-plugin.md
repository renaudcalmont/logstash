---
mapped_pages:
  - https://www.elastic.co/guide/en/logstash/current/output-new-plugin.html
---

# How to write a Logstash output plugin [output-new-plugin]

To develop a new output for Logstash, build a self-contained Ruby gem whose source code lives in its own GitHub repository. The Ruby gem can then be hosted and shared on RubyGems.org. You can use the example output implementation as a starting point. (If you’re unfamiliar with Ruby, you can find an excellent quickstart guide at [https://www.ruby-lang.org/en/documentation/quickstart/](https://www.ruby-lang.org/en/documentation/quickstart/).)

## Get started [_get_started_4]

Let’s step through creating an output plugin using the [example output plugin](https://github.com/logstash-plugins/logstash-output-example/).

### Create a GitHub repo for your new plugin [_create_a_github_repo_for_your_new_plugin_4]

Each Logstash plugin lives in its own GitHub repository. To create a new repository for your plugin:

1. Log in to GitHub.
2. Click the **Repositories** tab. You’ll see a list of other repositories you’ve forked or contributed to.
3. Click the green **New** button in the upper right.
4. Specify the following settings for your new repo:

    * **Repository name** — a unique name of the form `logstash-output-pluginname`.
    * **Public or Private** — your choice, but the repository must be Public if you want to submit it as an official plugin.
    * **Initialize this repository with a README** — enables you to immediately clone the repository to your computer.

5. Click **Create Repository**.


### Use the plugin generator tool [_use_the_plugin_generator_tool_4]

You can create your own Logstash plugin in seconds! The `generate` subcommand of `bin/logstash-plugin` creates the foundation for a new Logstash plugin with templatized files. It creates the correct directory structure, gemspec files, and dependencies so you can start adding custom code to process data with Logstash.

For more information, see [Generating plugins](/reference/plugin-generator.md)


### Copy the output code [_copy_the_output_code]

Alternatively, you can use the examples repo we host on github.com

1. **Clone your plugin.** Replace `GITUSERNAME` with your github username, and `MYPLUGINNAME` with your plugin name.

    * `git clone https://github.com/GITUSERNAME/logstash-``output-MYPLUGINNAME.git`

        * alternately, via ssh: `git clone git@github.com:GITUSERNAME/logstash``-output-MYPLUGINNAME.git`

    * `cd logstash-output-MYPLUGINNAME`

2. **Clone the output plugin example and copy it to your plugin branch.**

    You don’t want to include the example .git directory or its contents, so delete it before you copy the example.

    * `cd /tmp`
    * `git clone https://github.com/logstash-plugins/logstash``-output-example.git`
    * `cd logstash-output-example`
    * `rm -rf .git`
    * `cp -R * /path/to/logstash-output-mypluginname/`

3. **Rename the following files to match the name of your plugin.**

    * `logstash-output-example.gemspec`
    * `example.rb`
    * `example_spec.rb`

        ```txt
        cd /path/to/logstash-output-mypluginname
        mv logstash-output-example.gemspec logstash-output-mypluginname.gemspec
        mv lib/logstash/outputs/example.rb lib/logstash/outputs/mypluginname.rb
        mv spec/outputs/example_spec.rb spec/outputs/mypluginname_spec.rb
        ```


Your file structure should look like this:

```txt
$ tree logstash-output-mypluginname
├── Gemfile
├── LICENSE
├── README.md
├── Rakefile
├── lib
│   └── logstash
│       └── outputs
│           └── mypluginname.rb
├── logstash-output-mypluginname.gemspec
└── spec
    └── outputs
        └── mypluginname_spec.rb
```

For more information about the Ruby gem file structure and an excellent walkthrough of the Ruby gem creation process, see [http://timelessrepo.com/making-ruby-gems](http://timelessrepo.com/making-ruby-gems)


### See what your plugin looks like [_see_what_your_plugin_looks_like_4]

Before we dive into the details, open up the plugin file in your favorite text editor and take a look.

```ruby
require "logstash/outputs/base"
require "logstash/namespace"

# Add any asciidoc formatted documentation here
# An example output that does nothing.
class LogStash::Outputs::Example < LogStash::Outputs::Base
  config_name "example"

  # This sets the concurrency behavior of this plugin. By default it is :legacy, which was the standard
  # way concurrency worked before Logstash 2.4
  #
  # You should explicitly set it to either :single or :shared as :legacy will be removed in Logstash 6.0
  #
  # When configured as :single a single instance of the Output will be shared among the
  # pipeline worker threads. Access to the `#multi_receive/#multi_receive_encoded/#receive` method will be synchronized
  # i.e. only one thread will be active at a time making threadsafety much simpler.
  #
  # You can set this to :shared if your output is threadsafe. This will maximize
  # concurrency but you will need to make appropriate uses of mutexes in `#multi_receive/#receive`.
  #
  # Only the `#multi_receive/#multi_receive_encoded` methods need to actually be threadsafe, the other methods
  # will only be executed in a single thread
  concurrency :single

  public
  def register
  end # def register

  public
  # Takes an array of events
  # Must be threadsafe if `concurrency :shared` is set
  def multi_receive(events)
  end # def multi_receive
end # class LogStash::Outputs::Example
```



## Coding output plugins [_coding_output_plugins]

Now let’s take a line-by-line look at the example plugin.

### `require` Statements [_require_statements_4]

Logstash output plugins require parent classes defined in `logstash/outputs/base` and logstash/namespace:

```ruby
require "logstash/outputs/base"
require "logstash/namespace"
```

Of course, the plugin you build may depend on other code, or even gems. Just put them here along with these Logstash dependencies.



## Plugin Body [_plugin_body_4]

Let’s go through the various elements of the plugin itself.

### `class` Declaration [_class_declaration_4]

The output plugin class should be a subclass of `LogStash::Outputs::Base`:

```ruby
class LogStash::Outputs::Example < LogStash::Outputs::Base
```

The class name should closely mirror the plugin name, for example:

```ruby
LogStash::Outputs::Example
```


### `config_name` [_config_name_4]

```ruby
  config_name "example"
```

This is the name your plugin will call inside the output configuration block.

If you set `config_name "example"` in your plugin code, the corresponding Logstash configuration block would need to look like this:



## Configuration Parameters [_configuration_parameters_4]

```ruby
  config :variable_name, :validate => :variable_type, :default => "Default value", :required => boolean, :deprecated => boolean, :obsolete => string
```

The configuration, or `config` section allows you to define as many (or as few) parameters as are needed to enable Logstash to process events.

There are several configuration attributes:

* `:validate` - allows you to enforce passing a particular data type to Logstash for this configuration option, such as `:string`, `:password`, `:boolean`, `:number`, `:array`, `:hash`, `:path` (a file-system path), `uri`, `:codec` (since 1.2.0), `:bytes`.  Note that this also works as a coercion in that if I specify "true" for boolean (even though technically a string), it will become a valid boolean in the config.  This coercion works for the `:number` type as well where "1.2" becomes a float and "22" is an integer.
* `:default` - lets you specify a default value for a parameter
* `:required` - whether or not this parameter is mandatory (a Boolean `true` or
* `:list` - whether or not this value should be a list of values. Will typecheck the list members, and convert scalars to one element lists. Note that this mostly obviates the array type, though if you need lists of complex objects that will be more suitable. `false`)
* `:deprecated` - informational (also a Boolean `true` or `false`)
* `:obsolete` - used to declare that a given setting has been removed and is no longer functioning. The idea is to provide an informed upgrade path to users who are still using a now-removed setting.


## Plugin Methods [_plugin_methods_4]

Logstash outputs must implement the `register` and `multi_receive` methods.

### `register` Method [_register_method_4]

```ruby
  public
  def register
  end # def register
```

The Logstash `register` method is like an `initialize` method. It was originally created to enforce having `super` called, preventing headaches for newbies. (Note: It may go away in favor of `initialize`, in conjunction with some enforced testing to ensure `super` is called.)

`public` means the method can be called anywhere, not just within the class. This is the default behavior for methods in Ruby, but it is specified explicitly here anyway.

You can also assign instance variables here (variables prepended by `@`). Configuration variables are now in scope as instance variables, like `@message`



## Building the Plugin [_building_the_plugin_4]

At this point in the process you have coded your plugin and are ready to build a Ruby Gem from it. The following information will help you complete the process.

### External dependencies [_external_dependencies_4]

A `require` statement in Ruby is used to include necessary code. In some cases your plugin may require additional files.  For example, the collectd plugin [uses](https://github.com/logstash-plugins/logstash-codec-collectd/blob/main/lib/logstash/codecs/collectd.rb#L148) the `types.db` file provided by collectd.  In the main directory of your plugin, a file called `vendor.json` is where these files are described.

The `vendor.json` file contains an array of JSON objects, each describing a file dependency. This example comes from the [collectd](https://github.com/logstash-plugins/logstash-codec-collectd/blob/main/vendor.json) codec plugin:

```txt
[{
        "sha1": "a90fe6cc53b76b7bdd56dc57950d90787cb9c96e",
        "url": "http://collectd.org/files/collectd-5.4.0.tar.gz",
        "files": [ "/src/types.db" ]
}]
```

* `sha1` is the sha1 signature used to verify the integrity of the file referenced by `url`.
* `url` is the address from where Logstash will download the file.
* `files` is an optional array of files to extract from the downloaded file. Note that while tar archives can use absolute or relative paths, treat them as absolute in this array.  If `files` is not present, all files will be uncompressed and extracted into the vendor directory.

Another example of the `vendor.json` file is the [`geoip` filter](https://github.com/logstash-plugins/logstash-filter-geoip/blob/main/vendor.json)

The process used to download these dependencies is to call `rake vendor`.  This will be discussed further in the testing section of this document.

Another kind of external dependency is on jar files.  This will be described in the "Add a `gemspec` file" section.


### Deprecated features [_deprecated_features_4]

As a plugin evolves, an option or feature may no longer serve the intended purpose, and the developer may want to *deprecate* its usage. Deprecation warns users about the option’s status, so they aren’t caught by surprise when it is removed in a later release.

{{ls}} 7.6 introduced a *deprecation logger* to make handling those situations easier. You can use the [adapter](https://github.com/logstash-plugins/logstash-mixin-deprecation_logger_support) to ensure that your plugin can use the deprecation logger while still supporting older versions of {{ls}}. See the [readme](https://github.com/logstash-plugins/logstash-mixin-deprecation_logger_support/blob/main/README.md) for more information and for instructions on using the adapter.

Deprecations are noted in the `logstash-deprecation.log` file in the `log` directory.


### Add a Gemfile [_add_a_gemfile_4]

Gemfiles allow Ruby’s Bundler to maintain the dependencies for your plugin. Currently, all we’ll need is the Logstash gem, for testing, but if you require other gems, you should add them in here.

::::{tip}
See [Bundler’s Gemfile page](http://bundler.io/gemfile.html) for more details.
::::


```ruby
source 'https://rubygems.org'
gemspec
gem "logstash", :github => "elastic/logstash", :branch => "master"
```



## Add a `gemspec` file [_add_a_gemspec_file_4]

Gemspecs define the Ruby gem which will be built and contain your plugin.

::::{tip}
More information can be found on the [Rubygems Specification page](http://guides.rubygems.org/specification-reference/).
::::


```ruby
Gem::Specification.new do |s|
  s.name = 'logstash-output-example'
  s.version = '0.1.0'
  s.licenses = ['Apache License (2.0)']
  s.summary = "This output does x, y, z in Logstash"
  s.description = "This gem is a logstash plugin required to be installed on top of the Logstash core pipeline using $LS_HOME/bin/logstash-plugin install gemname. This gem is not a stand-alone program"
  s.authors = ["Elastic"]
  s.email = 'info@elastic.co'
  s.homepage = "http://www.elastic.co/guide/en/logstash/current/index.html"
  s.require_paths = ["lib"]

  # Files
  s.files = Dir['lib/**/*','spec/**/*','vendor/**/*','*.gemspec','*.md','CONTRIBUTORS','Gemfile','LICENSE','NOTICE.TXT']
   # Tests
  s.test_files = s.files.grep(%r{^(test|spec|features)/})

  # Special flag to let us know this is actually a logstash plugin
  s.metadata = { "logstash_plugin" => "true", "logstash_group" => "output" }

  # Gem dependencies
  s.add_runtime_dependency "logstash-core-plugin-api", ">= 1.60", "<= 2.99"
  s.add_development_dependency 'logstash-devutils'
end
```

It is appropriate to change these values to fit your plugin. In particular, `s.name` and `s.summary` should reflect your plugin’s name and behavior.

`s.licenses` and `s.version` are also important and will come into play when you are ready to publish your plugin.

Logstash and all its plugins are licensed under [Apache License, version 2 ("ALv2")](https://github.com/elastic/logstash/blob/main/LICENSE.txt). If you make your plugin publicly available via [RubyGems.org](http://rubygems.org), please make sure to have this line in your gemspec:

* `s.licenses = ['Apache License (2.0)']`

The gem version, designated by `s.version`, helps track changes to plugins over time. You should use [semver versioning](http://semver.org/) strategy for version numbers.

### Runtime and Development Dependencies [_runtime_and_development_dependencies_4]

At the bottom of the `gemspec` file is a section with a comment: `Gem dependencies`.  This is where any other needed gems must be mentioned. If a gem is necessary for your plugin to function, it is a runtime dependency. If a gem are only used for testing, then it would be a development dependency.

::::{note}
You can also have versioning requirements for your dependencies—​including other Logstash plugins:

```ruby
  # Gem dependencies
  s.add_runtime_dependency "logstash-core-plugin-api", ">= 1.60", "<= 2.99"
  s.add_development_dependency 'logstash-devutils'
```

This gemspec has a runtime dependency on the logstash-core-plugin-api and requires that it have a version number greater than or equal to version 1.60 and less than or equal to version 2.99.

::::


::::{important}
All plugins have a runtime dependency on the `logstash-core-plugin-api` gem, and a development dependency on `logstash-devutils`.
::::



### Jar dependencies [_jar_dependencies_4]

In some cases, such as the [Elasticsearch output plugin](https://github.com/logstash-plugins/logstash-output-elasticsearch/blob/main/logstash-output-elasticsearch.gemspec#L22-L23), your code may depend on a jar file.  In cases such as this, the dependency is added in the gemspec file in this manner:

```ruby
  # Jar dependencies
  s.requirements << "jar 'org.elasticsearch:elasticsearch', '5.0.0'"
  s.add_runtime_dependency 'jar-dependencies'
```

With these both defined, the install process will search for the required jar file at [http://mvnrepository.com](http://mvnrepository.com) and download the specified version.



## Document your plugin [_document_your_plugin_4]

Documentation is an important part of your plugin. All plugin documentation is rendered and placed in the [Logstash Reference](/reference/index.md) and the [Versioned plugin docs](logstash-docs-md://vpr/integration-plugins.md).

See [Document your plugin](/extend/plugin-doc.md) for tips and guidelines.


## Add Tests [_add_tests_4]

Logstash loves tests. Lots of tests. If you’re using your new output plugin in a production environment, you’ll want to have some tests to ensure you are not breaking any existing functionality.

::::{note}
A full exposition on RSpec is outside the scope of this document. Learn more about RSpec at [http://rspec.info](http://rspec.info)
::::


For help learning about tests and testing, look in the `spec/outputs/` directory of several other similar plugins.


## Clone and test! [_clone_and_test_4]

Now let’s start with a fresh clone of the plugin, build it and run the tests.

* **Clone your plugin into a temporary location** Replace `GITUSERNAME` with your github username, and `MYPLUGINNAME` with your plugin name.

    * `git clone https://github.com/GITUSERNAME/logstash-``output-MYPLUGINNAME.git`

        * alternately, via ssh: `git clone git@github.com:GITUSERNAME/logstash-``output-MYPLUGINNAME.git`

    * `cd logstash-output-MYPLUGINNAME`


Then, you’ll need to install your plugins dependencies with bundler:

```
bundle install
```

::::{important}
If your plugin has an external file dependency described in `vendor.json`, you must download that dependency before running or testing.  You can do this by running:

```
rake vendor
```

::::


And finally, run the tests:

```
bundle exec rspec
```

You should see a success message, which looks something like this:

```
Finished in 0.034 seconds
1 example, 0 failures
```

Hooray! You’re almost there! (Unless you saw failures…​ you should fix those first).


## Building and Testing [_building_and_testing_4]

Now you’re ready to build your (well-tested) plugin into a Ruby gem.

### Build [_build_4]

You already have all the necessary ingredients, so let’s go ahead and run the build command:

```sh
gem build logstash-output-example.gemspec
```

That’s it!  Your gem should be built and be in the same path with the name

```sh
logstash-output-mypluginname-0.1.0.gem
```

The `s.version` number from your gemspec file will provide the gem version, in this case, `0.1.0`.


### Test installation [_test_installation_4]

You should test install your plugin into a clean installation of Logstash. Download the latest version from the [Logstash downloads page](https://www.elastic.co/downloads/logstash/).

1. Untar and cd in to the directory:

    ```sh
    curl -O https://download.elastic.co/logstash/logstash/logstash-9.0.0.tar.gz
    tar xzvf logstash-9.0.0.tar.gz
    cd logstash-9.0.0
    ```

2. Using the plugin tool, we can install the gem we just built.

    * Replace `/my/logstash/plugins` with  the correct path to the gem for your environment, and `0.1.0` with the correct version number from the gemspec file.

        ```sh
        bin/logstash-plugin install /my/logstash/plugins/logstash-output-example/logstash-output-example-0.1.0.gem
        ```

    * After running this, you should see feedback from Logstash that it was successfully installed:

        ```sh
        validating /my/logstash/plugins/logstash-output-example/logstash-output-example-0.1.0.gem >= 0
        Valid logstash plugin. Continuing...
        Successfully installed 'logstash-output-example' with version '0.1.0'
        ```

        ::::{tip}
        You can also use the Logstash plugin tool to determine which plugins are currently available:

        ```sh
        bin/logstash-plugin list
        ```

        Depending on what you have installed, you might see a short or long list of plugins: inputs, codecs, filters and outputs.

        ::::

3. Now try running Logstash with a simple configuration passed in via the command-line, using the `-e` flag.

    ::::{note}
    Your results will depend on what your output plugin is designed to do.
    ::::


Congratulations! You’ve built, deployed and successfully run a Logstash output.



## Submitting your plugin to [RubyGems.org](http://rubygems.org) and [logstash-plugins](https://github.com/logstash-plugins) [_submitting_your_plugin_to_rubygems_orghttprubygems_org_and_logstash_pluginshttpsgithub_comlogstash_plugins_4]

Logstash uses [RubyGems.org](http://rubygems.org) as its repository for all plugin artifacts. Once you have developed your new plugin, you can make it available to Logstash users by simply publishing it to RubyGems.org.

### Licensing [_licensing_4]

Logstash and all its plugins are licensed under [Apache License, version 2 ("ALv2")](https://github.com/elasticsearch/logstash/blob/main/LICENSE). If you make your plugin publicly available via [RubyGems.org](http://rubygems.org), please make sure to have this line in your gemspec:

* `s.licenses = ['Apache License (2.0)']`


### Publishing to [RubyGems.org](http://rubygems.org) [_publishing_to_rubygems_orghttprubygems_org_4]

To begin, you’ll need an account on RubyGems.org

* [Sign-up for a RubyGems account](https://rubygems.org/sign_up).

After creating an account, [obtain](http://guides.rubygems.org/rubygems-org-api/#api-authorization) an API key from RubyGems.org. By default, RubyGems uses the file `~/.gem/credentials` to store your API key. These credentials will be used to publish the gem. Replace `username` and `password` with the credentials you created at RubyGems.org:

```sh
curl -u username:password https://rubygems.org/api/v1/api_key.yaml > ~/.gem/credentials
chmod 0600 ~/.gem/credentials
```

Before proceeding, make sure you have the right version in your gemspec file and commit your changes.

* `s.version = '0.1.0'`

To publish version 0.1.0 of your new logstash gem:

```sh
bundle install
bundle exec rake vendor
bundle exec rspec
bundle exec rake publish_gem
```

::::{note}
Executing `rake publish_gem`:

1. Reads the version from the gemspec file (`s.version = '0.1.0'`)
2. Checks in your local repository if a tag exists for that version. If the tag already exists, it aborts the process. Otherwise, it creates a new version tag in your local repository.
3. Builds the gem
4. Publishes the gem to RubyGems.org

::::


That’s it! Your plugin is published! Logstash users can now install your plugin by running:

```sh
bin/logstash-plugin install logstash-output-mypluginname
```



## Contributing your source code to [logstash-plugins](https://github.com/logstash-plugins) [_contributing_your_source_code_to_logstash_pluginshttpsgithub_comlogstash_plugins_4]

It is not required to contribute your source code to [logstash-plugins](https://github.com/logstash-plugins) github organization, but we always welcome new plugins!

### Benefits [_benefits_4]

Some of the many benefits of having your plugin in the logstash-plugins repository are:

* **Discovery.** Your plugin will appear in the [Logstash Reference](/reference/index.md), where Logstash users look first for plugins and documentation.
* **Documentation.** Your plugin documentation will automatically be added to the [Logstash Reference](/reference/index.md).
* **Testing.** With our testing infrastructure, your plugin will be continuously tested against current and future releases of Logstash.  As a result, users will have the assurance that if incompatibilities arise, they will be quickly discovered and corrected.


### Acceptance Guidelines [_acceptance_guidelines_4]

* **Code Review.** Your plugin must be reviewed by members of the community for coherence, quality, readability, stability and security.
* **Tests.** Your plugin must contain tests to be accepted.  These tests are also subject to code review for scope and completeness.  It’s ok if you don’t know how to write tests — we will guide you. We are working on publishing a guide to creating tests for Logstash which will make it easier.  In the meantime, you can refer to [http://betterspecs.org/](http://betterspecs.org/) for examples.

To begin migrating your plugin to logstash-plugins, simply create a new [issue](https://github.com/elasticsearch/logstash/issues) in the Logstash repository. When the acceptance guidelines are completed, we will facilitate the move to the logstash-plugins organization using the recommended [github process](https://help.github.com/articles/transferring-a-repository/#transferring-from-a-user-to-an-organization).
