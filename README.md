# Rails Openshift Template #
This is a rails quickstart for openshift with the following extras:
 - Bootstrap integration via [twitter-bootstrap-rails](https://github.com/seyhunak/twitter-bootstrap-rails)
 - [Paperclip](https://github.com/thoughtbot/paperclip) integration
 - Global YAML configuration file in `config/application.yml`
 - General Openshift Configuration (props to [rails-example](https://github.com/openshift/rails-example))

## Usage ##
This is built for Openshift, create an account on Openshift.com, install rhc, and then run the following:
~~~
rhc app-create APP_NAME ruby-1.9 mysql-5.1 --from-code https://github.com/tiwillia/rails-bootstrap-openshift.git
~~~

For general Openshift usage help, see the [Getting Started page](https://www.openshift.com/get-started)

## Paperclip Configuration for Openshift ##

For those looking for just the paperclip configuration example:

In `Gemfile`:
~~~
gem 'paperclip'
gem 'rmagick' 
~~~

In `.openshift/action_hooks/deploy`:
~~~
STORED_IMAGE_ASSETS="${OPENSHIFT_DATA_DIR}public/images" 
LIVE_IMAGE_ASSETS="${OPENSHIFT_REPO_DIR}public/"

# Ensure the image asset directory exists
if [ ! -d "${STORED_IMAGE_ASSETS}" ]; then
  echo " Creating permanent images directory"
  mkdir -p "${STORED_IMAGE_ASSETS}"
fi

# Ensure the image asset directory exists
if [ ! -d "${LIVE_IMAGE_ASSETS}" ]; then
  echo " Creating live images directory"
  mkdir "${LIVE_IMAGE_ASSETS}"
fi

ln -sf "${STORED_IMAGE_ASSETS}" "${LIVE_IMAGE_ASSETS}"
~~~

In `config/application.rb`:
~~~
# This block below adds a global YAML configuration file in config/application.yml
CONFIG = YAML.load(File.read(File.expand_path('../application.yml', __FILE__)))
CONFIG.merge! CONFIG.fetch(Rails.env, {})
CONFIG.symbolize_keys!

# This is required for paperclip to work with openshift
if ENV['OPENSHIFT_DATA_DIR']
  CONFIG[:data_dir] = ENV['OPENSHIFT_DATA_DIR']
end
~~~

In `config/application.yml`:
~~~
# data_dir should be defined as the local repo root directory in your development environment
data_dir: "/home/tiwillia/Projects/rails-openshift-bootstrap/"
~~~

In your model, have at least:
~~~
has_attached_file :photo,
  :url => "/images/:id.:extension",
  :path => "#{CONFIG[:data_dir]}public/images/:id.:extension"
~~~

