# Autosuggest

Generate autocomplete suggestions based on what your users search

:tangerine: Battle-tested at [Instacart](https://www.instacart.com/opensource)

[![Build Status](https://github.com/ankane/autosuggest/workflows/build/badge.svg?branch=master)](https://github.com/ankane/autosuggest/actions)

## Installation

Add this line to your application’s Gemfile:

```ruby
gem 'autosuggest'
```

## Getting Started

#### Prepare your data

Start with a hash of queries and their popularity, like the number of users who have searched it.

```ruby
top_queries = {
  "bananas" => 353,
  "apples"  => 213,
  "oranges" => 140
}
```

With [Searchjoy](https://github.com/ankane/searchjoy), you can do:

```ruby
top_queries = Searchjoy::Search.group(:normalized_query)
  .having("COUNT(DISTINCT user_id) >= 5").distinct.count(:user_id)
```

Then pass them to Autosuggest.

```ruby
autosuggest = Autosuggest.new(top_queries)
```

#### Filter duplicates

[Stemming](https://en.wikipedia.org/wiki/Stemming) is used to detect duplicates like `apple` and `apples`.

The most popular query is preferred by default.  To override this, use:

```ruby
autosuggest.prefer ["apples"]
```

To fix false positives, use:

```ruby
autosuggest.not_duplicates [["straws", "straus"]]
```

#### Filter misspellings

We tried open-source libraries like [Aspell](http://aspell.net) and [Hunspell](http://hunspell.sourceforge.net/) but quickly realized we needed to build a corpus specific to our application.

There are two ways to build the corpus, which can be used together.

1. Add words

  ```ruby
  autosuggest.parse_words Product.pluck(:name)
  ```

  Use the `min` option to only add words that appear multiple times.

2. Add concepts

  ```ruby
  autosuggest.add_concept "brand", Brand.pluck(:name)
  ```

#### Filter words

[Profanity](https://github.com/tjackiw/obscenity/blob/master/config/blacklist.yml) is blocked by default. Add custom words with:

```ruby
autosuggest.block_words ["boom"]
```

#### Profit

Get suggestions with:

```ruby
autosuggest.suggestions
```

Filter queries without results and you’re set. We also prefer to have someone manually approve them by hand.

## Full Example

```ruby
top_queries = Searchjoy::Search.group(:normalized_query)
  .having("COUNT(DISTINCT user_id) >= 5").distinct.count(:user_id)
product_names = Product.pluck(:name)
brand_names = Brand.pluck(:name)

autosuggest = Autosuggest.new(top_queries)
autosuggest.parse_words product_names
autosuggest.add_concept "brand", brand_names
autosuggest.prefer brand_names
autosuggest.not_duplicates [["straws", "straus"]]
autosuggest.block_words ["boom"]

puts autosuggest.pretty_suggestions
# or
suggestions = autosuggest.suggestions.reject { |s| s[:duplicate] || s[:misspelling] || s[:profane] || s[:blocked] }
```

## History

View the [changelog](https://github.com/ankane/autosuggest/blob/master/CHANGELOG.md)

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/autosuggest/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/autosuggest/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features

To get started with development:

```sh
git clone https://github.com/ankane/autosuggest.git
cd autosuggest
bundle install
bundle exec rake test
```
