1. [How to set up CORS for a new project](#how-to-set-up-cors-for-a-new-project)
2. [How to create basic email + password auth](#how-to-create-basic-email-and-password-auth)
3. [How to create CRUD API controller](#how-to-create-crud-api-controller)
4. [How to create comments system](#how-to-create-comments-system)
5. [How to implement FB sign up](#how-to-implement-fb-sign-up)
6. [How to implement Google sign up](#how-to-implement-google-sign-up)
7. [How to set up email notifications with SendGrid](#how-to-set-up-email-notifications-with-sendgrid)
8. [How To set up file uploads with AWS](#how-to-set-up-file-uploads-with-aws)
9. [How to set up 2FA authorization with Twilio](#how-to-set-up-2fa-authorization-with-Twilio)

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

## How To set up file uploads with AWS

1. Add carrierwave, mini_magick and fog-aws to Gemfile:
```ruby
# Files uploading/storing
gem 'carrierwave'

# ImageMagick for Carrierwave
gem 'mini_magick'

# S3 files storing
gem 'fog-aws'
```

2. Add base CarrierWave uploader (app/uploaders/base_uploader.rb):
```ruby
class BaseUploader < CarrierWave::Uploader::Base
end
```

3. Add store_dir method to BaseUploader. It will determine the path to the uploaded file:
```ruby
"uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
```

4. Create config/initializers/carrierwave.rb file. Configure carrierwave for test and development environments:

```ruby
CarrierWave.configure do |config|

  config.enable_processing = true

  if Rails.env.development? || Rails.env.test?
    config.storage = :file
    config.asset_host = ActionController::Base.asset_host
    config.cache_dir = "#{Rails.root}/public/uploads/tmp"
  end
end
```

5. Provide fog storage for production base_uploader:
```ruby
if Rails.env.development? || Rails.env.test?
  storage :file
else
  storage :fog
end
```

6. Add CLIENT_ADDRESS, SERVER_ADDRESS and S3_BUCKET to config/application.yml

7. Add S3_BUCKET to required_keys (config/initializers/figaro.rb):
```ruby
required_keys += %w[
  ...
  S3_BUCKET
  ...
]
```

8. Add s3, access_key_id and secret_access_key to application's credentials;

9. Provide carrierwave settings for production (config/initializers/carrierwave.rb):
```ruby
if Rails.env.development? || Rails.env.test?
  ...
else
  credentials = Rails.application.credentials[:s3] || {}
  config.cache_dir = "#{Rails.root}/tmp/uploads"
  config.fog_provider = 'fog/aws'
  config.fog_credentials = {
    provider: 'AWS',
    aws_access_key_id: credentials[:access_key_id],
    aws_secret_access_key: credentials[:secret_access_key],
    region: 'ap-southeast-1',
    endpoint: 'https://s3-ap-southeast-1.amazonaws.com'
  }
  config.fog_directory = ENV['S3_BUCKET']
  config.fog_public = true
  config.fog_attributes = { cache_control: "public, max-age=#{365.day.to_i}" }
end
```
## How to set up 2FA authorization with Twilio

1. Add twilio-ruby gem to Gemfile:
```ruby
# Twilio wrapper
gem 'twilio-ruby'
```

2. Add SMS acronym to config/initializers/inflections.rb:
```ruby
inflect.acronym 'SMS'
```

3. Add twilio credentials (account_sid, auth_token, sender_number) to application's credentials:

4. Add two_fa, two_fa_token, sms_code, sms_code_sent_at and phone columns to users table:
```ruby
class AddTwoFaToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :two_fa, :boolean, default: false, null: false
    add_column :users, :two_fa_token, :string, unique: true, index: true
    add_column :users, :sms_code, :string
    add_column :users, :sms_code_sent_at, :datetime
    add_column :users, :phone, :string, unique: true
  end
end
```

5. Create TwilioService. Initialize account_sid and auth_token there:
```ruby
class TwilioService

  def initialize
    twilio_credentials = Rails.application.credentials[:twilio]
    @account_sid = twilio_credentials[:account_sid]
    @auth_token = twilio_credentials[:auth_token]
  end

end
```

6. Add Twilio client initialization to TwilioService:
```ruby
def client
  Twilio::REST::Client.new(@account_sid, @auth_token)
end
```

7. Add send_sms method to TwilioService. Initialize sender_number there:
```ruby
def send_sms
  sender_number = Rails.application.credentials[:twilio][:sender_number]
end
```

8. Create new messages through the Twilio client. Also add body and phone params to the send_sms method:
```ruby
def send_sms(body, phone)
  ...

  client.messages.create(
    from: sender_number,
    to: phone,
    body: body
  )
end
```

9. Handle Twilio error during message sending:
```ruby
begin
  client.messages.create(
    from: sender_number,
    to: phone,
    body: body
  )
rescue Twilio::REST::RestError => error
  puts "Message '#{body}' can't be send to #{phone}!\n#{error}"
end
```

10. Create SMSAuthCodeWorker. It will send Twilio messages:
```ruby
class SMSAuthCodeWorker

  include Sidekiq::Worker

  sidekiq_options queue: 'low'

end
```

11. Add perform method to send Twilio messages:
```ruby
def perform(user_id)
  user = User.find(user_id)
  TwilioService.new.send_sms("Code: #{user.sms_code}", user.phone)
end
```

12. Create change_2fa method in order to check phone presence:
```ruby
validate :change_2fa, on: :update

def change_2fa
  if phone.blank? && will_save_change_to_attribute?(:two_fa, from: false, to: true)
    errors.add(:two_fa, 'Need to have telephone number')
  end
end
```

13. Create TwoFactorAuthentication service. Add send_2fa_code method. It will generate two_fa_token and sms_code and send it to user:
```ruby
module Auth
  module TwoFactorAuthentication

    extend ActiveSupport::Concern

    def send_2fa_code(user)
      two_fa_token = SecureRandom.hex(32)
      user.update_columns(
        two_fa_token: two_fa_token,
        sms_code: rand(999..9999),
        sms_code_sent_at: Time.now
      )

      SMSAuthCodeWorker.perform_async(user.id)
      two_fa_token
    end

  end
end
```

14. Add private sms_code_expired method to TwoFactorAuthentication service. Code has expired if it was sent more than 2 minutes ago:
```ruby
private

def sms_code_expired?(user)
  user.sms_code_sent_at && user.sms_code_sent_at < 2.minutes.ago
end
```

15. Add confirm_2fa method to TwoFactorAuthentication service. Find user by sms_code and two_fa_token. Raise exception if the user is blank or sms_code has expired:
```ruby
def confirm_2fa
  user = User.find_by(sms_code: @sms_code, two_fa_token: @two_fa_token)
  if user.blank?
    raise CustomException.new(sms_code: 'Code is incorrect')
  elsif sms_code_expired?(user)
    raise CustomException.new(sms_code: 'Code is expired')
  end
end
```

Otherwise, authorize user:
```ruby
def confirm_2fa
  ...
  if user.blank?
    ...
  else
    user.update_columns(two_fa_token: nil, sms_code_sent_at: nil, sms_code: nil)
    user.update_tracked_fields!(@remote_ip)
    authorize(user.id)
  end
end
```

16. Add confirm_2fa params to auth helper:
```ruby
requires :sms_code, type: String
requires :two_fa_token, type: String
```

And add this params to AuthService initialize:
```ruby
def initialize(params = {})
  ...
  @two_fa_token = params[:two_fa_token]
  @sms_code = params[:sms_code]
end
```

17. Add confirm_2fa endpoint:
```ruby
desc 'Confirm 2FA by SMS code'
params do
  use :confirm_2fa
end
post 'confirm_2fa', root: false do
  attrs = declared(params).to_h.symbolize_keys
  attrs[:remote_ip] = remote_ip
  AuthService.new(attrs).confirm_2fa
end
```

18. Check two_fa flag and send_2fa_code before login:
```ruby
if user.two_fa
  send_2fa_code(user)
else
  user.update_tracked_fields!(remote_ip)
  authorize(user.id)
end
```
