---
title: Override Devise Auth Token Controllers
date: 2020-01-29 15:56:25
author: Ashish Jajoria
categories:
- ["Ruby on Rails"]
tags: 
- backend
- authentication
- devise
- devise_auth_token
- rails
- ruby
- ror
- Ashish_Jajoria
---

For authentication and token management at backend in Ruby On Rails we use [devise-token-auth](https://github.com/lynndylanhurley/devise_token_auth).

{% asset_img devise_token_auth.png %}

## Sometimes we need to update some of the following default behaviours:-

- Registration(via facebook, twitter, mobile, email etc.)
- Password reset flow(email reset link OR OTP based)
- We would like to add or remove some fields from the signin API.
etc

## Configutation:-

Use [devise-token-auth-guide](https://devise-token-auth.gitbook.io/devise-token-auth/) to setup your devise configuration.

After configuration, your **routes.rb** would look like this:

```ruby
# config/routes.rb
mount_devise_token_auth_for 'User', at: 'auth'
```

## Overriding:-

- Create a package named `overrides`, in cotrollers package.
- **For overriding RegistrationsController** used for signup flow, add `registrations_controller.rb` to the package we just created and extent the RegistrationsController by `DeviseTokenAuth::RegistrationsController`.

```ruby
module Overrides
 class RegistrationsController < DeviseTokenAuth::RegistrationsController
  ...
 end
end
```

- Now write the `create` method yourself for your custom parameters you want to use while signing up a user with custom conditions and if there is any condition when you don't want to handle, then just call **super** and the default signup flow will work for that case.

```ruby
module Overrides
 class RegistrationsController < DeviseTokenAuth::RegistrationsController
  def create

   ... #Your custom conditions and handling

   @resource = User.new(email: email) #This may vary based on your params and conditions you want
   @resource.name = params[:name]
   @resource.password = params[:password]

   unless @resource.save
    render json: { message: @resource.errors.full_messages.join(', ') }, status: :bad_request
    return
   end

   @token = @resource.create_token
   @resource.save

   update_auth_header
   render_create_success
  end
 end
end
```

- Now Update your **routes.rb**

```ruby
mount_devise_token_auth_for 'User', at: 'auth', controllers: {
 registrations: 'overrides/registrations'
}
```

## Likewise we can override following controllers:-

- ConfirmationsController
- PasswordsController
- OmniauthCallbacksController
- SessionsController
- TokenValidationsController
&nbsp;

## References:-

1. [devise_token_auth](https://github.com/lynndylanhurley/devise_token_auth) gem
2. [Devise Token Auth](https://devise-token-auth.gitbook.io/devise-token-auth/) Guide
