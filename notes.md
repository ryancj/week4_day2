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
