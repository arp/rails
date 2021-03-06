*   Don't allow `quote_value` to be called without a column.

    Some adapters require column information to do their job properly.
    By enforcing the provision of the column for this internal method
    we ensure that those using adapters that require column information
    will always get the proper behavior.

    *Ben Woosley*

*   When using optimistic locking, `update` was not passing the column to `quote_value`
    to allow the connection adapter to properly determine how to quote the value. This was
    affecting certain databases that use specific colmn types.

    Fixes: #6763

    *Alfred Wong*

*   rescue from all exceptions in `ConnectionManagement#call`

    Fixes #11497

    As `ActiveRecord::ConnectionAdapters::ConnectionManagement` middleware does
    not rescue from Exception (but only from StandardError), the Connection
    Pool quickly runs out of connections when multiple erroneous Requests come
    in right after each other.

    Rescuing from all exceptions and not just StandardError, fixes this
    behaviour.

    *Vipul A M*

*   `change_column` for PostgreSQL adapter respects the `:array` option.

    *Yves Senn*

*   Remove deprecation warning from `attribute_missing` for attributes that are columns.

    *Arun Agrawal*

*   Remove extra decrement of transaction deep level.

    Fixes: #4566

    *Paul Nikitochkin*

*   Reset @column_defaults when assigning `locking_column`.
    We had a potential problem. For example:

    class Post < ActiveRecord::Base
      self.column_defaults  # if we call this unintentionally before setting locking_column ...
      self.locking_column = 'my_locking_column'
    end

    Post.column_defaults["my_locking_column"]
    => nil # expected value is 0 !

    *kennyj*

*   Remove extra select and update queries on save/touch/destroy ActiveRecord model
    with belongs to reflection with option `touch: true`.

    Fixes: #11288

    *Paul Nikitochkin*

*   Remove deprecated nil-passing to the following `SchemaCache` methods:
    `primary_keys`, `tables`, `columns` and `columns_hash`.

    *Yves Senn*

*   Remove deprecated block filter from `ActiveRecord::Migrator#migrate`.

    *Yves Senn*

*   Remove deprecated String constructor from `ActiveRecord::Migrator`.

    *Yves Senn*

*   Remove deprecated `scope` use without passing a callable object.

    *Arun Agrawal*

*   Remove deprecated `transaction_joinable=` in favor of `begin_transaction`
    with `:joinable` option.

    *Arun Agrawal*

*   Remove deprecated `decrement_open_transactions`.

    *Arun Agrawal*

*   Remove deprecated `increment_open_transactions`.

    *Arun Agrawal*

*   Remove deprecated `PostgreSQLAdapter#outside_transaction?`
    method. You can use `#transaction_open?` instead.

    *Yves Senn*

*   Remove deprecated `ActiveRecord::Fixtures.find_table_name` in favor of
    `ActiveRecord::Fixtures.default_fixture_model_name`.

    *Vipul A M*

*   Removed deprecated `columns_for_remove` from `SchemaStatements`.

    *Neeraj Singh*

*   Remove deprecated `SchemaStatements#distinct`.

    *Francesco Rodriguez*

*   Move deprecated `ActiveRecord::TestCase` into the rails test
    suite. The class is no longer public and is only used for internal
    Rails tests.

    *Yves Senn*

*   Removed support for deprecated option `:restrict` for `:dependent`
    in associations.

    *Neeraj Singh*

*   Removed support for deprecated `delete_sql` in associations.

    *Neeraj Singh*

*   Removed support for deprecated `insert_sql` in associations.

    *Neeraj Singh*

*   Removed support for deprecated `finder_sql` in associations.

    *Neeraj Singh*

*   Support array as root element in JSON fields.

    *Alexey Noskov & Francesco Rodriguez*

*   Removed support for deprecated `counter_sql` in associations.

    *Neeraj Singh*

*   Do not invoke callbacks when `delete_all` is called on collection.

    Method `delete_all` should not be invoking callbacks and this
    feature was deprecated in Rails 4.0. This is being removed.
    `delete_all` will continue to honor the `:dependent` option. However
    if `:dependent` value is `:destroy` then the default deletion
    strategy for that collection will be applied.

    User can also force a deletion strategy by passing parameter to
    `delete_all`. For example you can do `@post.comments.delete_all(:nullify)` .

    *Neeraj Singh*

*   Calling default_scope without a proc will now raise `ArgumentError`.

    *Neeraj Singh*

*   Removed deprecated method `type_cast_code` from Column.

    *Neeraj Singh*

*   Removed deprecated options `delete_sql` and `insert_sql` from HABTM
    association.

    Removed deprecated options `finder_sql` and `counter_sql` from
    collection association.

    *Neeraj Singh*

*   Remove deprecated `ActiveRecord::Base#connection` method.
    Make sure to access it via the class.

    *Yves Senn*

