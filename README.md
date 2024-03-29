## Notes

#### Create a new app and add devise and pundit and configure

- `ruby rails new blog_with_pundit`
- In Gemfile:
```
ruby gem "devise"
ruby gem "pundit"
```
- `bundle` to install the added gems
- Configure devise ```rails g devise:install```
- Generate a User model (with a role attribute) with devise ```rails g devise User role```
- Generate a Post model:
    - ```rails g scaffold Post user_id:integer title:string body:text```
- ```rake db:migrate``` & add the root path to ```root 'posts=index'```
- Configure/install Pundit ```rails g pundit:install```
    - This should create an application_policy.rb that is by default designed to lockdown access to everything

#### Get Pundit to do it's thing
- In ApplicationController: ```include Pundit```

#### Add policies for Post model
- Create a post_policy.rb
- Override the index? method from application_policy.rb
```ruby
def index?
  true
end
```
- Authorize the @posts in the posts_controller.rb index action
```
authorize @posts
```

#### Use helper methods to only show links when authorized to use
```
<% if policy(Post).new? %>
  <%= link_to 'New Post', new_post_path %>
<% end %>
```

#### Check for authorization in all controllers
In ApplicationController: ```after_action :verify_authorized```

#### Assign has_many and belongs_to relationships
- In user.rb ```has_many :posts```
- In post.rb ```belongs_to :user```

#### Make devise actions work
- Since we are verifying authorization for all controller actions, we need to create an exception for devise actions
- Update the after_action in ApplicationController: ```after_action :verify_authorized, unless: :devise_controller?```
- Test this by adding signup, login and logout links in application.html.erb
```
<% if user_signed_in? %>
  <%= current_user.email %>
  <%= link_to "Logout", destroy_user_session_path, method: :delete %>
<% else %>
  <%= link_to "Sign Up", new_user_registration_path %>
  <%= link_to "Login", new_user_session_path %>
<% end %>
```

#### Roles-based authorization
- Add some method to check for user roles in user.rb
```
def admin?
  role == "admin"
end

def regular?
  role == "regular"
end

def guest?
  role == "guest"
end
```
- Then in post_policy, you can check the authorization based on a user's role.
```
def destroy?
  user.present? && user.admin?
end
```
