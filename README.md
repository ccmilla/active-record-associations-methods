# Active Record Association Methods

## Learning Goals

- Understand the common methods we have access to from our Active Record
  associations
- Use the methods that Active Record gives you based on your associations

## Introduction

Previously, we learned what Active Record associations are and how to use them.
In this lab, we are going to start with the association relationships already
coded for `Songs`, `Genres`, and `Artists`. These associations look like this:

- An artist has many songs
- An artist has many genres through songs.
- A song belongs to a genre.
- A song belongs to a artist.
- A genre has many songs.
- A genre has many artists through songs.

Here's what our Entity Relationship Diagram looks like:

![Playlister ERD](https://curriculum-content.s3.amazonaws.com/phase-3/active-record-associations-and-migrations/artists-songs-genres.png)

In the previous lessons, we had a similar set of associations between games,
reviews and users, where the reviews table joined between our users and games.

While the models are different for this lab, our approach will be the same. By
writing a few migrations and making use of the appropriate Active Record macros,
we will be able to:

- ask an Artist about its songs and genres
- ask a Song about its genre and its artist
- ask a Genre about its songs and artists

We will build these associations through the use of Active Record migrations and
macros.

### Building our Migrations

You can take a look at the migrations to see how our tables are structured. Pay
special attention to the migration for the `songs` table — this is the table
that joins between our `artists` and `genres` tables.

To run the migrations, run:

```console
$ bundle exec rake db:migrate
```

### Building our Associations using Active Record Macros

In the previous lesson, we used the following Active Record macros (or methods):
[has_many][], [has_many through][], [belongs_to][]. They helped us associate
`games`, `reviews`, and `users`.

[has_many]: http://guides.rubyonrails.org/association_basics.html#the-has-many-association
[has_many through]: http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association
[belongs_to]: http://guides.rubyonrails.org/association_basics.html#the-belongs-to-association

Here's how they look for our new domain:

```rb
class Song < ActiveRecord::Base
  belongs_to :artist
  belongs_to :genre
end
```

```rb
class Artist < ActiveRecord::Base
  has_many :songs
  has_many :genres, through: :songs
end
```

```rb
class Genre < ActiveRecord::Base
  has_many :songs
  has_many :artists, through: :songs
end
```

And that's it! With this relatively small amount of code, we now have access to
a whole host of methods provided by Active Record.

## Association Methods

Go ahead and run the test suite and you'll see that we are passing the first 14
tests. Our associations are all working, just because of our migrations and use
of macros.

We can now call methods on the objects we associated with one another. Let's
play around with our code using the console task we wrote for you in the
`Rakefile`. Run the migrations (if you haven't already):

```console
$ bundle exec rake db:migrate
```

Then seed the database:

```console
$ bundle exec rake db:seed
```

Then open the console:

```console
$ bundle exec rake console
```

Then try out some methods:

```rb
hello = Song.create(name: "Hello")
# => #<Song:0x007fc75a8de3d8 id: 1, name: "Hello", artist_id: nil, genre_id: nil>
adele = Artist.create(name: "Adele")
# => #<Artist:0x007fc75b8d9490 id: 1, name: "Adele">
```

So, we know that an individual song has an `artist_id` attribute. We _could_
associate `hello` to `adele` by setting `hello.artist_id=` equal to the `id` of
the `adele` object. BUT! Active Record makes it so easy for us. The macros we
implemented in our classes allow us to associate a song object directly to an
artist object:

```rb
hello.artist = adele
# => #<Artist:0x007fc75b8d9490 id: 1, name: "Adele">
```

Now, we can ask `hello` who its artist is:

```rb
hello.artist
# => #<Artist:0x007fc75b8d9490 id: 1, name: "Adele">
```

We can even chain methods to ask `hello` for the _name_ of its artist:

```rb
hello.artist.name
# => "Adele"
```

We can tell the artist about their song:

```rb
rolling_in_the_deep = Song.create(name: "Rolling in the Deep")
# => #<Song:0x007fc75bb4d1e0 id: 2, name: "Rolling in the Deep", artist_id: nil, genre_id: nil>
```

```rb
adele.songs << rolling_in_the_deep
# => #[ <Song:0x007fc75bb4d1e0 id: 2, name: "Rolling in the Deep", artist_id: 1, genre_id: nil> ]

rolling_in_the_deep.artist
# => #<Artist:0x007fc75b8d9490 id: 1, name: "Adele">
```

## Starting the Lab

We are going to write some methods of our own. We want to take advantage of the
instance methods provided by the Active Record macros. Therefore, every method
we write will use some code that was generated by a macro. For example:

```rb
class Artist
  has_many :songs
  has_many :genres, through: :songs

  def get_first_song

  end
end
```

How would you write the `#get_first_song` method so that it returns the first
song saved to the artist it's called on? By using the macros! Just like
above when we called `adele.songs`, we now want to call `#songs` on the instance
that the method will be called on in the future. How do we do that? Yes, `self`!

```rb
class Artist
  has_many :songs
  has_many :genres, through: :songs

  def get_first_song
    self.songs
  end
end
```

Recall that using `has_many :songs` creates an instance method `#songs` that we
can call on any instance of an artist.

Calling this method now will return an array of the artist's songs. Since our
method is specifically looking for the first song, we just have to chain on a
`#first`.

```rb
class Artist
  has_many :songs
  has_many :genres, through: :songs

  def get_first_song
    self.songs.first
  end
end
```

We'll do a handful of methods like this one for the `Song`, `Artist`, and
`Genre` classes. This lab is test-driven, so follow the specs and read the test
error messages for additional information.

The below methods are defined in the `artist.rb`, `genre.rb` and `song.rb` files
within `app/models`, but are all currently empty. Write implementations for each
using ActiveRecord methods.

### Artist Methods

#### `#get_genre_of_first_song`

Returns the genre of the artist's first saved song (maybe the `#get_first_song`
method can be used here?).

#### `#song_count`

Return the total number of songs associated with the artist.

#### `#genre_count`

Return the total number of genres associated with the artist.

### Genre Methods

#### `#song_count`

Return the total number of songs associated with the genre.

#### `#artist_count`

Return the number of artists associated with the genre.

#### `#all_artist_names`

Return an array of strings containing every artist's name.

### Song Methods

#### `#get_genre_name`

Return the name of the genre this song belongs to.

#### `#drake_made_this`

For the final method in this lab, rather than return a specific value or set of
values like the previous labs, your task is to create an association between a
song and an artist. In this case, we'll use one artist for simplicity — Drake.

When this method is called, it should assign the song's artist to Drake. Drake
doesn't exist in the database as an artist yet, so you'll have to create a
record for him. However, if this method is run multiple times, you won't want to
create a new record _each time_. Rather, you only want to create a record if
Drake is not found in the database already. Once found or created, assign this
song to the drake Artist instance.

**Hint**: Look into the `.find_or_create_by` Active Record method!
# active-record-associations-methods
