# Modeling Relationships in Rails

### Objectives
*After this lesson, students will be able to:*

- Build models with has_many, belongs_to, and has_many :through
- Describe macros that match different relationship types

### Preparation
*Before this lesson, students should already be able to:*

- Create models that inherit from ActiveRecord
- Explain and generate migrations
- Describe a relational database

## Why are relationships important?

As we think about designing applications we naturally come across objects that have relationships. Bands have members, 
Customers make orders. Stores have inventory.

ActiveRecord makes it easy to connect these logical relationships to a database. Previously we looked briefly at how to 
connect models with a many-to-one and one-to-one relationships.  We saw that we could assocaite artists to songs and then 
managers to songs through managers. 

To we're going to build on these relationships to work through how to build a many-to-many relationships. These relationships
require a bit more work than many-to-one relationships because of how relational database works. We'll work through how to 
create these relationships in ActiveRecord that can be used in Rails and Sinatra.

## Goal
We'll start by building the models in a blank rails app. We'll create a relationships between bands and musicians.

What our end goal? Our goal is that we'll be able to refer to ``bands.musicians`` and ``musicians.bands``. ``bands.musician``
will act similar to an array. We'll be able to add and remove new musicians to a a band and get information about a musician 
if I know a band. How do we do this?

### Create a baseline app

Inside this repo create a new Rails app called bands. Remember to set the database to Postgres. You don't have to include 
tests if you don't want to. 

We'll also add two models to the app.

Bands
- Name
- Formation Year

Musician
- First Name
- Last Name 
- Instrument

Need help, here's the [ActiveRecord Migration Rails Guide](http://edgeguides.rubyonrails.org/active_record_migrations.html).

## Creating a many-to-many relationship

If we just wanted bands to have many musicians then we'd have an one-to-many relationship. What would we need to do make 
bands have many musicians?

<details>
  We would need to do three things:
  1. Add ``has_many :musicians`` to Band model
  2. Add ``belongs_to :band`` to Musician model
  3. Create a migration that adds the band reference to the musician table.
</details>

But then we come to him

[] 

Where does he go! 

We have to enable a many-to-many relationship between bands and musicians. Rails offers two ways to create a many-to-many 
relationships. Here's the three step process

1. Add ``has_many_and_belongs_to_many :bands`` to the Musician model
2. Add ``has_many_and_belongs_to_many :musicians`` to the Band model
3. Create a migration that creates a new table to link the two models
    1. On the command line type in ``rails g migration create_bands_musicians``
    2. In the new migration add the following code

    ```ruby
       create_table :bands_musicians do |t|
          t.belongs_to :band
          t.belongs_to :musician
        end
    ```

Lets associate everything back together. Open the rail console and lets look at the following:

```
paul = Musician.create({first_name: 'Paul', last_name: 'McCartney', instrument: 'bass'})
beatles = Band.create({name: 'The Beatles', formation_year: 1960})
wings = Band.create({name: 'Wings', formation_year: 1971})
paul.bands << beatles
paul.bands << wings
paul.bands
the_mc = Musician.find_by first_name: 'Paul'
the_mc.bands
```


Rejoice

[]

Why do we need the extra table? Relational databases are defined so that each field can only hold one value. To include link 
multiple items, we need multiple records. For one-to-many relationships we basically piggy back on the many side of the 
relationship but for many-to-many relationships we need to store the information somewhere else.

The down-side of this method of working with many-to-many relationships is that if need to actually work with the link 
information it's very unintuitive.
 
## Many-to-Many

A second way to create to create that linking table yourself. It's very similar to the has many-through 

Consider a second system of many-to-many relationships. If we are talking customers and items we might have customer have 
many items by the link between them orders has importance in our application beyond just linking the customer to items.

In this case our setup might look like 
```ruby
class Customer < ActiveRecord::Base 
  has_many :orders
  has_many :items, through: :orders
end

class Item < ActiveRecord::Base
  has_many :orders
  has_many :customers, through: :orders
end

class Order < ActiveRecord::Base
  belongs_to :customer
  belongs_to :order
end

class CreateBands < ActiveRecord::Migration[5.0]
  def change
    create_table :customers do |t|
      t.string :name
      t.string :address 

      t.timestamps
    end

    create_table :items do |t|
      t.string :name
      t.string :address 

      t.timestamps    
    end

    create_table :orders do |t|
      t.belongs_to :item
      t.belongs_to :customer
      t.date :order_date

    end
  end
end

```


<!--
# SQL `JOIN`S

## Joins

Each table in a relational database is considered a relation. All of the table's data is naturally related by single set of attributes defined for it. However, in order to be relational, we need to be able to make queries among relations or tables of data.

**JOIN**s are our means of implementing queries that combine data and show results from multiple tables.

There are many kinds of joins, based on how you want to combine data.

