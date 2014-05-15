# Query language

The Backbeam Query Language is the way you will make queries to the data stored in your database. It is a language similar to SQL but much simpler and with the ability to describe complex queries easily.

## Overview

First of all BQL is just for querying. You cannot modifiy the database structure or insert, delete or update information using BQL. With BQL you just query (read) data

Another difference with SQL is that you don't need to specify a "SELECT" or a "FROM" like in SQL. You define the entity to which you are querying in other ways. For example in JavaScript you do

```sql
backbeam.select(entity).query(your-bql-goes-here)
```

If you see a "select" somewhere in the documentation it is just to clarify an example but it is not a valid BQL query.

This is the general structure of a BQL query

```sql
[where <constraints>] [<joins>] [sort by <sort-field>]
```

And this is an example of query with a few options

```sql
[select news] where title like ? join last 5 comments having score > 10 fetch author sort by created_at
```

## Constraints

You can define multiple constraints in your BQL queries. You use the and and or operators and you can use parenthesis to group constraints. It is not recommended to use and and or in the same group, so better use parenthesis. It is not recommended because the precedence order is from right to left. For example in this query it is first calculated the or between the last two constraints and the result is applied to the and operator with the first constraint.

```sql
where <constraint1> and <constraint2> or <constraint3>
```

You can rewrite this query using parenthesis to change the precedence order

```sql
where (<constraint1> and <constraint2>) or <constraint3>
```

Constraints are usually composed by the name of a field followed by an operator and finish in the ? character. The ? acts as a placeholder character. This is done to prevent query injections. You pass the query parameters usually as extra arguments in the methods of the SDK you are using.

## Fields and relationships

You define the fields and relationships of your entities in the control panel of your project. You assign them a name and Bacbkeam automatically creates an identifier from that name by stripping some characters and replacing white spaces with dashes. For example you can create a field named "Last event" and Backbeam will give it the "last-event" identifier. The name is used in the control panel and you use the identifier in the API, SDKs and BQL queries. The same applies to entity names. Each entity has a name and Backbeam creates an identifier from that name. So you can have an entity named "Points of interest" and Backbeam will give it the identifier "points-of-interest".

There are two predefined entities (User and File) and each entity has also some predefined fields including ('created_at' and 'updated_at') that you can use in BQL queries.

## Operators

There are many operators available: `=`, `<`, `<=`, `>`, `>=`, `is`, `has`, `in`, `like`

* The `=` operator can be used for numbers, strings and dates
* The inequallity operators (`<`, `<=`, `>`, `>=`) can be used for fields containing numbers and dates
* The `like` operator can be used for fields containing strings (no matter if it is defined as text, textarea or richtextarea). This operator performs a full-text search. This behaviour differs from the `LIKE` operator in SQL
* The `is` operator is used in relationships "to-one"
* The `in` and has operators are used to define query constraints between "to-many" relationships

## Full-text searches

You can perform full-text searches in fields storing text (even fields defined as richtext) using the like operator. Note that this operator behaves different from the LIKE operator in SQL. It is more similar to the `MATCH - AGAINST` operator of MySQL

```javascript
backbeam.select(entity).query('where title like ?', 'text to search')
```

Backbeam automatically searches objects with those words (one or more) and the results are sorted by default by relevance (bigger density of those words in the text). Backbeam ignores some stop-words. It first tries to detect in which language the text is written and then removes the stop-words of that language. Languages such as Chinese, Japanese, and Korean are not well indexed because these languages don't have separation between words.

## Constraints over objects

You can perform basic constraints over "to-one" relationships using the is operator

```javascript
backbeam.select('places').query('where city is ?', someCity)
```

This query searches places whose city is the given city as parameter.

You can also perform queries over the "to-many" part of your relationships using the in and has operators.

For example, having a database with langauges and countries with a "many-to-many" relationship (each country has many languages and each language is spoken in many contries) you can do:

```javascript
backbeam.select('countries').query('where languages has ?', someLanguage)
```

This searches those countries having someLanguage in its "languages" relationship. Finally the use of in is similar. Suppose a database with cities and users with favoriteCities.

```javascript
backbeam.select('cities').query('where this in ?.favoriteCities', someUser)
```

This query searches cities in the "favoriteCities" of a given user. The in operator is the only operator different in their structure. The this keyword is mandatory and in the right side of the operator you define the name of an external relationships (in this case a relationship in the 'user' entity).

The in operator also supports a negative form: the not in operator. Additionally this operator supports 'collection constraints'. Learn more about [collection constraints and social queries](https://backbeam.io/article/introducing-collection-constraints-and-social-queries)

## Performing joins

You can perform a join over a "to-one" part of a relationship very easily. For example:

```javascript
backbeam.select('cities').query('join country')
```

You can also perform joins over any "to-many" side of a relationship. In these joins you usually define a number of results and a sort order with `first`/`last` that refers to the date in wich the object was added to the relationship. Some examples:

```javascript
// The number of cities in the relationship is returned.
// It only returns the counter.
// It doesn't return any object
backbeam.select('country').query('join cities')

// Returns the 5 last cities added to the relationship
backbeam.select('country').query('join last 5 cities')

// Returns the 5 first cities added to the relationship
backbeam.select('country').query('join first 5 cities')
```

If you don't specify a number of elements to join then no results are returned. But this could be useful because additionally you always get a number with the total number of objects in the relationship.

Additionally you can also specify constraints to the joins with the having keyword. For example

```javascript
backbeam.select('news').query('join last 5 comments having score > ?', someMinimumScore)
```

Finally you can perform joins with fetch. In the example above you could be interested not only in the comments but also in the author of the joined comments. It is possible and easy to do, just include one fetch

```javascript
backbeam.select('news').query('join last 5 comments having score > ? fetch author', someMinimumScore)
```

You can use several `fetch` in the same `join` separating them by commas.

## Sorting the results

You can specify a sort order in your BQL queries. This is very simple.

```javascript
backbeam.select('cities').query('join country sort by name asc')
```

You can specify a sort order, either 'asc' or 'desc' and it's 'desc' by default. If you don't specify a sort field the default is usually 'created_at'. However the default sorting could change in some circumstances. For example if you don't specify a sort field and you are making a 'like' constraint then de default sorting is by the relevance of the search results by that criteria. Geolocalizated queries also are sorted by distance by default.
