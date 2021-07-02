1. [How to set up CORS for a new project](#how-to-set-up-cors-for-a-new-project)
2. [How to create basic email + password auth](#how-to-create-basic-email-and-password-auth)
3. [How to create CRUD API controller](#how-to-create-crud-api-controller)
4. [How to create comments system](#how-to-create-comments-system)
5. [How to implement FB sign up](#how-to-implement-fb-sign-up)
6. [How to implement Google sign up](#how-to-implement-google-sign-up)
7. [How to set up email notifications with SendGrid](#how-to-set-up-email-notifications-with-sendgrid)

## How to set up CORS for a new project

1. Add a CORS_ALLOWED_ORIGINS array that includes all the allowed path for the app (config/initializers/cors.rb):
```ruby
CORS_ALLOWED_ORIGINS = [
  'localhost', 'http://localhost', '127.0.0.1', /localhost:(\d+)/, /127.0.0.1:(\d+)/,
  %r{(http|https):\/\/example.herokuapp.com}
].freeze
```

2. Pass CORS_ALLOWED_ORIGINS array to origins:
```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins CORS_ALLOWED_ORIGINS
    resource '*',
       headers: :any,
       methods: [:get, :post, :put, :patch, :delete, :options, :head]
    end
  end
end
```

3. Replace '*' resource with 'api':
```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins CORS_ALLOWED_ORIGINS
    resource '/api/*', headers: :any, methods: %i[get post options delete put patch]
  end
end
```

## How to create basic email and password auth

1. Add the required authentication gems::
```ruby
gem 'devise'
gem 'doorkeeper'
gem 'wine_bouncer'
```

2. Install devise and generate devise user:
```ruby
rails generate devise:install
rails generate devise User
```

3. Tell the router to use users controller (passwords only):
```ruby
devise_for :users, only: [:passwords]
```

4. Generate doorkeeper configuration file by running in terminal:
```ruby
rails generate doorkeeper:install
```

5. Generate doorkeeper migration:
```ruby
  rails generate doorkeeper:migration
```

6. Create AuthService class and initialize it with data from passed parameters:
```ruby
def initialize(params = {})
  # data from passed parameters
  @email = params[:email]&.strip&.downcase
  @password = params[:password]
end
```
7. Add authorize method to create auth_token and build response object:
```ruby
def authorize(record_id)
  # create auth token and build response object
  token = AccessToken.create(resource_owner_id: record_id)
  Doorkeeper::OAuth::TokenResponse.new(token).body
end
```

8. Add registration method to create a new user and authorize them:
```ruby
def registration
  user = User.create!(email: @email, password: @password)
  authorize(user.id)
end
```

9. Add login method to find user by email and authorize them:
```ruby
def login
  user = User.find_by(email: @email)

  if user&.valid_password?(@password)
    authorize(user.id)
  else
    raise CustomException.new(password: 'Password is incorrect or email not found')
  end
end
```

## How to create CRUD API controller

1. Create concern with params helper:
```ruby
module Params
  module Admin
    module Users

      extend Grape::API::Helpers

      params :user_params do
        optional :parameter, type: String
      end
    end
  end
end
```

2. Create serializer with necessary attributes:
```ruby
class ModelSerializer < ActiveModel::Serializer

  attributes :attribute1,
             :attribute2


end
```

3. Initialize API controller with included Defaults controller and appropriate helper:
```ruby
module API
  module V1
    class Users < API::V1::Root

      include API::V1::Defaults
      helpers Params::Admin::Users

      resource :users do
      end
    end
  end
end
```

4. Add endpoint to get list of objects:
```ruby
desc 'Get users'
oauth2
get root: false do
  AdminEngine.new(
    model: 'User',
    fields: %i[
      attribute1
      attribute2
    ],
    params: params
  ).collection
end
```

5. Add endpoint to get object:
```ruby
desc 'Get user'
oauth2
get ':id', root: false do
  User.find(params[:id])
end
```

6. Add endpoint to create object:
```ruby
desc 'Create user'
oauth2
params do
  use :user_params
end
post root: false do
  attrs = declared(params, include_missing: false).to_h
  User.create!(attrs)
end
```

7. Add endpoint to update object:
```ruby
desc 'Update user'
oauth2
params do
  use :user_params
end
put ':id', root: false do
  user = User.find(params[:id])
  attrs = declared(params, include_missing: false).to_h
  user.update!(attrs)
  user
end
```

8. Add endpoint to delete object:
```ruby
desc 'Delete user'
oauth2
delete ':id', root: false do
  User.find(params[:id]).destroy
  status(200)
end
```

9. Add controller to Root API class:
```ruby
module API
  module V1
    class Root < Dispatch
      ...
      mount API::V1::Users
      ...
    end
  end
end
```

## How to create comments system

1. Generate create_comments migration to create comments entity. Required fields: user_id, content:
```ruby
class CreateComments < ActiveRecord::Migration[6.0]
  def change
    create_table :comments do |t|
      t.references :user, index: true

      t.text :content

      t.timestamps
    end
  end
end
```

2. Add index, get, put and delete endpoints:
[How to create API CRUD controller](#how-to-create-api-crud-controller)

## How to implement FB sign up

1. Add facebook_uid index to users table:
```ruby
class AddFacebookUidToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :facebook_uid, :string
    add_index :users, :facebook_uid, unique: true
  end
end
```

2. Add facebook_uid to initialize method of AuthService:
```ruby
class AuthService
  ...
  def initialize(params = {})
    ...
    @facebook_uid = params[:facebook_uid]
  end
  ...
end
```

3. Create Facebook auth concern:
```ruby
module AuthConcerns
  module Facebook

    extend ActiveSupport::Concern

  end
end
```

and include it in AuthService
```ruby
class AuthService
  include AuthConcerns::Facebook
  ...
end
```

4. Add facebook_register method to Facebook AuthConcern (create new user with facebook_uid):
```ruby
def facebook_register
  user = User.new(name: @name, password: SecureRandom.hex, facebook_uid: @facebook_uid)
  user.email = @email.present? ? @email : "fb_#{@facebook_uid}@generated.com"
  user.save!
  user
end
```

5. Add facebook method to Google AuthConcern (add facebook_uid to user or call facebook_register):
```ruby
def facebook
  user = User.find_by(facebook_uid: @facebook_uid)

  # try to find by Facebook email
  if user.nil? && @email.present?
    user = User.find_by(email: @email)
    user&.update(facebook_uid: @facebook_uid)
  end

  # try to set missing info
  if user.present? && user.name.blank? && @name.present?
    user.update(name: @name)
  end

  # register new user
  user ||= facebook_register
  # login
  authorize(user.id)
end
```

6. Add facebook params with required google_uid and optional email and name:
```ruby
params :facebook do
  requires :facebook_uid, type: String, allow_blank: false
  optional :email, type: String
  optional :name, type: String
end
```

7. Add facebook endpoint to auth controller and call previously added facebook auth method:
```ruby
desc 'Facebook login'
params do
  use :facebook
end
post 'facebook' do
  attrs = declared(params, include_missing: false).to_h.symbolize_keys
  AuthService.new(attrs).facebook
end
```

## How to implement Google sign up

1. Add google_uid index to users table:

```ruby
class AddGoogleUidToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :google_uid, :string
    add_index :users, :google_uid, unique: true
  end
end
```

2. Add google_uid to initialize method of AuthService:

```ruby
class AuthService
  ...
  def initialize(params = {})
    ...
    @google_uid = params[:google_uid]
  end
  ...
end
```

3. Add Google auth concern:

```ruby
module AuthConcerns
  module Google

    extend ActiveSupport::Concern

  end
end
```

and include it in AuthService
```ruby
class AuthService
  include AuthConcerns::Google

  ...
end
```

4. Add google_register method to Google AuthConcern (create new user with google_uid):
```ruby
def google_register
  User.create!(
    google_uid: @google_uid,
    name: @name,
    password: SecureRandom.hex,
    email: @email,
  )
end
```

5. Add google method to Google AuthConcern (add google_uid to user or call google_register):
```ruby
def google
  user = User.find_by(google_uid: @google_uid)

  if user.nil?
    user = User.find_by(email: @email)
    user&.update(google_uid: @google_uid)
  end

  # register new user
  user ||= google_register
  # login
  authorize(user.id)
end
```

6. Add google params with required google_uid, email and name:
```ruby
params :google do
  requires :google_uid, type: String, allow_blank: false
  requires :email, type: String, allow_blank: false
  requires :name, type: String, allow_blank: false
end
```

7. Add google endpoint to auth controller and call previously added google auth method:
```ruby
desc 'Google login'
params do
  use :google
end
post 'google' do
  attrs = declared(params, include_missing: false).to_h.symbolize_keys
  AuthService.new(attrs).google
end
```

## How to set up email notifications with SendGrid

1. Set default from value in app/mailers/application_mailer.rb:
```ruby
default from: 'no-reply@email_host.com'
```

2. Add sendgrid_api_key to application's credentials.

3. Call a sendgrid_api_key in config/initializers/setup_mail.rb:
```ruby
api_key = Rails.application.credentials[:sendgrid_api_key]
```

4. Configure smtp_settings in config/initializers/setup_mail.rb:
```ruby
ActionMailer::Base.smtp_settings = {
  address: 'smtp.sendgrid.net',
  port: '587',
  authentication: :plain,
  user_name: 'apikey',
  password: api_key,
  enable_starttls_auto: true
}
```
