# Annotations

- [Installation](#installation)
- [Scanning](#scanning)
- [Event Annotations](#events)
- [Route Annotations](#routes)

<a name="installation"></a>
## Installation

> If you have changed the top-level namespace to something like 'MyCompany', then you would use the new namespace instead of 'App'.

Begin by installing this package through Composer. Edit your project's `composer.json` file to require `laravelcollective/annotations`.

    "require": {
        "laravelcollective/annotations": "~5.0"
    }

Next, update Composer from the Terminal:

    composer update

Once composer is done, you'll need to create a Service Provider in `app/Providers/AnnotationsServiceProvider.php`.

```php
<?php namespace App\Providers;

use Collective\Annotations\AnnotationsServiceProvider as ServiceProvider;

class AnnotationsServiceProvider extends ServiceProvider {

    /**
     * The classes to scan for event annotations.
     *
     * @var array
     */
    protected $scanEvents = [];

    /**
     * The classes to scan for route annotations.
     *
     * @var array
     */
    protected $scanRoutes = [];

    /**
     * Determines if we will auto-scan in the local environment.
     *
     * @var bool
     */
    protected $scanWhenLocal = false;

}
```

Finally, add your new provider to the `providers` array of `config/app.php`:

```php
  'providers' => [
    // ...
    'App\Providers\AnnotationsServiceProvider',
    // ...
  ];
```

<a name="scanning"></a>
## Setting up Scanning

Scanning your controllers for annotations can be configured by editing the `protected $scanEvents` and `protected $scanRoutes` in your `AnnotationsServiceProvider`. For example, if you wanted to scan `App\Handlers\Events\MailHandler` for event annotations, you would add it to `protected $scanEvents` like so:

```php
    /**
     * The classes to scan for event annotations.
     *
     * @var array
     */
    protected $scanEvents = [
      'App\Handlers\Events\MailHandler',
    ];
```

Likewise, if you wanted to scan `App\Http\Controllers\HomeController` for route annotations, you would add it to `protected $scanRoutes` like so:

```php
    /**
     * The classes to scan for route annotations.
     *
     * @var array
     */
    protected $scanRoutes = [
      'App\Http\Controllers\HomeController',
    ];
```

Scanning your event handlers and controllers can be done manually by using `php artisan event:scan` and `php artisan route:scan` respectively, or automatically by setting `protected $scanWhenLocal = true`.

<a name="events"></a>
## Event Annotations

### @Hears

The `@Hears` annotation registers an event listener for a particular event. Annotating any method with `@Hears("SomeEventName")` will register an event listener that will call that method when the `SomeEventName` event is fired.

```php
<?php namespace App\Handlers\Events;

use App\User;

class MailHandler {

  /**
   * Send welcome email to User
   * @Hears("UserWasRegistered")
   */
  public function sendWelcomeEmail(User $user)
  {
    // send welcome email to $user
  }

}
```

<a name="routes"></a>
## Route Annotations

### @Get

The `@Get` annotation registeres a route for an HTTP GET request.

```php
<?php namespace App\Http\Controllers;

class HomeController {

  /**
   * Show the Index Page
   * @Get("/")
   */
  public function getIndex()
  {
    return view('index');
  }

}
```

You can also set up route names.

```php
  /**
   * @Get("/", as="index")
   */
```

... or middlewares.

```php
  /**
   * @Get("/", middleware="guest")
   */
```

... or both.

```php
  /**
   * @Get("/", as="index", middleware="guest")
   */
```

Here's an example that uses all of the available parameters for a `@Get` annotation:

```php
  /**
   * @Get("/profiles/{id}", as="profiles.show", middleware="guest", domain="foo.com", where={"id": "[0-9]+"})
   */
```

### @Post, @Options, @Put, @Patch, @Delete, @any

The `@Post`, `@Options`, `@Put`, `@Patch`, `@Delete`, and `@Any` annotations have the exact same syntax as the `@Get` annotation, except it will register a route for the respective HTTP verb, as opposed to the GET verb.

### @Middleware

As well as defining middleware inline in the route definition tags (`@Get`, `@Post`, etc.), the `@Middleware` tag can be used on its own. It works both individual methods:

```php
  /**
   * Show the Login Page
   *
   * @Get("login")
   * @Middleware("guest")
   */
  public function login()
  {
    return view('index');
  }
```

Or on a whole controller, with the same only/exclude filter syntax that you can use elsewhere in laravel:

```php
/**
 * @Middleware("guest", except={"logout"})
 */
class AuthController extends Controller {

  /**
   * Log the user out.
   *
   * @Get("logout", as="logout")
   * @Middleware("auth")
   *
   * @return Response
   */
  public function logout()
  {
    $this->auth->logout();

    return redirect( route('login') );
  }

}
```

### @Resource

Using the `@Resource` annotation on a controller allows you to easily set up a Resource Controller.

```php
<?php
/**
 * @Resource('users')
 */
class UsersController extends Controller {
  // index(), create(), store(), show($id), edit($id), update($id), destroy($id)
}
```

You can specify the `only` and `except` arguments, just like you can with the regular `Route::resource()` command.

```php
  /**
   * @Resource('users', only={"index", "show"})
   */
```

You can also specify the route names of each resource method.

```php
  /**
   * @Resource('users', names={"index"="user.all", "show"="user.view"})
   */
```