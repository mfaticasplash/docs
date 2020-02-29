---
title: Data Properties
extends: _layouts.documentation
section: content
---

Livewire components store and track data as public properties on the component class.

@code(['lang' => 'php'])
@verbatim
class HelloWorld extends Component
{
    public $message = 'Hello World!';
}
@endverbatim
@endcode

Public properties in Livewire are automatically made available to the view. No need to explicitly pass them into the view (although you can if you want).

@codeComponent([
    'className' => 'HelloWorld.php',
    'viewName' => 'hello-world.blade.php',
])
@slot('class')
use Livewire\Component;

class HelloWorld extends Component
{
    public $message = 'Hello World!';

    public function render()
    {
        return view('livewire.hello-world');
    }
}
@endslot
@slot('view')
@verbatim
<div>
    <h1>{{ $message }}</h1>
    <!-- Will output "Hello World!" -->
</div>
@endverbatim
@endslot
@endcodeComponent

### First, Three Important Notes {#important-notes}

Here are three ESSENTIAL things to note about public properties before embarking on your Livewire journey:

1. Data stored in public properties is made visible to the front-end JavaScript. Therefore, you SHOULD NOT store sensitive data in them.
2. Properties can ONLY be either a JavaScript-friendly data types (`string`, `int`, `array`, `boolean`), OR an eloquent model (or collection of models).

For more information on storing sensitive data in your Livewire components or storing different data types in them, see `this guide` (replace me!!!)

@warning
<code>protected</code> and <code>private</code> properties DO NOT persist between Livewire updates. In general, you should avoid using them for storing state.
@endwarning

## Initializing Properties {#initializing-properties}

You can initialize properties using the `mount` method of your component.

@codeComponent(['className' => 'HelloWorld.php'])
@slot('class')
use Livewire\Component;

class HelloWorld extends Component
{
    public $message;

    public function mount()
    {
        $this->message = 'Hello World!';
    }

    public function render()
    {
        return view('livewire.hello-world');
    }
}
@endslot
@endcodeComponent

Livewire also makes a `$this->fill()` method available to you for cases where you have to set lots of properties and want to remove visual noise.

