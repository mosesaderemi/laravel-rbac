# Laravel RBAC

A small role-based access control package for Laravel. Users have roles, roles have abilities, and every ability is registered automatically as a Laravel Gate, so you can use the standard `can` checks, policies and middleware.

## Features

- Roles, abilities and the pivot tables that connect them to users, shipped as migrations
- Eloquent models for `Role` and `Ability`, and a base `User` model with role helpers
- Every stored ability is defined as a Gate at boot time
- Helper methods `hasRole()` and `hasAbility()` on the user
- Can be switched on and off with a single environment variable
- Model factories for roles and abilities to speed up testing

## Database Schema

| Table | Columns |
| --- | --- |
| `roles` | `id`, `name`, `label` (nullable), timestamps |
| `abilities` | `id`, `name`, `label` (nullable), timestamps |
| `ability_role` | `ability_id`, `role_id` (foreign keys) |
| `role_user` | `user_id`, `role_id` (foreign keys) |

## Installation

```bash
composer require aries/laravel-rbac
```

Publish the package files:

```bash
php artisan vendor:publish --provider="Aries\LaravelRbac\LaravelRbacServiceProvider"
```

Enable the package in `.env`:

```bash
RBAC_ENABLE=true
```

Run the migrations:

```bash
php artisan migrate
```

If you are using the package from a local copy, add it as a path repository in your application's `composer.json` first.

## Setup

Make your application's `User` model extend the package's user model:

```php
<?php

namespace App\Models;

use Aries\LaravelRbac\Models\User as RbacUser;

class User extends RbacUser
{
    // ...
}
```

## Usage

### Create a role for a user

```php
$user = App\Models\User::first();

$user->roles()->create([
    'name' => 'editor',
    'label' => 'Content editor',
]);
```

Attach existing roles:

```php
$roles = Aries\LaravelRbac\Models\Role::all()->pluck('id')->toArray();
$user->roles()->attach($roles);
```

### Create an ability and give it to a role

```php
$ability = Aries\LaravelRbac\Models\Ability::create([
    'name' => 'publish-post',
    'label' => 'Publish posts',
]);

$role = Aries\LaravelRbac\Models\Role::first();
$role->abilities()->attach($ability->id);
```

You can also create an ability through the role with `$role->abilities()->create([...])`.

### Check permissions

Each ability is a Gate, so the usual Laravel authorization tools work:

```php
Gate::allows('publish-post');
$user->can('publish-post');
```

```php
Route::post('/posts/{post}/publish', PublishController::class)->middleware('can:publish-post');
```

Or ask the user model directly:

```php
$user->hasRole('editor');           // bool
$user->hasAbility('publish-post');  // bool
```

## Requirements

- PHP 8.0 or later
- Laravel with the default `users` table

## Author

Aderemi Moses Timileyin ([@mosesaderemi](https://github.com/mosesaderemi))

## License

Released under the MIT License. See the `LICENSE` file for details.
