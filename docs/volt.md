> [!warning] Get comfortable with Livewire first
> Before using Volt, we recommend getting familiar with standard, class-based Livewire usage. This will allow you to quickly transfer your knowledge of Livewire into writing components using Volt's functional API.

Volt is an elegantly crafted functional API for Livewire that supports single-file components, allowing a component's PHP logic and Blade templates to coexist in the same file. Behind the scenes, the functional API is compiled to Livewire class components and linked with the template present in the same file.

A simple Volt component looks like the following:

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

## Installation

To get started, install Volt into your project using the Composer package manager:

```bash
composer require livewire/livewire "^3.0@beta" # Or ensure Livewire v3.x is installed...
composer require livewire/volt "^1.0@beta"
```

After installing Volt, you may execute the `volt:install` Artisan command, which will install Volt's service provider file into your application. This service provider specifies the mounted directories in which Volt will search for single file components:

```bash
php artisan volt:install
```

## Creating components

You may create a Volt component by placing a file with the `.blade.php` extension in any of your Volt mounted directories. By default, the `VoltServiceProvider` mounts the `resources/views/livewire` and `resources/views/pages` directories, but you may customize these directories in your Volt service provider's `boot` method.

For convenience, you may use the `make:volt` Artisan command to create a new Volt component:

```bash
php artisan make:volt counter
```

