# Enum Type Definition
Enumeration types are a special kind of scalar that is restricted to a particular set 
of allowed values. 

In **graphql-php** enum type is an instance of `GraphQL\Type\Definition\EnumType` 
(or one of it subclasses) which accepts configuration array in constructor:

```php
use GraphQL\Type\Definition\EnumType;

$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => [
        'NEWHOPE' => [
            'value' => 4,
            'description' => 'Released in 1977.'
        ],
        'EMPIRE' => [
            'value' => 5,
            'description' => 'Released in 1980.'
        ],
        'JEDI' => [
            'value' => 6,
            'description' => 'Released in 1983.'
        ],
    ]
]);
```

This example uses **inline** style for Enum Type definition, but there are also 
[other styles](/type-system/#type-definition-styles) (using inheritance or composition).

# Configuration options
Enum Type constructor accepts array with following options:

Option | Type | Notes
------ | ---- | -----
name | `string` | **Required.** Name of the type. When not set - inferred from array key (read about [shorthand field definition](#) below)
description | `string` | Plain-text description of the type for clients (e.g. used by [GraphiQL](https://github.com/graphql/graphiql) for auto-generated documentation)
values | `array` | List of enumerated items, see below for expected structure of each entry

Each entry of **values** array in turn accepts following options:

Option | Type | Notes
------ | ---- | -----
name | `string` | **Required.** Name of the item. When not set - inferred from array key (read about [shorthand field definition](#) below)
value | `mixed` | Internal representation of enum item in your application (could be any value, including complex objects or callbacks)
description | `string` | Plain-text description of enum value for clients (e.g. used by [GraphiQL](https://github.com/graphql/graphiql) for auto-generated documentation)
deprecationReason | `string` | Text describing why this enum value is deprecated. When not empty - item will not be returned by introspection queries (unless forced)


# Shorthand definitions
If internal representation of enumerated item is the same as item name, then you can use
following shorthand for definition:

```php
$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => ['NEWHOPE', 'EMPIRE', 'JEDI']
]);
```

which is equivalent of:
```php
$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => [
        'NEWHOPE' => 'NEWHOPE', 
        'EMPIRE' => 'EMPIRE', 
        'JEDI' => 'JEDI'
    ]
]);
```

which is in turn equivalent of:
```php
$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => [
        'NEWHOPE' => ['value' => 'NEWHOPE'], 
        'EMPIRE' => ['value' => 'EMPIRE'], 
        'JEDI' => ['value' => 'JEDI']
    ]
]);
```

which is in turn equivalent of full form:

```php
$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => [
        ['name' => 'NEWHOPE', 'value' => 'NEWHOPE'], 
        ['name' => 'EMPIRE', 'value' => 'EMPIRE'], 
        ['name' => 'JEDI', 'value' => 'JEDI']
    ]
]);
```

# Field Resolution
When object field is of Enum Type, field resolver is expected to return internal 
representation of corresponding Enum item (**value** in config). **graphql-php** will 
then serialize this **value** to **name** to include in response:

```php
use GraphQL\Type\Definition\EnumType;
use GraphQL\Type\Definition\ObjectType;

$episodeEnum = new EnumType([
    'name' => 'Episode',
    'description' => 'One of the films in the Star Wars Trilogy',
    'values' => [
        'NEWHOPE' => [
            'value' => 4,
            'description' => 'Released in 1977.'
        ],
        'EMPIRE' => [
            'value' => 5,
            'description' => 'Released in 1980.'
        ],
        'JEDI' => [
            'value' => 6,
            'description' => 'Released in 1983.'
        ],
    ]
]);

$heroType = new ObjectType([
    'name' => 'Hero',
    'fields' => [
        'appearsIn' => [
            'type' => $episodeEnum,
            'resolve' => function() {
                return 5; // Actual entry in response will be 'appearsIn' => 'EMPIRE'
            }
        ]
    ]
])
```

Reverse is true when enum is used as input type (e.g. as field argument). 
GraphQL will treat enum input as **name** and convert it into **value** before passing to your app.

For example, given object type definition:
```php
$heroType = new ObjectType([
    'name' => 'Hero',
    'fields' => [
        'appearsIn' => [
            'type' => Type::boolean(),
            'args' => [
                'episode' => Type::nonNull($enumType)
            ]
            'resolve' => function($_value, $args) {
                return $args['episode'] === 5 ? true : false; 
            }
        ]
    ]
])
```

Then following query:
```
fragment on Hero {
    appearsInNewHope: appearsIn(NEWHOPE)
    appearsInEmpire: appearsIn(EMPIRE)
}
```
will return:
```php
[
    'appearsInNewHope' => false,
    'appearsInEmpire' => true
]
```