Setup for New Ruby on Rails Project
=

*Disclaimer: The following guide has not been tested extensively. Please use it at your own risk!*

## Create Rails Project
```bash
rails new <project name> --database=postgresql -T
cd <project name>
rvm gemset create <gemset name>
rvm gemset use <gemset name>
touch .ruby-version
touch .ruby-gemset
bundle install
```
```
# .ruby-version
2.4.1
```
```
# .ruby-gemset
<gemset name>
```

## Gems Installation
Instructions:
1. You may skip any gem that is not required.
2. Please run 'bundle' after each gem added in gemfile, even if not stated explicitly.
3. Highly recommend to commit after each gem installation.
4. Below is my preferred order of installation. You can install in any order, but be mindful of the interactions between gems (e.g Devise and Simple Form)

#### [Bootstrap](https://github.com/twbs/bootstrap-rubygem)
```ruby
gem 'bootstrap', '~> 4.0.0.beta2.1'
```
```scss
# app/assets/stylesheets/application.scss
# Remove all the *= require and *= require_tree
@import "bootstrap";
@import "/*";
```

#### [Simple Form](https://github.com/plataformatec/simple_form)
```ruby
gem 'simple_form'
```
```bash
# Terminal
bundle
rails generate simple_form:install --bootstrap #bootstrap option is optional
```

#### [Faker](https://github.com/stympy/faker)
```ruby
gem 'faker', :git => 'https://github.com/stympy/faker.git', :branch => 'master'
```
```bash
# Terminal
bundle
```
```ruby
# seeds.rb
3.times do
  Place.create(name: Faker::Name.name, email: "place"+ Faker::Number.between(1, 10).to_s + "@mail.com", description: Faker::Hipster.paragraph(2, false, 4))
end
p "places created"
```

#### [Slim Template](https://github.com/slim-template/slim-rails)
```ruby
gem "slim-rails"
```

#### [RSpecs](https://github.com/rspec/rspec-rails)
```ruby
# group :development, :test
gem 'rspec-rails', '~> 3.6'
```
```bash
# Terminal
bundle
rails generate rspec:install
rspec
```

#### [Shoulda Matchers](https://github.com/thoughtbot/shoulda-matchers)
```ruby
# group :development, :test
gem 'shoulda-matchers', '~> 3.1'
```
```ruby
# rails_helper.rb
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```
```bash
# Terminal
bundle
rspec
```

#### [Factory Bot Rails](https://github.com/thoughtbot/factory_bot_rails)
```ruby
# group :development, :test
gem 'factory_bot_rails'
```
```ruby
# rails_helper.rb
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```
```bash
# Terminal
bundle
rspec
```

#### [Rails Controller Testing](https://github.com/rails/rails-controller-testing)
```ruby
# gemfile
gem 'rails-controller-testing'
```
```bash
# Terminal
bundle
```
#### [Devise](https://github.com/plataformatec/devise)
```ruby
gem 'devise'
```
```bash
# Terminal
bundle
rails generate devise:install

# Models generation
rails generate devise MODEL
rails db:migrate

# Views generation
rails generate devise:views # for single model
# or
rails generate devise:views <resources> # for multiple models
```
```ruby
# config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

##### [Devise RSpecs](https://github.com/plataformatec/devise)
```ruby
# rails_helper.rb
RSpec.configure do |config|
  config.include Devise::Test::ControllerHelpers, type: :controller
  config.include Devise::Test::ControllerHelpers, type: :view