By adding the `--test` directive when generating a component, a corresponding test file will also be generated. If you want the associated test to use [Pest](https://pestphp.com/), you should use the `--pest` flag:

```bash
php artisan make:volt counter --test --pest
```

## API style

By utilizing Volt's functional API, we can define a Livewire component's logic through imported `Livewire\Volt` functions. Volt then transforms and compiles the functional code into a conventional Livewire class, enabling us to leverage the extensive capabilities of Livewire with reduced boilerplate.

Volt's API automatically binds any closure it uses to the underlying component. So, at any time, actions, computed properties, or listeners can refer to the component using the `$this` variable:

```php
use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

// ...
```

## Rendering and mounting components

Just like a typical Livewire component, Volt components may be rendered using Livewire's tag syntax or the `@livewire` Blade directive:

```blade
<livewire:user-index :users="$users" />
```

To declare the component's accepted properties, you may use the `state` function:

```php
use function Livewire\Volt\{state};

state('users');

// ...
```

If necessary, you can intercept the properties passed to the component by providing a closure to the `state` function, allowing you to interact with and modify the given value:

```php
use function Livewire\Volt\{state};

state(['count' => fn ($users) => count($users)]);
```

The `mount` function may be used to define the "mount" [lifecycle hook](/docs/lifecycle-hooks) of the Livewire component. The parameters provided to the component will be injected into this method. Any other parameters required by the mount hook will be resolved by Laravel's service container:

```php
use App\Services\UserCounter;
use function Livewire\Volt\{mount};

mount(function (UserCounter $counter, $users) {
    $counter->store('userCount', count($users));

    // ...
});
```

## Properties

Volt properties, like Livewire properties, are conveniently accessible in the view and persist between Livewire updates. You can define a property using the `state` function:

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

?>

<div>
    {{ $count }}
</div>
```

If the initial value of a state property relies on outside dependencies, such as database queries, models, or container services, its resolution should be encapsulated within a closure. This prevents the value from being resolved until it is absolutely necessary:

```php
use App\Models\User;
use function Livewire\Volt\{state};

state(['count' => fn () => User::count()]);
```

If the initial value of a state property is being injected via [Laravel Folio's](https://github.com/laravel/folio) route model binding, it should also be encapsulated within a closure:

```php
use App\Models\User;
use function Livewire\Volt\{state};

state(['user' => fn () => $user]);
```

Of course, properties may also be declared without explicitly specifying their initial value. In such cases, their initial value will be `null` or will be set based on the properties passed into the component when it is rendered:

```php
use function Livewire\Volt\{mount, state};

state(['count']);

mount(function ($users) {
    $this->count = count($users);

    //
});
```

### Locked properties

Livewire offers the ability to safeguard properties by enabling you to "lock" them, thereby preventing any modifications from occurring on the client-side. To achieve this using Volt, simply chain the `locked` method on the state you wish to protect:

```php
state(['id'])->locked();
```

### Reactive properties

When working with nested components, you may find yourself in a situation where you need to pass a property from a parent component to a child component, and have the child component [automatically update](/docs/nesting#reactive-props) when the parent component updates the property.

To achieve this using Volt, you may chain the `reactive` method on the state you wish to be reactive:

```php
state(['todos'])->reactive();
```

### Computed properties

Livewire also allows you to define [computed properties](/docs/computed-properties), which can be useful for lazily fetching information needed by your component. Computed property results are "memoized", or cached in memory, for an individual Livewire request lifecycle.

To define a computed property, you may use the `computed` function. The name of the variable will determine the name of the computed property:

```php
<?php

use App\Models\User;
use function Livewire\Volt\{computed};

$count = computed(function () {
    return User::count();
});

?>

<div>
    {{ $this->count }}
</div>
```

You may persist the computed property's value in your application's cache by chaining the `persist` method onto the computed property definition:

```php
$count = computed(function () {
    return User::count();
})->persist();
```

By default, Livewire caches the computed computed property's value for 3600 seconds. You may customize this value by providing the desired number of seconds to the `persist` method:

```php
$count = computed(function () {
    return User::count();
})->persist(seconds: 10);
```

## Actions

Livewire [actions](/docs/actions) provide a convenient way to listen to page interactions and invoke a corresponding method on your component, resulting in the re-rendering of the component. Often, actions are invoked in response to the user clicking a button.

To define a Livewire action using Volt, you simply need to define a closure. The name of the variable containing the closure will determine the name of the action:

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

Within the closure, the `$this` variable is bound to the underlying Livewire component, giving you the ability to access other methods on the component just as you would in a typical Livewire component:

```php
use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = function () {
    $this->dispatch('count-updated');

    //
};
```

Your action may also receive arguments or dependencies from Laravel's service container:

```php
use App\Repositories\PostRepository;
use function Livewire\Volt\{state};

state(['postId']);

$delete = function (PostRepository $posts) {
    $posts->delete($this->postId);

    // ...
};
```

### Renderless actions

In some cases, your component might declare an action that does not perform any operations that would cause the component's rendered Blade template to change. If that's the case, you can [skip the rendering phase](/docs/actions#skipping-re-renders) of Livewire's lifecycle by encapsulating the action within the `action` function and chaining the `renderless` method onto its definition:

```php
use function Livewire\Volt\{action};

$incrementViewCount = action(fn () => $this->viewCount++)->renderless();
```

### Protected helpers

By default, all Volt actions are "public" and may be invoked by the client. If you wish to create a function that is [only accessible from within your actions](/docs/actions#keep-dangerous-methods-protected-or-private), you may use the `protect` function:

```php
use App\Repositories\PostRepository;
use function Livewire\Volt\{protect, state};

state(['postId']);

$delete = function (PostRepository $posts) {
    $this->ensurePostCanBeDeleted();

    $posts->delete($this->postId);

    // ...
};

$ensurePostCanBeDeleted = protect(function () {
    // ...
});
```

## Forms

Livewire's [forms](/docs/forms) provide a convenient way to deal with form validation and submission within a single class. To use a Livewire form within a Volt component, you may utilize the `form` function:

```php
<?php

use App\Livewire\Forms\PostForm;
use function Livewire\Volt\{form};

form(PostForm::class);

$save = function () {
    $this->form->store();

    // ...
};

?>

<form wire:submit="save">
    <input type="text" wire:model="form.title">
    @error('form.title') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Save</button>
</form>
```

As you can see, the `form` function accepts the name of a Livewire form class. Once defined, the form can be accessed via the `$this->form` property within your component.

If you want to use a different property name for your form, you can pass the name as the second argument to the `form` function:

```php
form(PostForm::class, 'postForm');

$save = function () {
    $this->postForm->store();

    // ...
};
```

## Listeners

Livewire's global [event system](/docs/events) enables communication between components. If two Livewire components exist on a page, they can communicate by utilizing events and listeners. When using Volt, listeners can be defined using the `on` function:

```php
use function Livewire\Volt\{on};

on(['eventName' => function () {
    //
}]);
```

If you need to assign dynamic names to event listeners, such as those based on the authenticated user or data passed to the component, you can pass a closure to the `on` function. This closure can receive any component parameter, as well as additional dependencies which will be resolved via Laravel's service container:

```php
on(fn ($post) => [
    'event-'.$post->id => function () {
        //
    }),
]);
```

For convenience, component data may also be referenced when defining listeners using "dot" notation:

```php
on(['event-{post.id}' => function () {
    //
}]);
```

## Lifecycle hooks

Livewire has a variety of [lifecycle hooks](/docs/lifecycle-hooks) that may be used to execute code at various points in a component's lifecycle. Using Volt's convenient API, you can define these lifecycle hooks using their corresponding functions:

```php
use function Livewire\Volt\{boot, booted, ...};

