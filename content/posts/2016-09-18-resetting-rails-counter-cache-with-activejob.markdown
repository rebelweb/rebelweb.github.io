---
title:  "Resetting Rails Counter Cache with Active Job"
date:   2016-09-18 12:30:00 -0500
draft: false
---

I have recently tried something a little different, when working with Rails
counter caches. For those new to rails counter cache columns are where you setup
a column to hold a count of a __has_many__ relationship to make a lookup faster
than a count SQL query. You can read more about that more about them in the API,
[Rails Associations](http://guides.rubyonrails.org/association_basics.html)

I tried setting up resetting counter caches in ActiveJob,
instead of using a rake task to do so. Use case for this would be when someone
updates the count from SQL or when you first implement the counts. This would
allow me to call the job from the admin interface of my application, I still can
call the job from a rake task if I needed to. Lets take a closer look:

I started thinking how to do this efficiently without minimal coding. I started
digging to see how to turn snake case into a class name, so I take something
like __category__ and turn it into __Category__. I know the generators that come
baked into Rails work this way, so I searched the Rails repository on github, to
see how it works and used it in this example.

The update script uses a JSON file to store the class and the relating
relationships that need updated. It is structured with the class name in snake
case as the key, and the value of an array of all the relationships needing
updated. See the example below.

```json
{
  "category": [
    "article_categories"
  ]
}
```

Now for the job code (see below). The job accepts one parameter which is the
key. The key is the key from the JSON file discussed earlier so you can do all
tables or just a single table, to allow for maximum flexibility, or concurrency
since we are using Active Job. To use concurrency simple spin each class in its
own job. The job loads the JSON file storing the configuration, and depending on
if a key was passed it will loop through all keys and columns or just the one
specified.

For the actual business end of things the __update_cache_columns__ method does
the brunt of the work. The method takes the key and turns it into a class_name,
and updates each one of it's cache columns.

```ruby
  class UpdateCounterCacheJob < ApplicationJob
    attr_accessor :cache_columns

    def perform(key = 'all')
      self.cache_columns = counter_cache_hash
      if key == 'all'
        cache_columns.keys.each { |obj| update_cache_columns(obj) }
      else
        update_cache_columns(key)
      end
    end

    private

    def counter_cache_hash
      file_name = format('%s/config/counter_cache_columns.json', Rails.root)
      file = File.read(file_name)
      JSON.parse(file)
    end

    def update_cache_columns(key)
      model = key.camelize.constantize.base_class
      cache_columns[key].each do |col|
        model.find_each { |m| model.reset_counters(m.id, col) }
      end
    end
  end
```

Testing this is easy. First, we'll create the related object and update
the count via raw SQL. Then we'll finally run the job and verify it updated the
count successfully.

```ruby
  RSpec.describe UpdateCounterCachesJob, type: :job do
    it 'updates all cache columns when they are out of sync' do
      sql = <<-SQL
      update categories
      SET articles_count = 5
      SQL

      category = create(:category)
      ActiveRecord::Base.connection.execute(sql)
      category.reload
      expect(category.articles_count).to eq(5)
      UpdateCounterCachesJob.perform_now
      category.reload
      expect(category.articles_count).to eq(0)
    end

    it 'updates an individual model\'s cache columns' do
      sql = <<-SQL
      update categories
      SET articles_count = 5
      SQL

      category = create(:category)
      ActiveRecord::Base.connection.execute(sql)
      category.reload
      expect(category.articles_count).to eq(5)
      UpdateCounterCachesJob.perform_now('category')
      category.reload
      expect(category.articles_count).to eq(0)
    end
  end
```

I am including my base model code, to help any one see how a counter cache is
setup. The category relation on ArticleCategory contains
__counter_cache: :articles_count__, which is what typically updates the column
in the category table every time one is created or destroyed. The job above is
for when the counts are wrong due to something going a rye.

```ruby
  class Category < ApplicationRecord
    has_many :article_categories, class_name: ArticleCategory,
                                  foreign_key: :category_id, dependent: :destroy
    has_many :articles, through: :article_categories
  end

  class Article < ApplicationRecord
    belongs_to :author, class_name: User, foreign_key: :author_id
    has_many :article_categories, class_name: ArticleCategory,
                                  foreign_key: :article_id, dependent: :destroy
    has_many :categories, through: :article_categories
    has_many :images, class_name: ArticleImage, foreign_key: :article_id,
                    dependent: :destroy
    end

  class ArticleCategory < ApplicationRecord
    belongs_to :article, class_name: Article, foreign_key: :article_id
    belongs_to :category, class_name: Category, foreign_key: :category_id,
                          counter_cache: :articles_count
  end
```

This is a different spin on how this is typically handled. I welcome any
thoughts on this implementation.