![](https://raw.githubusercontent.com/sf-wdi-18/notes/master/lectures/week-07/day-1-intro-sql/dawn-simple-queries/images/join.png)

## Foreign Key

To implement a `JOIN` between two tables, one of our tables must have a **foreign key**. A foreign key is a field in one table that uniquely identifies a row of another table. We use the foreign key to **establish and enforce a link between the data in two tables**.

The foreign key always goes on the table with the data that belongs to data from another table. In the example below, a person **has_many** pets, and a pet **belongs_to** a person. The foreign key `person_id` goes on the `pets` table to indicate which person the pet belongs to.

![](https://raw.githubusercontent.com/sf-wdi-18/notes/master/lectures/week-07/day-1-intro-sql/dawn-simple-queries/images/primary_foreign_key.png)


## Migration Workflow

Getting your models and tables synced up is a bit tricky. Pay close attention to the following workflow, especially the rake tasks.

```
# create a new rails app
rails new my_app -d postgresql
cd my_app

# create the database
rake db:create

# REPEAT THESE TASKS FOR EVERY CHANGE TO YOUR DATABASE
# <<< BEGIN WORKFLOW LOOP >>>

# -- IF YOU NEED A NEW MODEL --
# auto-generate a new model (AND automatically creates a new migration)
rails g model Pet name:string
rails g model Owner name:string

# --- OTHERWISE ---

# if you only need to change fields in an *existing* model,
# you can just generate a new migration
rails g migration AddAgeToOwner age:integer

# never try to create a migration file yourself through the file system! it's really hard to get the name right!

# -- EITHER WAY --
### whether we're creating a new model or updating an existing one, we can manually edit our models and migrations in sublime
# update associations in model  this affects model interface
# update foreign keys in migrations this affects database tables

# generate schema for database tables
rake db:migrate

# <<< END LOOP >>>

# finally, we need some data to play with
# for now, we'll seed it manually, from the rails console...
rails c
> Pet.create(name: "Wowzer")
> Pet.create(name: "Rufus")

# --- OR ---

# but later we will run a seed task
rake db:seed
```

# Many-to-Many Challenges

Our goal is to build the relationship between `actors` and `movies`. An actor can appear in many movies, and a movie can have many actors. How would you set up this relationship? Is there an additional data table we need besides `actors` and `movies`? **Hint:** A *join* table has two different foreign keys, one for each model it is associating. 


Here's what our models' attributes might look like for actors and movies:
  * `Actor`: first_name, last_name
  * `Movie`: title, description, year

For these challenges, continue to work in your `practice` Rails app.

## Your Task

1. Create models and migrations for three tables: `actors`, `movies`, and a *join* table. Think about what you should name your join table and what columns it should have.
2. Implement a many-to-many relationship between `actors` and `movies`.
3. Use the Rails console to create at least three `actors` and two `movies`. Each movie should have at least one actor associated with it. 

## Stretch Challenges

1. Add <a href="http://guides.rubyonrails.org/active_record_validations.html" target="_blank">validations</a> to your `Actor` and `Movie` models:
  * All attributes for actors and movies should be required (**Hint:** `presence: true`)
  * For movies, the year should not be in the future (**Hint:** Look at <a href="http://guides.rubyonrails.org/active_record_validations.html#numericality" target="_blank">numericality</a>)

2. Test your validations in the Rails console:

  ```ruby
  a = Actor.create
  a.errors.messages
  # => What does this return?
  ```

## Stretch Challenge: Self-Referencing Assocations

Lots of real-world apps create assocations between items that are the same type of resource.  Read (or reread) <a href="http://guides.rubyonrails.org/association_basics.html#self-joins" target="_blank">the "self joins" section of the Associations Basics Rails Guide</a> and try to create a self-referencing association in your `practice_associations` app.  (Classic use cases are friends and following, where both related resources would be users.) No solution provided.

-->
## Helpful Hints

When you're **creating associations** in Rails ActiveRecord (or most any ORM, for that matter):

  * Define the relationships in your models (the blueprint for your objects)
    * Don't forget to define all sides of the relationship (e.g. `has_many`, `belongs_to`, `has_many_and_belongs_to_many`)
  * Remember to put the relationship in your migration
    * If you're not sure which side of the relationship has the foreign key, just use this simple rule: the model with `belongs_to` must include a foreign key.

## Less Common Associations

These are for your references and are not used nearly as often as `has_many` and `has_many through`.

  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-association" target="_blank">has_one</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#the-has-one-through-association" target="_blank">has_one through</a>
  * <a href="http://guides.rubyonrails.org/association_basics.html#has-and-belongs-to-many-association-reference" target="_blank">has_and_belongs_to_many</a>

## Useful Docs

* <a href="http://guides.rubyonrails.org/association_basics.html" target="_blank">Associations Rails Guide</a>
* <a href="http://edgeguides.rubyonrails.org/active_record_migrations.html" target="_blank">Migrations Rails Guide</a>


## Licensing
All content is licensed under a CC­BY­NC­SA 4.0 license.
All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.