boot(fn () => /* ... */);
booted(fn () => /* ... */);
mount(fn () => /* ... */);
hydrate(fn () => /* ... */);
hydrate(['count' => fn () => /* ... */]);
dehydrate(fn () => /* ... */);
dehydrate(['count' => fn () => /* ... */]);
updating(['count' => fn () => /* ... */]);
updated(['count' => fn () => /* ... */]);
```

## Lazy loading placeholders

When rendering Livewire components, you may pass the `lazy` parameter to a Livewire component to [defer its loading](/docs/lazy) until the initial page is fully loaded. By default, Livewire inserts `<div></div>` tags into the DOM where the component will be loaded.

If you would like to customize the HTML that is displayed within the component's placeholder while the initial page is loaded, you may use the `placeholder` function:

```php
use function Livewire\Volt\{placeholder};

placeholder('<div>Loading...</div>');
```

## Validation

Livewire offers easy access to Laravel's powerful [validation features](/docs/validation). Using Volt's API, you may define your component's validation rules using the `rules` function. Like traditional Livewire components, these rules will be applied to your component data when you invoke the `validate` method:

```php
<?php

use function Livewire\Volt\{rules};

rules(['name' => 'required|min:6', 'email' => 'required|email']);

$submit = function () {
    $this->validate();

    // ...
};

?>

<form wire:submit.prevent="submit">
    //
</form>
```

If you need to define rules dynamically, such as rules based on the authenticated user or a information from your database, you can provide a closure to the `rules` function:

```php
rules(fn () => [
    'name' => ['required', 'min:6'],
    'email' => ['required', 'email', 'not_in:'.Auth::user()->email]
]);
```

### Error messages and attributes

To modify the validation messages or attributes used during validation, you can chain the `messages` and `attributes` methods onto your `rules` definition:

```php
use function Livewire\Volt\{rules};

rules(['name' => 'required|min:6', 'email' => 'required|email'])
    ->messages([
        'email.required' => 'The :attribute may not be empty.',
        'email.email' => 'The :attribute format is invalid.',
    ])->attributes([
        'email' => 'email address',
    ]);
```

## File uploads

When using Volt, [uploading and storing files](/docs/uploads) is made incredibly thanks to Livewire. To include the `Livewire\WithFileUploads` trait on your functional Volt component, you may use the `usesFileUploads` function:

```php
use function Livewire\Volt\{state, usesFileUploads};

usesFileUploads();

state(['photo']);

$save = function () {
    $this->validate([
        'photo' => 'image|max:1024',
    ]);

    $this->photo->store('photos');
};
```

## URL query parameters

Sometimes it's useful to [update the browser's URL query parameters](/docs/url) when your component state changes. In these cases, you can use the `url` method to instruct Livewire to sync the URL query parameters with a piece of component state:

```php
<?php

