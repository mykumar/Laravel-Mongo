Library is not yet ready

Mongo Model for Laravel
=======================

This is Mongo Model for Laravel with many special features. For now it doesn't work as part of original Laravel models, but it's planned in future. Also this model has admin tool that provides handy and fast work with collections data.

Main idea is to let you write applications very fast and with minimum code.

Adventages:
* Model has strict scheme
* Easy multi-language support
* Each attribute can be a node with sub-attrbites
* Ability to work with attributes as with arrays
* Comfortable admin tool that generates itself automatically according to models scheme
* For configuration uses standart [Jenssegers\Mongodb library](https://github.com/jenssegers/laravel-mongodb/blob/master/README.md)

Current disadventages:
* Model has strict scheme
* Not part of standart Laravel models and has different interface
* Attribute '_id' is always a primaty key
* Each attribute is stored in collection as array (imposes restrictions on the multikey indexes)

Installation
------------

Make sure you have the MongoDB PHP driver installed. You can find installation instructions at http://php.net/manual/en/mongo.installation.php

Library requires standart [Jenssegers\Mongodb library](https://github.com/jenssegers/laravel-mongodb/blob/master/README.md). 
You can install it using composer:

```
composer require jenssegers/mongodb
```

Next step install the latest stable version using composer:

```
composer require masteryuri/laravel-mongomodel
```

### Version Compatibility

Library tested on Laravel 5.2.x

Configuration
-------------

Requires standart configuration of [Jenssegers\Mongodb library](https://github.com/jenssegers/laravel-mongodb/blob/master/README.md)

Model
-----

This package includes a MongoDB model class that you can use to define models for corresponding collections.

```php
use MasterYuri\Laravel\Mongodb\Model as Model;

class User extends Model {}
```
Note that we do not configure collection to use for the `User` model. Just like the original Eloquent, the lower-case, plural name of the class will be used as the table name unless another name is explicitly specified. You may specify a custom collection (alias for table) by defining a `collection` property on your model:

```php
use MasterYuri\LaravelMongo\Model as Model;

class User extends Model {

    protected $collection = 'users_collection';

}
```

Likewise, you may define a `connection` property to override the name of the database connection that should be used when utilizing the model.

```php
use MasterYuri\LaravelMongo\Model as Model;

class User extends Model {

    protected $connection = 'mongodb';

}
```

Also you can set collection and connection in init:

```php
use MasterYuri\LaravelMongo\Model as Model;

class User extends Model {

    protected function init()
    {
        $this->connection = 'mongodb';
        $this->collection = 'users_collection';
    }
}
```

### Init schema

Use `init()` method to declare model's schema:

```php
use MasterYuri\LaravelMongo\Model as Model;

define('GENDER_MALE',   'm');
define('GENDER_FEMALE', 'f');

class User extends Model {

    protected function init()
    {
        $this->initAttr('name',   self::TYPE_STRING,  'user.name');
        $this->initAttr('about',  self::TYPE_TEXT,    'user.about');
        $this->initAttr('rating', self::TYPE_INTEGER, 'user.rating');
		
        $this->initAttr('photo',  self::TYPE_IMAGE,   'user.photo');
		$this->initAttrCount('photo', 12); // For admin sector
		
        $this->initAttr('geo',     self::TYPE_NODE,  'user.geo');
        $this->initAttr('geo.lat', self::TYPE_FLOAT, 'user.geo.lat');
        $this->initAttr('geo.lng', self::TYPE_FLOAT, 'user.geo.lng');
		
        $this->initAttr('gender',  self::TYPE_ENUM,  'user.genger');
        $this->initAttrEnum('gender', GENDER_MALE,   'user.genger.m');
        $this->initAttrEnum('gender', GENDER_FEMALE, 'user.genger.f');
		
		$this->initModelTitle('user-model.model-name'); // For admin sector
    }
}
```

If in oject you try to get non-declared attribute you will get Exception.

Every attribute can be multi-language:

```php
use MasterYuri\LaravelMongo\Model as Model;

class Article extends Model {

    protected function init()
    {
        $this->initAttrMultiLang('title', self::TYPE_STRING, 'user-model.name');
        $this->initAttrMultiLang('desc',  self::TYPE_TEXT,   'user-model.about');
    }
}
```

It means that depending of current locale it will use correct value:

```php
$article = new Article();
App::setLocale('en');
$article->title = "About Longon";
App::setLocale('ru');
$article->title = "О Лондоне";
$user->save(); // Not required here

App::setLocale('en');
echo $article->title; // Gives "About Longon"
App::setLocale('ru');
echo $article->title; // Gives "О Лондоне"

App::setLocale('ru');
$article->setLocale('en'); // Force use inner locale
echo $article->title; // Gives "About Longon"

App::setLocale('ru');
$article->setLocale(null); // Reset inner locale and use App::getLocale()
echo $article->title; // Gives "О Лондоне"
```

If you have many level inheritage do not forget to call parent `init()` first:

```php

class UserEmployee extends User {

    protected function init()
    {
		parent::init();
        $this->initAttr('resume', self::TYPE_RICHTEXT);
    }
}
```

##### Available methods:

* initAttr($name, $type, $title = NULL) - init attribute
* initAttrMultiLang($name, $type, $title = NULL) - init multi language attribute
* initAttrCount($name, $count) - init count of attribute for admin tool
* initAttrEnum($name, $value, $enumTitle = NULL)  - init value for TYPE_ENUM
* initModelTitle($modelTitle) - init model title for admin tool

You can skip parameter $title. Model will try to get name of attribute by algoritm:

tolowercase(MODEL_NAME) + "." + str_replace("_", "-", ATTR_NAME)

##### Base data types

Base type are basement for all other types. They have different implementation of storing in base and way to write in base.
If you do not plan to use admin tool and `formatted()' method you can use only base types.

 Constant         | Description
:-----------------|:-----------
 TYPE_STRING      | String.
 TYPE_INTEGER     | Integer.
 TYPE_FLOAT       | Float.
 TYPE_MONGOID     | Mongo ID.
 TYPE_INT_AUTOINC | Autoincremental integer.
 TYPE_NODE        | Type to declare node attribute.

##### Extended data types
 
Extended types are added for better use in Admin Tool and for correct `formatted()` method support:
Also you must take into account base type when use `find` functions
Patterns for formatted output are declared in localization files.

 Constant          | Corresponding Base Type | Formatted output example | Description
:------------------|:------------------------|:-------------------------|:-----------
 TYPE_ENUM         | TYPE_STRING             | (as declared in scheme)  | Enumeration. Simple string. Described in next part. Select box in admin section.
 TYPE_BOOL         | TYPE_INTEGER            | "Yes"                    | Boolean. If false not saved in base. If true is equal to 1. Checkbox in admin section.
 TYPE_DATE         | TYPE_STRING             | "01.03.1998"             | Date. Stored in base in format "Y-m-d". Has date picker in admin section.
 TYPE_TIME         | TYPE_STRING             | "11:40"                  | Time. Stored in base in format "H:i". Has time picker in admin section.
 TYPE_COLOR        | TYPE_STRING             | "#FF0000"                | Color. Stored in base in format "#RRGGBB". Has color picker in admin section.
 TYPE_TEXT         | TYPE_STRING             | (as stored)              | Text. Has text area in admin section.
 TYPE_RICHTEXT     | TYPE_STRING             | (as stored)              | Text. Has rich text editor in admin section.
 TYPE_IMAGE        | TYPE_STRING             | "abcdfskfhs.jpg"         | Name of image file. Has file picker and preview in admin section.
 TYPE_FILE         | TYPE_STRING             | "abcdfskfhg.pdf"         | Name of file. Has file picker in admin section.
 TYPE_UDATE        | TYPE_INTEGER            | "01.03.1998"             | Unix time stump based on seconds since standard epoch of 1/1/1970. Has date picker in admin section.
 TYPE_UDATETIME    | TYPE_INTEGER            | "01.03.1998 11:40"       | Unix time stump based on seconds since standard epoch of 1/1/1970. Has date and time picker in admin section.
 TYPE_UFDATE       | TYPE_FLOAT              | "01.03.1998"             | Unix time stump based on seconds since standard epoch of 1/1/1970. Has date picker in admin section.
 TYPE_UFDATETIME   | TYPE_FLOAT              | "01.03.1998 11:40"       | Unix time stump based on seconds since standard epoch of 1/1/1970. Has date and time picker in admin section.

### Saving empty values

!!! Importatn: attributes that equal to 0 or empty string are not saved in base. You must take it into account when use `find` functions:

```php
$user = new User();
$user->rating = 0;
$user->save();

echo User::find(["rating" => 0])->count(); // Gives 0
echo User::find(["rating" => ["exists" => false]])->count(); // Gives 1
```

### Find objects

//@todo

```php
$q = User::find(); // Find all elements
$q = User::find(["rating" => 10]);
$q = User::find(["name" => ["like" => "David"]]);

// Find by 2 filters
$q = User::findMulti(["rating" => 10, "status" => STATUS_ONLINE], ["rating" => 10, "status" => STATUS_ONLINE]);

// Skip and limit
$q = $q->skip(10)->limit(5);
$q = $q->paginate();

// Order
$q = $q->orderBy("rating");
$q = $q->orderBy("rating", -1);
$q = $q->orderBy("rating", 'asc');

$exists  = $q->exists();

$first   = $q->first();
$firstId = $q->firstId();

$list    = $q->get();
$listIds = $q->getIds();

$hasNext = $q->hasNext();
$total   = $q->total();
```
