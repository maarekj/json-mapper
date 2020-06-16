# Hamlet Framework / Json Mapper

[![Build Status](https://travis-ci.org/hamlet-framework/json-mapper.svg)](https://travis-ci.org/hamlet-framework/json-mapper)

To map the following JSON structure:

```json
[
    { "name": "Yuri" },
    { "name": "Oleg", "email": "oleg@example.com", "address": { "city": "Vologda" } }
]
```

into the following class hierarchy:

```php
<?php
 
class User
{
    /** @var string */
    private $name;

    /** @var string|null */
    private $email;

    /** @var Address|null */
    private $address;
}

class Address 
{
    /** @var string */
    private $city;
}
```

use:

```php
<?php

$users = JsonMapper::map(
    _list(_class(User::class)), 
    json_decode($data)
);
```

The library uses [hamlet-framework/type](https://github.com/hamlet-framework/type) library for type specifications.

### Configuration Options

The third parameter of the `JsonMapper::map` is `JsonMapperConfiguration` used to customize the mapping process.

### Json Property

```php
<?php

$configuration
    ->withDefaultValue(User::class, 'name', 'unknown')
    ->withJsonName(User::class, 'homeAddress', 'home_address', 'homeaddress')
    ->ingnoreUnknown(User::class);
```

### Using Setters

```php
<?php

$configuaration
    ->withPropertySetters(User::class)
    ->withPropertySetter(User::class, 'homeAddress', 'updateHomeAddress');
```

### Using Converter

```php
<?php

$configuration
    ->withConverter(User::class, 'time', function (int $unixtime) {
        return DateTimeImmutable::createFomFormat('U', (string) $unixtime);
    })
    ->withConverter(User::class, 'preferences', function (string $json) {
        return _map(_string(), _string())->cast(json_decode($json)); 
    })
    ->withConverter(self::class, 'email', function ($email) {
        return filter_var($email, FILTER_VALIDATE_EMAIL) ?: null;
    });
```

### Using Type Dispatcher 

```php
<?php

$configuration
    ->withTypeDispatcher(User::class, function ($properties) {
        if (isset($properties['name'])) {
            return NamedUser::class;
        } else {
            return AnonymousUser::class;
        }
    });
```

```php
<?php

$coniguration
    ->withTypeDispatcher(User::class, '__resolveType');
```

### Using JsonMapperAware interface

If you want to keep your mapping configuration closer to the files you map, there's an option to implement `JsonMapperAware` interface

```php
<?php

class Car implements JsonMapperAware
{
    /** @var string */
    protected $make;

    public function make(): string
    {
        return $this->make;
    }

    public static function configureJsonMapper(JsonMapperConfiguration $configuration): JsonMapperConfiguration
    {
        return $configuration
            ->withTypeResolver(self::class, function ($properties) {
                if (array_key_exists('machineGunCapacity', (array) $properties)) {
                    return JamesBondCar::class;
                } else {
                    return Car::class;
                }
            });
    }
}

$cars = JsonMapper::map(_list(_class(Car::class)), json_decode($payload));
```

## To do

- Add validators
- Add examples with psalm specs
- Use a better parser for type resolving