use App\Models\Post;
use function Livewire\Volt\{computed, state};

state(['search'])->url();

$posts = computed(function () {
    return Post::where('title', 'like', '%'.$this->search.'%')->get();
});

?>

<div>
    <input wire:model="search" type="search" placeholder="Search posts by title...">

    <h1>Search Results:</h1>

    <ul>
        @foreach($this->posts as $post)
            <li wire:key="{{ $post->id }}">{{ $post->title }}</li>
        @endforeach
    </ul>
</div>
```

Additional URL query parameters options supported by Livewire, such as URL query parameters aliases, may also be provided to the `url` method:

```php
use App\Models\Post;
use function Livewire\Volt\{state};

state(['page' => 1])->url(as: 'p', history: true, keep: true);

// ...
```

## Pagination

Livewire and Volt also have complete support for [pagination](/docs/pagination). To include Livewire's `Livewire\WithPagination` trait on your functional Volt component, you may use the `usesPagination` function:

```php
<?php

use function Livewire\Volt\{computed, usesPagination};

usesPagination();

$posts = computed(function () {
    return Post::paginate(10);
});

?>

<div>
    @foreach ($this->posts as $post)
        //
    @endforeach

    {{ $this->posts->links() }}
</div>
```

Like Laravel, Livewire's default pagination view uses Tailwind classes for styling. If you use Bootstrap in your application, you can enable the Bootstrap pagination theme by specifying your desired theme when invoking the `usesPagination` function:

```php
usesPagination(theme: 'bootstrap');
```

## Custom traits

To include any arbitrary trait on your functional Volt component, you may use the `uses` function:

```php
use function Livewire\Volt\{uses};

use App\Concerns\WithSorting;

uses(WithSorting::class);
```

## Anonymous components

Sometimes, you may want to convert a small portion of a page a Volt component without extracting it into a separate file. For example, imagine a Laravel route that returns the following view:

```php
Route::get('/counter', fn () => view('pages/counter.blade.php'));
```

The view's content is a typical Blade template, including layout definitions and slots. However, by wrapping a portion of the view within the `@volt` Blade directive, we can convert that piece of the view into a fully-functional Volt component:

```php
<?php

use function Livewire\Volt\{state};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<x-app-layout>
    <x-slot name="header">
        Counter
    </x-slot>

    @volt('counter')
        <div>
            <h1>{{ $count }}</h1>
            <button wire:click="increment">+</button>
        </div>
    @endvolt
</x-app-layout>
```

#### Passing data to anonymous components

When rendering a view that contains an anonymous component, all of the data given to the view will also be available to the anonymous Volt component:

```php
use App\Models\User;

Route::get('/counter', fn () => view('users.counter', [
    'count' => User::count(),
]));
```

Of course, you may declare this data as "state" on your Volt component. When initializing state from data proxied to the component by the view, you only need to declare the name of the state variable. Volt will automatically hydrate the state's default value using the proxied view data:

```php
<?php

use function Livewire\Volt\{state};

state('count');

$increment = function () {
    // Store the new count value in the database...

    $this->count++;
};

?>

<x-app-layout>
    <x-slot name="header">
        Initial value: {{ $count }}
    </x-slot>

    @volt('counter')
        <div>
            <h1>{{ $count }}</h1>
            <button wire:click="increment">+</button>
        </div>
    @endvolt
</x-app-layout>
```

## Testing components

To begin testing a Volt component, you may invoke the `Volt::test` method, providing the name of the component:

```php
use Livewire\Volt\Volt;

it('increments the counter', function () {
    Volt::test('counter')
        ->assertSee('0')
        ->call('increment')
        ->assertSee('1');
});
```

When testing a Volt component, you may utilize all of the methods provided by the standard [Livewire testing API](/docs/testing).

If your Volt component is nested, you may use "dot" notation to specify the component that you wish to test:

```php
Volt::test('users.stats')
```
