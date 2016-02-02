#Week 4 Day 2

##Review of Day 1 Rails

####Controller
Delegates model and view
- There will be multiple controllers

####ORM
Object Relational Mapping
Active Record
- Create ruby object, accessors, and methods to access queries
Data Mapper is an alternative
- Model is singular, table is plural
```
bin/rails g model question title:string (varChar 255 by default) body:text
```
Creates a model called 'question', a title attribute, and a body attribute

- Executing migration
```
bin/rake db:migrate
```
- Look for migrations that have not been executed yet
- Do it once is enough
- pgadmin
plug -> hostname:127.0.0.1 -> kayu
- Rollback
```
bin/rake db:Rollback
```
- New migration without model
```
bin/rails g migration add_view_count_to_questions
```
```
class AddViewCountToQuestions < ActiveRecord::Migration
  def change
    add_column :questions, :view_count, :integer
  end
end

bin/rake db:migrate
```
####Rails Console with ORM/Active Records
```
group :development do
  # Access an IRB console on exception pages or by using <%= console %> in views
  gem 'web-console', '~> 2.0'

  # These gems are used for nicer display in the Rails console
  gem "awesome_print"
  gem "interactive_editor"
  gem "hirb"
end

bin/rails c
```
```
q = Question.new
q.class
q.title = "My first question"
q.persisted?
q.save
q1 = Question.new({title: "Hello world", body: "What's up!", view_count: 1})
Save automatically, if hash is the last parameter: you don't need to use brackets
q2 = Question.create title: "my second question", body: "my second question body", view_count: 10
```
```
Question.all
question = Question.find 3 (id)
Exception when id is not found

Question.first
Question.last
ordered by id from ASC, DESC

q = Question.find_by_title "abc"
No exception if not found, returns nil
Question.find_by_body "My first question body"

Question.select(:id, :title)
Specific columns

Question.where()
Question.where({view_count: 0})
Question.where("view_count > 0")
Question.where("view_count > 0").each {|x| puts x.title}
Question.where({view_count: 0, title: "abc"})
Question.where(["created_at > ?",5.days.ago])
5.minutes.ago
2.years.ago
Question.where(["created_at > ? OR view_count = 0",5.minutes.ago])

Question.where(["title ILIKE ? OR body ILIKE ?", "%my%", "%my%"])
%my
my%
%my%
```
```
100.times {Question.create(title: Faker::Company.bs, body: Faker::Lorem.paragraph, view_count: rand(10))}
Question.limit(10)
Question.limit(10).offset(10)
```
- Update
```
q = Question.find 10
q.body = "new question body"
q.save
q.persisted?
Updates because id already exists
q.update({body: "some new question body"})
```
- Delete
```
.destroy
.delete skips call backs, and may nullify/delete associations
```
- Create index

```
bin/rails g migration add_indicies_to_questions
```
```ruby
class AddIndiciesToQuestions < ActiveRecord::Migration
  def change
    # this will ad an index (not unique) to questions table on the title
    # column

    add_index :questions, :title
    add_index :questions, :body

    # This creates a unique index
    # add_index :questions, :body, unique: true

    # To create a composite index you can do
    # add_index :questions, [:title, :body]
  end
end
```
- Binary Tree?
- Seeds
```ruby
seeds.rb

100.times do |variable|
  Question.create title: Faker::Company.bs,
                  body:  Faker::Lorem.paragraph,
                  view_count:rand(100)
end
```
When running up for the first time so DB is not empty

---
##Validation
Avoid doing validations in controllers
- Instead do it at the model layer
```ruby
class Question < ActiveRecord::Base
  # this will fail validations (so it won't create or save) if the title is
  # not provided
  validates :title, presence: true
```
```
reload!
true
q = Question.new
q.save
q.valid?
q.errors
q.errors.full_messages
```
You can use multiple validations
```ruby
validates :title, presence: true, uniqueness: true { case_sensitive: false }

validates :body, uniqueness: {message: "must be unique!"}
```
```ruby
# this validates that the combination of the title and body is
# unique. Which means the title doesn't have to be unique by itself
# and the body doesn't have to be unique by itself. However, the
# combination of the two fields must be unique
# validates :title, uniqueness: {scope: :body}
```
```ruby
validates :view_count, numericality: true
validates :view_count, numericality: {greater_than_or_equal_to: 0}
validates :email,
          format: { with: /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i }
```
- Custom validators
```ruby
# This is using custom validation method. We must make sure that
# 'no monkey' is a method available for our class. The method can
# be public or private. It's more likely we will have it as a private
# method because we don't need to use it outside of the class
validate :no_monkey

private
def no_monkey
  if title && title.downcase.include? ("monkey")
    errors.add(:title, "No monkey!")
  end
end
```
- Bang in Rails
```
q.save!
```
Gives exception rather than true and false

####Callback
```ruby
after_initialize :set_defaults
before_save :capitalize_title

private

def set_defaults
  self.view_count ||= 0
end

def capitalize_title
  self.title.capitalize!
end
```

####Scopes
```ruby
# scope :recent, lambda { order("created_at DESC").limit(5) }
def self.recent(number = 5)
  order("created_at DESC").limit(number)
end

def self.popular
  where("view_count > 10")
end
```
Use lambda when it's short and simple, function
code is more complex