end
```

#### [dotenv](https://github.com/bkeepers/dotenv)
```ruby
# group :development, :test
gem 'dotenv-rails'
```
```bash
# Terminal
bundle
touch .env
```
```
# .gitignore
.env
```
#### [Mailer](http://culttt.com/2016/03/02/getting-started-with-action-mailer-in-ruby-on-rails/)
```bash
# Terminal
rails generate mailer UserMailer
```
```ruby
# application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout "mailer"
end
```
[Official guide](http://guides.rubyonrails.org/action_mailer_basics.html)

#### [MailCatcher](https://mailcatcher.me/)
```ruby
gem 'mailcatcher'
```
```ruby
# environments/development.rb
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }
```
```bash
# Terminal
mailcatcher
```
Go to http://localhost:1080/
### Github Stuff
```git
cd <project folder>
git init
# check git
git remote -v
# add git remote repo link
git remote add origin https://github.com/../
git status
git add .
git commit -m 'first commit'
git push -u origin master
```

### flash notice
```htmlerb
# app/views/layouts/application.html.erb
<body>
    <div><%= flash.class %></div>
    <div>
      <% flash.each do |key, value| %>
        <%= key %>: <%= value %>
        <% end %>
    </div>
    <div><%= flash %></div>
  <%= yield %>
```
```slim
body
  - flash.each do |name, msg|
    = content_tag :div, msg, class: name
  = yield  
```
```ruby
# put in controller
flash[:notice] = 'THIS IS A NOTICE'
# When controller is rendering, use flash.now[:notice]
# When controller is redirecting, use flash[:notice]
```

### User Following Other Users
```ruby
# migration file, to create join table called Relation
class CreateRelations < ActiveRecord::Migration[5.1]
  def change
    create_table :relations do |t|
      t.references :subscriber
      t.references :poster

      t.timestamps
    end
    add_foreign_key :relations, :users, column: :subscriber_id
    add_foreign_key :relations, :users, column: :poster_id
    add_index :relations, [:subscriber_id, :poster_id], unique: true
  end
end
```
```ruby
# user.rb, to create join table called Relation
class User < ApplicationRecord
  has_many :subscriber_users, class_name: "Relation", foreign_key: :poster_id, dependent: :destroy
  has_many :subscribers, through: :subscriber_users
  has_many :poster_users, class_name: "Relation", foreign_key: :subscriber_id
  has_many :posters, through: :poster_users
end
```
```ruby
# user.rb, to create join table called Relation
class Relation < ApplicationRecord
  belongs_to :subscriber, class_name: "User"
  belongs_to :poster, class_name: "User"
  validate :disallow_subscribe_to_self
  def disallow_subscribe_to_self
    if subscriber_id == poster_id
      errors.add(:poster_id, 'cannot follow yourself')
    end
  end
end
```
```ruby
# routes.rb
Rails.application.routes.draw do
  post 'pages/follow/:id', to: 'pages#relate', as: :follow
  delete 'pages/unfollow/:poster_id', to: 'pages#unrelate', as: :unfollow
end
```
```htmlerb
<h3>Suggested Users to Follow</h3>
<ul>
  <% @users.each do |user| %>
  <li>
    <%= link_to user.username, profile_user_path(user) %>
    &nbsp
    <%= link_to "follow", follow_path(user), method: :post if Relation.find_by(subscriber_id: current_user.id, poster_id: user.id) == nil %>
    <%= link_to "UnF", unfollow_path(user), method: :delete if Relation.find_by(subscriber_id: current_user.id, poster_id: user.id) != nil %>
  </li>
  <% end %>
```
```ruby
# application_controller.rb - follow & unfollow
class ApplicationController < ActionController::Base
  def relate
    @follow = Relation.new(subscriber_id: current_user.id, poster_id: params[:id])
    if @follow.save
      flash[:notice] = "Now following."
      redirect_to root_path
    else
      flash[:error] = "Unable to follow user."
      redirect_to root_path
    end
  end
  def unrelate
    @follow = Relation.find_by(subscriber_id: current_user.id, poster_id: params[:poster_id])
    if @follow != nil && @follow.destroy
      flash[:notice] = "Unfollowed."
      redirect_to root_path
    else
      flash[:alert] = "Unable to unfollow user."
      redirect_to root_path
    end
  end
end  
```