*   Remove deprecation warning for `auto_explain_threshold_in_seconds`.

    *Yves Senn*

*   Remove deprecated `:distinct` option from `Relation#count`.

    *Yves Senn*

*   Removed deprecated methods `partial_updates`, `partial_updates?` and
    `partial_updates=`.

    *Neeraj Singh*

*   Removed deprecated method `scoped`

    *Neeraj Singh*

*   Removed deprecated method `default_scopes?`

    *Neeraj Singh*

*   Remove implicit join references that were deprecated in 4.0.

    Example:

        # before with implicit joins
        Comment.where('posts.author_id' => 7)

        # after
        Comment.references(:posts).where('posts.author_id' => 7)

    *Yves Senn*

*   Apply default scope when joining associations. For example:

        class Post < ActiveRecord::Base
          default_scope -> { where published: true }
        end

        class Comment
          belongs_to :post
        end

    When calling `Comment.joins(:post)`, we expect to receive only
    comments on published posts, since that is the default scope for
    posts.

    Before this change, the default scope from `Post` was not applied,
    so we'd get comments on unpublished posts.

    *Jon Leighton*

*   Remove `activerecord-deprecated_finders` as a dependency

    *Łukasz Strzałkowski*

*   Remove Oracle / Sqlserver / Firebird database tasks that were deprecated in 4.0.

    *kennyj*

*   `find_each` now returns an `Enumerator` when called without a block, so that it
    can be chained with other `Enumerable` methods.

    *Ben Woosley*

*   `ActiveRecord::Result.each` now returns an `Enumerator` when called without
     a block, so that it can be chained with other `Enumerable` methods.

    *Ben Woosley*

*   Flatten merged join_values before building the joins.

    While joining_values special treatment is given to string values.
    By flattening the array it ensures that string values are detected
    as strings and not arrays.

    Fixes #10669.

    *Neeraj Singh and iwiznia*

*   Do not load all child records for inverse case.

    currently `post.comments.find(Comment.first.id)` would load all
    comments for the given post to set the inverse association.

    This has a huge performance penalty. Because if post has 100k
    records and all these 100k records would be loaded in memory
    even though the comment id was supplied.

    Fix is to use in-memory records only if loaded? is true. Otherwise
    load the records using full sql.

    Fixes #10509.

    *Neeraj Singh*

*   `inspect` on Active Record model classes does not initiate a
    new connection. This means that calling `inspect`, when the
    database is missing, will no longer raise an exception.
    Fixes #10936.

    Example:

        Author.inspect # => "Author(no database connection)"

    *Yves Senn*

*   Handle single quotes in PostgreSQL default column values.
    Fixes #10881.

    *Dylan Markow*

*   Log the sql that is actually sent to the database.

    If I have a query that produces sql
    `WHERE "users"."name" = 'a         b'` then in the log all the
    whitespace is being squeezed. So the sql that is printed in the
    log is `WHERE "users"."name" = 'a b'`.

    Do not squeeze whitespace out of sql queries. Fixes #10982.

    *Neeraj Singh*

*   Fixture setup does no longer depend on `ActiveRecord::Base.configurations`.
    This is relevant when `ENV["DATABASE_URL"]` is used in place of a `database.yml`.

    *Yves Senn*

*   Fix mysql2 adapter raises the correct exception when executing a query on a
    closed connection.

    *Yves Senn*

*   Ambiguous reflections are on :through relationships are no longer supported.
    For example, you need to change this:

        class Author < ActiveRecord::Base
          has_many :posts
          has_many :taggings, :through => :posts
        end

        class Post < ActiveRecord::Base
          has_one :tagging
          has_many :taggings
        end

        class Tagging < ActiveRecord::Base
        end

    To this:

        class Author < ActiveRecord::Base
          has_many :posts
          has_many :taggings, :through => :posts, :source => :tagging
        end

        class Post < ActiveRecord::Base
          has_one :tagging
          has_many :taggings
        end

        class Tagging < ActiveRecord::Base
        end

    *Aaron Patterson*

*   Remove column restrictions for `count`, let the database raise if the SQL is
    invalid. The previous behavior was untested and surprising for the user.
    Fixes #5554.

    Example:

        User.select("name, username").count
        # Before => SELECT count(*) FROM users
        # After => ActiveRecord::StatementInvalid

        # you can still use `count(:all)` to perform a query unrelated to the
        # selected columns
        User.select("name, username").count(:all) # => SELECT count(*) FROM users

    *Yves Senn*

