# Basics
## Installing Composer
```terminal
sudo php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
sudo php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
sudo php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

## Install Dependencies
```terminal
composer install
```

## Generate Key
```terminal
php artisan key:generate
```

## Installing Laravel
```terminal
composer global require "laravel/installer"
```

## Creating a project in Laravel
```terminal
composer create-project laravel/laravel firstProject
```

## Running a Laravel Project
```terminal
php artisan serve
```

# Routes

```web.php``` for basic routing.

```api.php``` for creating APIs.

# Blade

You can use ```{{}}``` for calling data.

You can also use directives like ```@foreach``` and ```@unless```.

# Basic Model Creation

Inside ```app/Models/```, you can create a model php file.

The syntax looks something like this:

```php
// app/Models/Listing.php
namespace App\Models;

class Listing {
    public static function all() {
        return <INSERT_ARRAY_DATA_HERE>
    }
    public static function find($id) {
        $listings = self::all();

        forreach($listings as $listing) {
            if($listing[$id] == $id) {
                return $listing;
            }
        }
    }
}
```

```php
// routes/web.php

...
use App\Models\Listing;

Route::get('/', function () {
    return view('listings',[
        'listings' => Listing::all()
    ]);
});

Route::get('/listings/id', function($id) {
    return view('listing', [
        'listing' => Listing::find($id);
    ]);
});
```