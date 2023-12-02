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

# Model Creation
## Basic (Foundational)
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

## Eloquent ORM
```terminal
php artisan make:model Listing
```

Laravel automatically generates a listing model for you with ```all()``` and ```find($id)``` inherited from ```Model```.

## Using the Listing Model
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

# Database
1. Update ```config/database.php``` based on the databse you will use.
2. Update ```.env``` based on the settings of your database.

## Updating Data in the Database
This command will update the database with the changes made in ```database/seeders/DatabaseSeeder.php```.

**Add**

```php
Listing::create([<INSERT_DATA_HERE>])
```

**Command**

```terminal
php artisan migrate:refresh --seed
```

## Factory Creation
```terminal
php artisan make:factory ListingFactory
```

This creates ```database/factories/ListingFactory.php```. It allows you to define default factory values.

**Using Factory in ```DatabaseSeeder.php```**

```php
Listing::factory(6)->create();
```

Run the migrate seed command after.

> [!NOTE]
> Read the documentation to view the different functions for creating dummy data.

# Creating Layouts
Inside ```resources/views/```, create ```layout.blade.php``` file.

```html
<!-- remember that this is layout.blade.php -->
...
<body>
    <!-- Blade files that extends this layout will appear here. -->
    @yield('content')
</body
...
```

**Child Blade File**

```php
@extends('layout');

@section('content')
...
@endsection
```

# Adding Themes

In the ```public``` folder, you can add an ```image``` folder.

**Using Images**
```php
<img src="{{asset('images/no-image.png')}}" />
```

> [!NOTE]
> For collections, you can use ```$listing->title``` instead of ```$listing['title']```.

# Template Partials
Create a ```resources/views/partials``` folder to trim down your code.

Inside the ```paritals``` folder, create a ```_*.blade.php``` file. Paste the partial code.

**Using Template Partials**
```php
@include('partials._hero')
```

# Route Model Building
```php
Route::get('/listings/id{id}', function($id){
    $listing = Listing::find($id);

    if($listing) {
        return view('listing', [
            'listing' => $listing
        ]);
    } else {
        abort('404');
    }
})
```

# Blade Components
Create a ```resources/views/components``` folder.

Inside, you can create components with props inside.

```php
// resources/views/components/listing-card.blade.php
@props(['listing'])

<x-card>
    {{ $listing->location }}
</x-card>
```

```php
// resources/views/listings.blade.php
...

@foreach($listings as $listing)
    <x-listing-card :listing="$listing" />
@endforeach
```

```php
// resources/views/components/card.blade.php
<div {{$atttributes->merge(['class' => 'bg-gray-50 border border-gray-200 rounded p-6'])}}>
    {{ $slot }}
</div>
```

# Controllers

You can add controllers in ```app/Http/Controllers```.

**Example**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Listing;
use Illuminate\Http\Request;

class ListingController extends Controller
{
    // Show all listings
    public function index(){
        return view('listings', [
            'listings' => Listing:all()
        ])
    }

    // Show single listing
    public function show(Listing $listing) {
        return view('listings', [
            'listing' => $listing 
        ])
    }
}
```

```php
// routes/web.php

...

// All Listings
Route::get('/', [ListingController::class, 'index']);

// Single Listing
Route::get('/listings/{listing}', [ListingController::class, 'show']);

// Common Resource Routes:
// index  - Show all listings
// show - Show single listing
// create - Show form to create new listing
// store - Store new listing
// edit - Show form to edit listing
// update - Update listing
// destroy - Delete listing
```

# Resource Method Naming
Create a ```resources/views/listings``` folder.

Then, you can implement this:

| Old Name | New Name |
| --- | --- |
| ```listings.blade.php``` | ```index.blade.php``` |
| ```listing.blade.php``` | ```show.blade.php``` |

Then, you can update ```ListingController.php```.

```php
class ListingController extends Controller
{
    // Show all listings
    public function index(){
        return view('listings.index', [
            'listings' => Listing:all()
        ])
    }

    // Show single listing
    public function show(Listing $listing) {
        return view('listings.show', [
            'listing' => $listing 
        ])
    }
}
```

# Layout Component
```php
// resources/views/components/layout.blade.php
...
<main>
    {{$slot}}
</main>
...
```

```php
// resources/show.blade.php
<x-layout>
...
</x-layout>
```

# Filters
```php
// Http/Models/Listings.php

...

class Listing extends Model {
    use HasFactory;

    public function scopeFilter($query, array $filters) {
        if($filters['tag'] ?? false) {
            $query->where('tags', 'like', '%', . request('tag' . '%'))
        }
    }
}
```

```php
// Http/Controllers/ListingController.php

class ListingController extends Controller
{
    public function index() {
        return voew('listings.index', [
            'listings' => Listing::latest()->filter(request(['tag'])))->get()
        ])
    }
    ...
}