*   Rails now automatically detects inverse associations. If you do not set the
    `:inverse_of` option on the association, then Active Record will guess the
    inverse association based on heuristics.

    Note that automatic inverse detection only works on `has_many`, `has_one`,
    and `belongs_to` associations. Extra options on the associations will
    also prevent the association's inverse from being found automatically.

    The automatic guessing of the inverse association uses a heuristic based
    on the name of the class, so it may not work for all associations,
    especially the ones with non-standard names.

    You can turn off the automatic detection of inverse associations by setting
    the `:inverse_of` option to `false` like so:

        class Taggable < ActiveRecord::Base
          belongs_to :tag, inverse_of: false
        end

    *John Wang*

*   Fix `add_column` with `array` option when using PostgreSQL. Fixes #10432

    *Adam Anderson*

*   Usage of `implicit_readonly` is being removed`. Please use `readonly` method
    explicitly to mark records as `readonly.
    Fixes #10615.

    Example:

        user = User.joins(:todos).select("users.*, todos.title as todos_title").readonly(true).first
        user.todos_title = 'clean pet'
        user.save! # will raise error

    *Yves Senn*

*   Fix the `:primary_key` option for `has_many` associations.
    Fixes #10693.

    *Yves Senn*

*   Fix bug where tiny types are incorrectly coerced as boolean when the length is more than 1.

    Fixes #10620.

    *Aaron Patterson*

*   Also support extensions in PostgreSQL 9.1. This feature has been supported since 9.1.

    *kennyj*

*   Deprecate `ConnectionAdapters::SchemaStatements#distinct`,
    as it is no longer used by internals.

    *Ben Woosley*

*   Fix pending migrations error when loading schema and `ActiveRecord::Base.table_name_prefix`
    is not blank.

    Call `assume_migrated_upto_version` on connection to prevent it from first
    being picked up in `method_missing`.

    In the base class, `Migration`, `method_missing` expects the argument to be a
    table name, and calls `proper_table_name` on the arguments before sending to
    `connection`. If `table_name_prefix` or `table_name_suffix` is used, the schema
    version changes to `prefix_version_suffix`, breaking `rake test:prepare`.

    Fixes #10411.

    *Kyle Stevens*

*   Method `read_attribute_before_type_cast` should accept input as symbol.

    *Neeraj Singh*

*   Confirm a record has not already been destroyed before decrementing counter cache.

    *Ben Tucker*

*   Fixed a bug in `ActiveRecord#sanitize_sql_hash_for_conditions` in which
    `self.class` is an argument to `PredicateBuilder#build_from_hash`
    causing `PredicateBuilder` to call non-existent method
    `Class#reflect_on_association`.

    *Zach Ohlgren*

*   While removing index if column option is missing then raise IrreversibleMigration exception.

    Following code should raise `IrreversibleMigration`. But the code was
    failing since options is an array and not a hash.

        def change
          change_table :users do |t|
            t.remove_index [:name, :email]
          end
        end

    Fix was to check if the options is a Hash before operating on it.

    Fixes #10419.

    *Neeraj Singh*

*   Do not overwrite manually built records during one-to-one nested attribute assignment

    For one-to-one nested associations, if you build the new (in-memory)
    child object yourself before assignment, then the NestedAttributes
    module will not overwrite it, e.g.:

        class Member < ActiveRecord::Base
          has_one :avatar
          accepts_nested_attributes_for :avatar

          def avatar
            super || build_avatar(width: 200)
          end
        end

        member = Member.new
        member.avatar_attributes = {icon: 'sad'}
        member.avatar.width # => 200

    *Olek Janiszewski*

*   fixes bug introduced by #3329. Now, when autosaving associations,
    deletions happen before inserts and saves. This prevents a 'duplicate
    unique value' database error that would occur if a record being created had
    the same value on a unique indexed field as that of a record being destroyed.

    *Johnny Holton*

*   Handle aliased attributes in ActiveRecord::Relation.

    When using symbol keys, ActiveRecord will now translate aliased attribute names to the actual column name used in the database:

    With the model

        class Topic
          alias_attribute :heading, :title
        end

    The call

        Topic.where(heading: 'The First Topic')

    should yield the same result as

        Topic.where(title: 'The First Topic')

    This also applies to ActiveRecord::Relation::Calculations calls such as `Model.sum(:aliased)` and `Model.pluck(:aliased)`.

    This will not work with SQL fragment strings like `Model.sum('DISTINCT aliased')`.

    *Godfrey Chan*

*   Mute `psql` output when running rake db:schema:load.

    *Godfrey Chan*

*   Trigger a save on `has_one association=(associate)` when the associate contents have changed.

    Fix #8856.

    *Chris Thompson*

*   Abort a rake task when missing db/structure.sql like `db:schema:load` task.

    *kennyj*

*   rake:db:test:prepare falls back to original environment after execution.

    *Slava Markevich*

Please check [4-0-stable](https://github.com/rails/rails/blob/4-0-stable/activerecord/CHANGELOG.md) for previous changes.
