# Omniauth::Wordpress::Oauth2::Plugin

A Wordpress Oauth2 Provider Plugin Strategy for Omniauth

Use with the Wordpress Oauth2 Provider plugin to turn  your wordpress install into an authentication provider: https://github.com/jwickard/wordpress-oauth


## Installation

Add this line to your application's Gemfile:

    gem 'omniauth-wordpress-oauth2-plugin', github: 'jwickard/omniauth-wordpress-oauth2-plugin'

And then execute:

    $ bundle

<!-- Or install it yourself as:

    $ gem install omniauth-wordpress-oauth2-plugin
-->
## Usage

### Devise / Omniauth
Add provider to your `config/initializers/devise.rb` ex:

```ruby
config.omniauth :wordpress_oauth2, 'APP_KEY', 'APP_SECRET',
                  strategy_class: OmniAuth::Strategies::WordpressOauth2Plugin, 
                  client_options: { site: 'http://yourcustomwordpress.com' }
```

### Omniauth / Rails
Add provider to your `config/initializers/omniauth.rb` ex:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :wordpress_oauth2, 'APP_KEY', 'APP_SECRET',
                  strategy_class: OmniAuth::Strategies::WordpressOauth2Plugin, 
                  client_options: { site: 'http://yourcustomwordpress.com' }
end
```

## Add Provider to Wordpress
Asuming you have already installed the wordpress oauth2 provider plugin, add a provider for this rails applicaiton to the plugin configuration:

![alt tag](https://raw.github.com/jwickard/omniauth-wordpress-oauth2-plugin-example/master/config-screen.png)

make sure the callback url you have configured is of the form:

`http://your-rails-site.com/users/auth/wordpress_oauth2/callback`

Then configure your rails application with the app key and secret generated by the wordpress plugin.

## Add callback

Configure routes to add omniauth callbacks controller `config/routes.rb` (which you implement):

```ruby
devise_for :users, controllers: { omniauth_callbacks: 'omniauth_callbacks' }
```
Implement omniauth_callbacks controller `app/controllers/omniauth_callbacks_controller.rb` :

```ruby
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def wordpress_oauth2
    #You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.find_for_wordpress_oauth2(request.env["omniauth.auth"], current_user)

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Wordpress") if is_navigational_format?
    else
      session["devise.wordpress_oauth2_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end
end
```

### Example application 

https://github.com/jwickard/omniauth-wordpress-oauth2-plugin-example

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