@codeComponent(['className' => 'HelloWorld.php'])
@slot('class')
public function mount()
{
    $this->fill(['message' => 'Hello World!]);
}
@endslot
@endcodeComponent

Additionally, Livewire offers `$this->reset()` to programatically reset public property values to their initial state. This is useful for cleaning input fields after performing an action.

@codeComponent
@slot('class')
public $name = '';

public function savePost()
{
    $this->post->update([
        'name' => $this->name,
    ]);

    $this->reset();
    // Will reset all public properties.

    $this->reset('name');
    // Will only reset the name property.
}
@endslot
@endcodeComponent

## Data Binding {#data-binding}
If you've used front-end frameworks like Vue, or Angular, you are already familiar with this concept. However, if you are new to this concept, Livewire can "bind" (or "synchronize") the current value of some HTML element with a specific property in your component.

@codeComponent([
    'className' => 'HelloWorld.php',
    'viewName' => 'hello-world.blade.php',
])
@slot('class')
@verbatim
use Livewire\Component;

class HelloWorld extends Component
{
    public $message;

    public function render()
    {
        return view('livewire.hello-world');
    }
}
@endverbatim
@endslot
@slot('view')
@verbatim
<div>
    <input wire:model="message" type="text">

    <h1>{{ $message }}</h1>
</div>
@endverbatim
@endslot
@endcodeComponent

When the user types something into the text field, the value of the `$message` property will automatically update.

Internally, Livewire will listen for an `input` event on the element, and when triggered, it will send an AJAX request to re-render the component with the new data.

@tip
You can add <code>wire:model</code> to any element that dispataches an <code>input</code> event. Even custom elements, or third-party JavaScript libraries.
@endtip

Common elements to use `wire:model` on include:

Element Tag |
--- |
`<input type="text">` |
`<input type="radio">` |
`<input type="checkbox">` |
`<select>` |
`<textarea>` |

### Input Debounce {#input-debounce}

By default, Livewire applies a 150ms debounce to text inputs. This avoids too many network requests being sent as a user types into a text field.

If you wish to override this default (or add it to a non-text input), Livewire offers a "debounce" modifier. If you want to apply a half-second debounce to an input, you would include the modifier like so:

@code
<input type="text" wire:model.debounce.500ms="name">
@endcode

### Nested Data {#nested-binding}

Livewire supports binding to nested data inside arrays using dot notation:

@code
<input type="text" wire:model="parent.message">
@endcode

### Lazily Updating {#lazilly-updating}

By default, Livewire sends a request to server after every `input` event (or `change` in some cases). This is usually fine for things like `<select>` elements that don't typically fire rapid updates, however, this is often unnecessary for text fields that update as the user types.

In those cases, use the `lazy` directive modifier to listen for the native "change" event.

@code
<input type="text" wire:model.lazy="message">
@endcode

Now, the `$message` property will only be updated when the user clicks away from the input field.

## Updating The Query String {#update-query-string}

Sometimes it's useful to update the browser's query string when your component state changes.

For example, if you were building a "search posts" component, and wanted the query string to reflect the current search value like so:

`https://your-app.com/search-posts?search=some-search-string`

This way, when a user hits the back button, or bookmarks the page, you can get the initial state out of the query string, rather than resetting the component every time.

In these cases, you can add a property's name to `protected $updatesQueryString`, and Livewire will update the query string every time the property value changes.

@codeComponent([
    'className' => 'SearchPosts.php',
    'viewName' => 'search-posts.blade.php',
])
@slot('class')
use Livewire\Component;

class SearchPosts extends Component
{
    public $search;

    protected $updatesQueryString = ['search'];

    public function mount()
    {
        $this->search = request()->query('search', $this->search);
    }

    public function render()
    {
        return view('livewire.search-posts', [
            'posts' => Post::where('title', 'like', '%'.$this->search.'%')->get(),
        ]);
    }
}
@endslot
@slot('view')
@verbatim
<div>
    <input wire:model="search" type="text" placeholder="Search posts by title...">

    <h1>Search Results:</h1>

    <ul>
        @foreach($posts as $post)
            <li>{{ $post->title }}</li>
        @endforeach
    </ul>
</div>
@endverbatim
@endslot
@endcodeComponent

### Keeping A Clean Query String {#clean-query-string}

In the case above, when the search property is empty, the query string will look like this:

`?search=`

There are other cases where you might want to only represent a value in the query string if it is NOT the default setting.

For example, if you have a `$page` property to track pagination in a component, you may want to remove the `page` property from the query string when the user is on the first page.

In cases like these, you can use the following syntax:

@codeComponent(['className' => 'SearchPosts.php'])
@slot('class')
use Livewire\Component;

class SearchPosts extends Component
{
    public $foo;
    public $search = '';
    public $page = 1;

    protected $updatesQueryString = [
        'foo'
        ['search' => ['except' => '']],
        ['page' => ['except' => 1]],
    ];

    public function mount()
    {
        $this->fill(request()->only('search', 'page'));
    }

    ...
}
@endslot
@endcodeComponent

## Casting Properties {#casting-properties}

Livewire offers an api to "cast" public properties to a more usable data type. Two common use-cases for this are working with date objects like `Carbon` instances, and dealing with Laravel collections:

@code(['lang' => 'php'])
@verbatim
class CastedComponent extends Component
{
    public $options = ['foo', 'bar', 'bar'];
    public $expiresAt = 'tomorrow';
    public $formattedDate = 'today';

    protected $casts = [
        'options' => 'collection',
        'expiresAt' => 'date',
        'formattedDate' => 'date:m-d-y'
    ];

    public function getUniqueOptions()
    {
        return $this->options->unique();
    }

    public function getExpirationDateForHumans()
    {
        return $this->expiresAt->format('m/d/Y');
    }

    ...
@endverbatim
@endcode

### Custom Casters {#custom-casters}

Livewire allows you to build your own custom casters for custom use-cases. Implement the `Livewire\Castable` interface in a class and reference it in the `$casts` property:

@code(['lang' => 'php'])
@verbatim
class FooComponent extends Component
{
    public $foo = ['bar', 'baz'];

    protected $casts = ['foo' => CollectionCaster::class];

    ...
@endverbatim
@endcode

@code(['lang' => 'php'])
@verbatim
use Livewire\Castable;

class CollectionCaster implements Castable
{
    public function cast($value)
    {
        return collect($value);
    }

    public function uncast($value)
    {
        return $value->all();
    }
}
@endverbatim
@endcode

## Computed Properties {#computed-properties}

Livewire offers an api for accessing dynamic properties. This is especially helpful for deriving properties from the database or another persistent store like a cache.

@code(['lang' => 'php'])
@verbatim
class ShowPost extends Component
{
    // Computed Property
    public function getPostProperty()
    {
        return Post::find($this->postId);
    }
@endverbatim
@endcode

Now, you can access `$this->post` from either the component's class or Blade view:

@codeComponent([
    'className' => 'ShowPost.php',
    'viewName' => 'show-post.blade.php',
])
@slot('class')
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function getPostProperty()
    {
        return Post::find($this->postId);
    }

    public function deletePost()
    {
        $this->post->delete();
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
@endslot
@slot('view')
@verbatim
<div>
    <h1>{{ $this->post->title }}</h1>
    ...
    <button wire:click="deletePost">Delete Post</button>
</div>
@endverbatim
@endslot
@endcodeComponent

@tip
Computed properties are cached for an individual Livewire request lifecycle. Meaning, if you call `$this->post` 5 times in a component's blade view, it won't make a seperate database query every time.
@endtip
