---
layout: post
title:  "Using Rails Active Records with UUID"
date:   2023-04-17 22:04:47 +0100
categories: rails
---

[Active Records] are a major part of any [Ruby on Rails] application. This _model_ layer is basically in charge of representing business data and logic. By default, Rails comes with SQLite3 configured, and all the data stored uses sequential IDs, which are _integers_. This means that whenever a new record is added, it is stored in the database as a number, and newer records use the previous reacord ID and increment +1.

There is no problem with this approach, but for some reasons, it is more secure or you just want to use a non-sequential ID to identify records in the database. Imagine that you have a User _model_, and to access each user, you have an address like: `http://example.com/users/1`. Anyone can just guess other users by incrementing this value. By having something like: `http://example.com/users/4702ea41-d269-4015-acbf-d3e411d05647`, it's almost impossible to guess the ID for other users, making your app a little bit more secure.

Thankfully, Rails and Postgres give us all the tools to set this up.

Let's start creating a brand new Rails app already defined with a PostgreSQL database:

```bash
rails new myApp -d postgres
```

After that, we can create a new migration to enable pgcrypto's extension:

```bash
rails g migration EnableUuid
```

```ruby
# db/migrate/xxxxxxxxx_enable_uuid.rb
class EnableUuid < ActiveRecord::Migration[7.0]
  def change
    enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
  end
end
```

```bash
rails db:migrate
```

This basically enables the extension only if it's not already enabled.

After that, we just need to set Rails to automatically use UUIDs as the default identifier for all new records.

```ruby
# config/application.rb
module MyApp
  class Application < Rails::Application
    # ...

    # Generate all models with UUID
    config.generators do |g|
      g.orm :active_record, primary_key_type: :uuid
    end
  end
end
```

From now on, all your records will be using UUID instead of incremental IDs without needing to change each migration.

If you are not sure if you should use UUID for your records, I recommend this article (pt-BR) about [UUID as PK] created by [@rponte].

[Active Records]: https://guides.rubyonrails.org/active_record_basics.html
[Ruby on Rails]: https://guides.rubyonrails.org/index.html
[UUID as PK]: https://gist.github.com/rponte/bf362945a1af948aa04b587f8ff332f8
[@rponte]: https://twitter.com/rponte