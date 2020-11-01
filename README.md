
# 客製化Rails App的template


需求：打造快速工作流程，我需要`rails new`一個app時快速幫我裝好需要的`gem`及其他常用設定。



## Step1. 到自己的根目錄，新增`~/.railsrc`

使用Mac，在自己的根目錄下：`/Users/tingtinghsu`
（跟編輯其他設定檔如`~/.zshrc`的位置一樣）

```
touch ~/.railsrc
open ~/.railsrc
```

## Step2. 編輯`~/.railsrc`檔案

例如：
```
--skip-action-mailbox --database=postgresql --webpack=vue --template=/Users/tingtinghsu/Documents/projects/template_demo/template.rb
```

## Step3. 製作屬於自己的`template_demo`

- 在GitHub上從`tingtinghsu/kickoff_tailwind`建立一個`template`叫`template_demo`

![](https://i.imgur.com/HH5APZD.png)


- 把`template_demo` clone到本地，並且把正確路徑寫在剛剛的`~/.railsrc`檔案

```
--database=postgresql --webpack=vue --template=/Users/tingtinghsu/Documents/projects/template_demo/template.rb
```

## Step4. 編寫`template.rb`檔案，根據需求安裝gem

```
def source_paths
  [File.expand_path(File.dirname(__FILE__))]
end

def add_gems
  gem 'devise', '~> 4.7', '>= 4.7.2'
  gem 'friendly_id', '~> 5.3'
  gem 'sidekiq', '~> 6.1', '>= 6.1.1'
  gem 'name_of_person', '~> 1.1', '>= 1.1.1'
end

gem_group :development, :test do
  gem 'hirb-unicode', '~> 0.0.5'
  gem 'rspec-rails', '~> 4.0'
  gem 'factory_bot_rails', '~> 5.1', '>= 5.1.1'
  gem 'faker', '~> 2.11'
  gem 'pry-rails', '~> 0.3.9'
end

def add_users
  # Install Devise
  generate "devise:install"

  # Configure Devise
  environment "config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }",
              env: 'development'

  route "root to: 'home#index'"

  # Create Devise User
  generate :devise, "User", "first_name", "last_name", "admin:boolean"

  # set admin boolean to false by default
  in_root do
    migration = Dir.glob("db/migrate/*").max_by{ |f| File.mtime(f) }
    gsub_file migration, /:admin/, ":admin, default: false"
  end

  # name_of_person gem
  append_to_file("app/models/user.rb", "\nhas_person_name\n", after: "class User < ApplicationRecord")
end

def copy_templates
  directory "app", force: true
end

def add_tailwind
  run "yarn add tailwindcss"
  run "yarn add @fullhuman/postcss-purgecss"

  run "mkdir -p app/javascript/stylesheets"

  append_to_file("app/javascript/packs/application.js", 'import "stylesheets/application"')
  inject_into_file("./postcss.config.js",
  "let tailwindcss = require('tailwindcss');\n",  before: "module.exports")
  inject_into_file("./postcss.config.js", "\n    tailwindcss('./app/javascript/stylesheets/tailwind.config.js'),", after: "plugins: [")

  run "mkdir -p app/javascript/stylesheets/components"
end

def copy_postcss_config
  run "rm postcss.config.js"
  copy_file "postcss.config.js"
end

# Remove Application CSS
def remove_app_css
  remove_file "app/assets/stylesheets/application.css"
end

def add_sidekiq
  environment "config.active_job.queue_adapter = :sidekiq"

  insert_into_file "config/routes.rb",
    "require 'sidekiq/web'\n\n",
    before: "Rails.application.routes.draw do"

  content = <<-RUBY
    authenticate :user, lambda { |u| u.admin? } do
      mount Sidekiq::Web => '/sidekiq'
    end
  RUBY
  insert_into_file "config/routes.rb", "#{content}\n\n", after: "Rails.application.routes.draw do\n"
end

def add_foreman
  copy_file "Procfile"
  copy_file "Procfile.dev"
end

def add_friendly_id
  generate "friendly_id"
end

# 略
```

## Step6. 產生一個App出來，可以馬上使用`devise`, `tailwind.css`和`Vue`！

`rails new demo_rails_app`
`foreman start -f Procfile.dev`

![](https://i.imgur.com/ZboNkWZ.png)


Ref:
* [How to Super Charge Your New Rails App Workflow](https://www.youtube.com/watch?v=ez5d_TEMS-w)
* [A Guide to Using Ruby on Rails Application Templates](https://www.youtube.com/watch?v=kuKMRl8nj2w&feature=youtu.be)
* [Rails Kickoff – Tailwind](https://github.com/tingtinghsu/kickoff_tailwind)
* [rails-template](https://github.com/kaochenlong/rails-template)
