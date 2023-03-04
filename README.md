# symfony-certification

My persoanl notes in order to prepare for symfony certification exam.

# Symfony Architecture 
## Flex:
```bash
your-project/
├── assets/
├── bin/
│   └── console
├── config/
│   ├── bundles.php
│   ├── packages/
│   ├── routes.yaml
│   └── services.yaml
├── public/
│   └── index.php
├── src/
│   ├── ...
│   └── Kernel.php
├── templates/
├── tests/
├── translations/
├── var/
└── vendor/
```
Now add the  `symfony/symfony`  package to the  `conflict`  section of the project's  `composer.json`  file as  [shown in this example of the skeleton-project](https://github.com/symfony/skeleton/blob/a0770a7f26eeda9890a104fa3de8f68c4120fca5/composer.json#L55-L57)  so that it will not be installed again:
```json
{
      "require": {
          "symfony/flex": "^1.0",
+   },
+   "conflict": {
+       "symfony/symfony": "*"
      }
  }
```
### Customizing Flex Paths:

The Flex recipes make a few assumptions about your project's directory structure. Some of these assumptions can be customized by adding a key under the  `extra`  section of your  `composer.json`  file. For example, to tell Flex to copy any PHP classes into  `src/App`  instead of  `src`:
```json
{
    "...": "...",

    "extra": {
        "src-dir": "src/App"
    }
}
```

The configurable paths are:

-   `bin-dir`: defaults to  `bin/`
-   `config-dir`: defaults to  `config/`
-   `src-dir`  defaults to  `src/`
-   `var-dir`  defaults to  `var/`
-   `public-dir`  defaults to  `public/`

If you customize these paths, some files copied from a recipe still may contain references to the original path. In other words: you may need to update some things manually after a recipe is installed.

## Symfony Code License:
Symfony code is released under  [the MIT license](https://en.wikipedia.org/wiki/MIT_License):

## Bundle:
A bundle is similar to a plugin in other software, but even better. Bundles used in your applications must be enabled per [environment](https://symfony.com/doc/current/configuration.html#configuration-environments) in the `config/bundles.php` file:
```php
// config/bundles.php
return [
    // 'all' means that the bundle is enabled for any Symfony environment
    Symfony\Bundle\FrameworkBundle\FrameworkBundle::class => ['all' => true],
    // this bundle is enabled only in 'dev' and 'test', so you can't use it in 'prod'
    Symfony\Bundle\WebProfilerBundle\WebProfilerBundle::class => ['dev' => true, 'test' => true],
];
```
> In a default Symfony application that uses [Symfony Flex](https://symfony.com/doc/current/setup.html#symfony-flex), bundles are enabled/disabled automatically for you when installing/removing them, so you don't need to look at or edit this `bundles.php` file.

### [Creating a Bundle](https://symfony.com/doc/current/bundles.html#creating-a-bundle "Permalink to this headline")

Start by adding creating a new class called  `AcmeTestBundle`:
```php
// src/AcmeTestBundle.php
namespace Acme\TestBundle;

use Symfony\Component\HttpKernel\Bundle\AbstractBundle;

class AcmeTestBundle extends AbstractBundle
{
}
```

### [Bundle Name](https://symfony.com/doc/current/bundles/best_practices.html#bundle-name "Permalink to this headline")

A bundle is also a PHP namespace. The namespace must follow the  [PSR-4](https://www.php-fig.org/psr/psr-4/)  interoperability standard for PHP namespaces and class names: it starts with a vendor segment, followed by zero or more category segments, and it ends with the namespace short name, which must end with  `Bundle`.

A namespace becomes a bundle as soon as you add "a bundle class" to it (which is a class that extends  [Bundle](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Bundle/Bundle.php "Symfony\Component\HttpKernel\Bundle\Bundle")). The bundle class name must follow these rules:

-   Use only alphanumeric characters and underscores;
-   Use a StudlyCaps name (i.e. camelCase with an uppercase first letter);
-   Use a descriptive and short name (no more than two words);
-   Prefix the name with the concatenation of the vendor (and optionally the category namespaces);
-   Suffix the name with  `Bundle`.

Here are some valid bundle namespaces and class names:

|Namespace|Bundle Class Name|`Acme\Bundle\BlogBundle`|
---|---|---|
|AcmeBlogBundle|`Acme\BlogBundle`|AcmeBlogBundle

By convention, the  `getName()`  method of the bundle class should return the class name.

If you share your bundle publicly, you must use the bundle class name as the name of the repository (AcmeBlogBundle and not BlogBundle for instance).

Symfony core Bundles do not prefix the Bundle class with  `Symfony`  and always add a  `Bundle`  sub-namespace; for example:  [FrameworkBundle](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Bundle/FrameworkBundle/FrameworkBundle.php "Symfony\Bundle\FrameworkBundle\FrameworkBundle").

Each bundle has an alias, which is the lower-cased short version of the bundle name using underscores (`acme_blog`  for AcmeBlogBundle). This alias is used to enforce uniqueness within a project and for defining bundle's configuration options (see below for some usage examples).

### [Bundle Directory Structure](https://symfony.com/doc/current/bundles.html#bundle-directory-structure "Permalink to this headline")

|dir| description|
|---|---|
|`src/`| Contains all PHP classes related to the bundle logic (e.g.  `Controller/RandomController.php`).|
|`config/`| Houses configuration, including routing configuration (e.g.  `routing.yaml`).|
|`templates/`| Holds templates organized by controller name (e.g.  `random/index.html.twig`).|
|`translations/`| Holds translations organized by domain and locale (e.g.  `AcmeTestBundle.en.xlf`).|
|`public/`|Contains web assets (images, stylesheets, etc) and is copied or symbolically linked into the project  `public/`  directory via the  `assets:install`  console command.|
|`assets/`| Contains JavaScript, CSS, images and other assets related to the bundle that are not in  `public/`  (e.g. stimulus controllers)
|`tests/`|Holds all tests for the bundle.|


## http fundamentals
### [Symfony Request Object](https://symfony.com/doc/current/introduction/http_fundamentals.html#symfony-request-object "Permalink to this headline")

The  [Request](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php "Symfony\Component\HttpFoundation\Request")  class is an object-oriented representation of the HTTP request message. With it, you have all the request information at your fingertips:
```php
use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();

// the URI being requested (e.g. /about) minus any query parameters
$request->getPathInfo();

// retrieves $_GET and $_POST variables respectively
$request->query->get('id');
$request->request->get('category', 'default category');

// retrieves $_SERVER variables
$request->server->get('HTTP_HOST');

// retrieves an instance of UploadedFile identified by "attachment"
$request->files->get('attachment');

// retrieves a $_COOKIE value
$request->cookies->get('PHPSESSID');

// retrieves an HTTP request header, with normalized, lowercase keys
$request->headers->get('host');
$request->headers->get('content-type');

$request->getMethod();    // e.g. GET, POST, PUT, DELETE or HEAD
$request->getLanguages(); // an array of languages the client accepts
```
As a bonus, the `Request` class does a lot of work in the background about which you will never need to worry. For example, the `isSecure()` method checks the _three_ different values in PHP that can indicate whether or not the user is connecting via a secure connection (i.e. HTTPS).
```php
use Symfony\Component\HttpFoundation\Response;

$response = new Response();

$response->setContent('<html><body><h1>Hello world!</h1></body></html>');
$response->setStatusCode(Response::HTTP_OK);

// sets a HTTP response header
$response->headers->set('Content-Type', 'text/html');

// prints the HTTP headers followed by the content
$response->send();
```
> The `Request` and `Response` classes are part of a standalone component called [symfony/http-foundation](https://symfony.com/doc/current/components/http_foundation.html) that you can use in _any_ PHP project. This also contains classes for handling sessions, file uploads and more.

### [Summary: The Request-Response Flow](https://symfony.com/doc/current/introduction/http_fundamentals.html#summary-the-request-response-flow "Permalink to this headline")

Here's what we've learned so far:

1.  A client (e.g. a browser) sends an HTTP request;
2.  Each request executes the same, single file (called a "front controller");
3.  The front controller boots Symfony and passes the request information;
4.  Internally, Symfony uses  _routes_  and  _controllers_  to create the Response for the page (we'll learn about these soon!);
5.  Symfony turns your  `Response`  object into the text headers and content (i.e. the HTTP response), which are sent back to the client.

This renderer uses the HTTP status code and the following logic to determine the template filename:

1.  Look for a template for the given status code (like  `error500.html.twig`);
2.  If the previous template doesn't exist, discard the status code and look for a generic error template (`error.html.twig`).

To override these templates, rely on the standard Symfony method for [overriding templates that live inside a bundle](https://symfony.com/doc/current/bundles/override.html#override-templates) and put them in the `templates/bundles/TwigBundle/Exception/` directory.

A typical project that returns HTML pages might look like this:
```bash
templates/
└─ bundles/
   └─ TwigBundle/
      └─ Exception/
         ├─ error404.html.twig
         ├─ error403.html.twig
         └─ error.html.twig      # All other HTML errors (including 500
```
### [Overriding Error output for non-HTML formats](https://symfony.com/doc/current/controller/error_pages.html#overriding-error-output-for-non-html-formats "Permalink to this headline")

```bash
composer require symfony/serializer-pack
```
The Serializer component has a built-in `FlattenException` normalizer ([ProblemNormalizer](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Serializer/Normalizer/ProblemNormalizer.php "Symfony\Component\Serializer\Normalizer\ProblemNormalizer")) and JSON/XML/CSV/YAML encoders.

### [Overriding the Default ErrorController](https://symfony.com/doc/current/controller/error_pages.html#overriding-the-default-errorcontroller "Permalink to this headline")

If you need a little more flexibility beyond just overriding the template, then you can change the controller that renders the error page. For example, you might need to pass some additional variables into your template.

To do this, create a new controller anywhere in your application and set the  [framework.error_controller](https://symfony.com/doc/current/reference/configuration/framework.html#config-framework-error_controller)  configuration option to point to it:
```yaml
# config/packages/framework.yaml
framework:
    error_controller: App\Controller\ErrorController::show
```
The  [ErrorListener](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/EventListener/ErrorListener.php "Symfony\Component\HttpKernel\EventListener\ErrorListener")  class used by the FrameworkBundle as a listener of the  `kernel.exception`  event creates the request that will be dispatched to your controller. In addition, your controller will be passed two parameters:

| type | description |
|---|---|
|	`exception`	|	The original  Throwable instance being handled.	|
|	`logger`	|	A  DebugLoggerInterface  instance which may be  `null`  in some circumstances.	|


When an exception is thrown, the  [HttpKernel](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/HttpKernel.php "Symfony\Component\HttpKernel\HttpKernel")  class catches it and dispatches a  `kernel.exception`  event. This gives you the power to convert the exception into a  `Response`  in a few different ways.

[Writing your own event listener](https://symfony.com/doc/current/event_dispatcher.html) for the `kernel.exception` event allows you to have a closer look at the exception and take different actions depending on it. Those actions might include logging the exception, redirecting the user to another page or rendering specialized error pages.

> If your listener calls `setThrowable()` on the [ExceptionEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/ExceptionEvent.php "Symfony\Component\HttpKernel\Event\ExceptionEvent") event, propagation will be stopped and the response will be sent to the client.

## Built-in Symfony Events
## [Kernel Events](https://symfony.com/doc/current/reference/events.html#kernel-events "Permalink to this headline")
Each event dispatched by the **HttpKernel** component is a subclass of  [KernelEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/KernelEvent.php "Symfony\Component\HttpKernel\Event\KernelEvent"), which provides the following information:

|name|description|
|---|---|
|[getRequestType()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/KernelEvent.php#method_getRequestType "Symfony\Component\HttpKernel\Event\KernelEvent::getRequestType()")|Returns the  _type_  of the request (`HttpKernelInterface::MAIN_REQUEST`  or  `HttpKernelInterface::SUB_REQUEST`).|
|[getKernel()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/KernelEvent.php#method_getKernel "Symfony\Component\HttpKernel\Event\KernelEvent::getKernel()")|Returns the Kernel handling the request.|
|[getRequest()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/KernelEvent.php#method_getRequest "Symfony\Component\HttpKernel\Event\KernelEvent::getRequest()")|Returns the current  `Request`  being handled.|
|[isMainRequest()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/KernelEvent.php#method_isMainRequest "Symfony\Component\HttpKernel\Event\KernelEvent::isMainRequest()")|Checks if this is a main request.|

#### [`kernel.request`](https://symfony.com/doc/current/reference/events.html#kernel-request "Permalink to this headline")

**Event Class**:  [RequestEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/RequestEvent.php "Symfony\Component\HttpKernel\Event\RequestEvent")

This event is dispatched very early in Symfony, before the controller is determined. It's useful to add information to the Request or return a Response early to stop the handling of the request.

#### [`kernel.controller`](https://symfony.com/doc/current/reference/events.html#kernel-controller "Permalink to this headline")

**Event Class**:  [ControllerEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/ControllerEvent.php "Symfony\Component\HttpKernel\Event\ControllerEvent")

This event is dispatched after the controller has been resolved but before executing it. It's useful to initialize things later needed by the controller, such as  [param converters](https://symfony.com/doc/master/bundles/SensioFrameworkExtraBundle/annotations/converters.html), and even to change the controller entirely:

#### [`kernel.controller_arguments`](https://symfony.com/doc/current/reference/events.html#kernel-controller-arguments "Permalink to this headline")

**Event Class**:  [ControllerArgumentsEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/ControllerArgumentsEvent.php "Symfony\Component\HttpKernel\Event\ControllerArgumentsEvent")

This event is dispatched just before a controller is called. It's useful to configure the arguments that are going to be passed to the controller. Typically, this is used to map URL routing parameters to their corresponding named arguments; or pass the current request when the  `Request`  type-hint is found

#### [`kernel.view`](https://symfony.com/doc/current/reference/events.html#kernel-view "Permalink to this headline")

**Event Class**:  [ViewEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/ViewEvent.php "Symfony\Component\HttpKernel\Event\ViewEvent")

This event is dispatched after the controller has been executed but  _only_  if the controller does  _not_  return a  [Response](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Response.php "Symfony\Component\HttpFoundation\Response")  object. It's useful to transform the returned value (e.g. a string with some HTML contents) into the  `Response`  object needed by Symfony

#### [`kernel.finish_request`](https://symfony.com/doc/current/reference/events.html#kernel-finish-request "Permalink to this headline")

**Event Class**:  [FinishRequestEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/FinishRequestEvent.php "Symfony\Component\HttpKernel\Event\FinishRequestEvent")

This event is dispatched after the  `kernel.response`  event. It's useful to reset the global state of the application (for example, the translator listener resets the translator's locale to the one of the parent request)

#### [`kernel.terminate`](https://symfony.com/doc/current/reference/events.html#kernel-terminate "Permalink to this headline")

**Event Class**:  [TerminateEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/TerminateEvent.php "Symfony\Component\HttpKernel\Event\TerminateEvent")

This event is dispatched after the response has been sent (after the execution of the  [handle()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/HttpKernel.php#method_handle "Symfony\Component\HttpKernel\HttpKernel::handle()")  method). It's useful to perform slow or complex tasks that don't need to be completed to send the response (e.g. sending emails).

#### [`kernel.exception`](https://symfony.com/doc/current/reference/events.html#kernel-exception "Permalink to this headline")

**Event Class**:  [ExceptionEvent](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Event/ExceptionEvent.php "Symfony\Component\HttpKernel\Event\ExceptionEvent")

This event is dispatched as soon as an error occurs during the handling of the ***HTTP*** request. It's useful to recover from errors or modify the exception details sent as response.

The first event that is dispatched inside [HttpKernel::handle](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/HttpKernel.php#method_handle "Symfony\Component\HttpKernel\HttpKernel::handle()") is `kernel.request`, which may have a variety of different listeners.

Listeners of this event can be quite varied. Some listeners - such as a ***security listener*** - might have enough information to create a `Response` object immediately. For example, if a security listener determined that a user doesn't have access, that listener may return a [RedirectResponse](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/RedirectResponse.php "Symfony\Component\HttpFoundation\RedirectResponse") to the login page or a 403 Access Denied response.

Overall, the purpose of the `kernel.request` event is either to create and return a `Response` **directly**, or to **add information** to the `Request` (e.g. setting the locale or setting some other information on the `Request` attributes).

> When setting a response for the  `kernel.request`  event, the propagation is stopped. This means listeners with lower priority won't be executed.

The most important listener to `kernel.request` in the Symfony Framework is the [RouterListener](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/EventListener/RouterListener.php "Symfony\Component\HttpKernel\EventListener\RouterListener"). 

**2**  _how_  you determine the exact controller for a request is entirely up to your application. This is the job of the "controller resolver" - a class that implements  [ControllerResolverInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Controller/ControllerResolverInterface.php "Symfony\Component\HttpKernel\Controller\ControllerResolverInterface")  and is one of the constructor arguments to  `HttpKernel`.

#### [The  `kernel.controller`  Event](https://symfony.com/doc/current/components/http_kernel.html#3-the-kernel-controller-event "Permalink to this headline")

`kernel.controller`  in the Symfony Framework

An interesting listener to  `kernel.controller`  in the Symfony Framework is  [CacheAttributeListener](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/EventListener/CacheAttributeListener.php "Symfony\Component\HttpKernel\EventListener\CacheAttributeListener"). This class fetches  `#[Cache]`  attribute configuration from the controller and uses it to configure  [HTTP caching](https://symfony.com/doc/current/http_cache.html)  on the response.

There are a few other minor listeners to the  `kernel.controller`  event in the Symfony Framework that deal with collecting profiler data when the profiler is enabled.

### Getting the Controller Arguments in the Symfony Framework

Now that you know exactly what the controller callable (usually a method inside a controller object) is, the  `ArgumentResolver`  uses  [reflection](https://www.php.net/manual/en/book.reflection.php)  on the callable to return an array of the  _names_  of each of the arguments. It then iterates over each of these arguments and uses the following tricks to determine which value should be passed for each argument:

a) If the  `Request`  attributes bag contains a key that matches the name of the argument, that value is used. For example, if the first argument to a controller is  `$slug`  and there is a  `slug`  **key in the  `Request`  `attributes`  bag**, that value is used (and typically this value came from the  `RouterListener`).

b) If the argument in the controller is type-hinted with Symfony's [Request](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php "Symfony\Component\HttpFoundation\Request")  object, the  `Request`  is passed in as the value.

c) If the function or method argument is  [variadic](https://www.php.net/manual/en/functions.arguments.php#functions.variable-arg-list)  and the  `Request` `attributes`  bag contains an array for that argument, they will all be available through the  [variadic](https://www.php.net/manual/en/functions.arguments.php#functions.variable-arg-list)  argument.

This functionality is provided by resolvers implementing the  [ArgumentValueResolverInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Controller/ArgumentValueResolverInterface.php "Symfony\Component\HttpKernel\Controller\ArgumentValueResolverInterface"). There are four implementations which provide the default behavior of Symfony but customization is the key here. By implementing the  `ArgumentValueResolverInterface`  yourself and passing this to the  `ArgumentResolver`, you can extend this functionality.

> A controller must return _something_. If a controller returns `null`, an exception will be thrown immediately.

#### `kernel.view`  in the Symfony Framework

There is a default listener inside the Symfony Framework for the  `kernel.view`  event. If your controller action returns an array, and you apply the **[#[Template()] attribute](https://symfony.com/doc/current/templates.html#templates-template-attribute)**  to that controller action, then this listener renders a template, passes the array you returned from your controller to that template, and creates a  `Response`  containing the returned content from that template.

#### `kernel.response`  in the Symfony Framework

There are several minor listeners on this event inside the Symfony Framework, and most modify the response in some way. For example, the  [WebDebugToolbarListener](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Bundle/WebProfilerBundle/EventListener/WebDebugToolbarListener.php "Symfony\Bundle\WebProfilerBundle\EventListener\WebDebugToolbarListener")  injects some JavaScript at the bottom of your page in the  `dev`  environment which causes the web debug toolbar to be displayed. Another listener,  [ContextListener](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Security/Http/Firewall/ContextListener.php "Symfony\Component\Security\Http\Firewall\ContextListener")  serializes the current user's information into the session so that it can be reloaded on the next request.

> Using the `kernel.terminate` event is optional, and should only be called if your kernel implements [TerminableInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/TerminableInterface.php "Symfony\Component\HttpKernel\TerminableInterface").

### Event Dispatcher
The Symfony EventDispatcher component implements the  **[Mediator](https://en.wikipedia.org/wiki/Mediator_pattern)** and  **[Observer](https://en.wikipedia.org/wiki/Observer_pattern)**  design patterns to make all these things possible and to make your projects truly extensible.

Take an example from  [the HttpKernel component](https://symfony.com/doc/current/components/http_kernel.html). Once a  `Response`  object has been created, it may be useful to allow other elements in the system to modify it (e.g. add some cache headers) before it's actually used. To make this possible, the Symfony kernel throws an event -  `kernel.response`. Here's how it works:

-   A  _listener_  (PHP object) tells a central  _dispatcher_  object that it wants to listen to the  `kernel.response`  event;
-   At some point, the Symfony kernel tells the  _dispatcher_  object to dispatch the  `kernel.response`  event, passing with it an  `Event`  object that has access to the  `Response`  object;
-   The dispatcher notifies (i.e. calls a method on) all listeners of the  `kernel.response`  event, allowing each of them to make modifications to the  `Response`  object.
>**The unique event name can be any string**

but optionally follows a few naming conventions:

-   Use only lowercase letters, numbers, dots (`.`) and underscores (`_`);
-   Prefix names with a namespace followed by a dot (e.g.  `order.*`,  `user.*`);
-   End names with a verb that indicates what action has been taken (e.g.  `order.placed`).

> An optional priority, defined as a positive or negative integer (defaults to  `0`). **The higher the number, the earlier the listener is called**. If two listeners have the same priority, they are executed in the order that they were added to the dispatcher.

#### [Using Event Subscribers](https://symfony.com/doc/current/components/event_dispatcher.html#using-event-subscribers "Permalink to this headline")
Another way to listen to events is via an _event subscriber_. An event subscriber is a PHP class that's able to tell the dispatcher exactly which events it should subscribe to. It implements the **[EventSubscriberInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/EventDispatcher/EventSubscriberInterface.php "Symfony\Component\EventDispatcher\EventSubscriberInterface")** interface, which requires a single static method called **[getSubscribedEvents()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/EventDispatcher/EventSubscriberInterface.php#method_getSubscribedEvents "Symfony\Component\EventDispatcher\EventSubscriberInterface::getSubscribedEvents()")**.

This is very similar to a listener class, except that the class itself can tell the dispatcher which events it should listen to.
The dispatcher will automatically register the subscriber for each event returned by the `getSubscribedEvents()` method. This method returns an array indexed by event names and **whose values are either the method name to call or an array composed of the method name to call and a priority** (a positive or negative integer that defaults to `0`).

In other words, the listener needs to be able to tell the dispatcher to stop all propagation of the event to future listeners (i.e. to not notify any more listeners). This can be accomplished from inside a listener via the [stopPropagation()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Contracts/EventDispatcher/Event.php#method_stopPropagation "Symfony\Contracts\EventDispatcher\Event::stopPropagation()") method. It is possible to detect if an event was stopped by using the [isPropagationStopped()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Contracts/EventDispatcher/Event.php#method_isPropagationStopped "Symfony\Contracts\EventDispatcher\Event::isPropagationStopped()") method which returns bool.

The [ImmutableEventDispatcher](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/EventDispatcher/ImmutableEventDispatcher.php "Symfony\Component\EventDispatcher\ImmutableEventDispatcher") is a locked or frozen event dispatcher. The dispatcher cannot register new listeners or subscribers.

## Best Practices:
 
 1-  [Use the Symfony Binary to Create Symfony Applications](https://symfony.com/doc/current/best_practices.html#use-the-symfony-binary-to-create-symfony-applications "Permalink to this headline") 
 ```bash
symfony new my_project_directory
```
2 - Keep Sensitive Information Secret
> The Secrets system requires the Sodium PHP extension.
```bash
php bin/console secrets:set DATABASE_PASSWORD
APP_RUNTIME_ENV=prod php bin/console secrets:set DATABASE_PASSWORD
```
# Controller

The `redirect()` method does not check its destination in any way.

If you throw an exception that extends or is an instance of  [HttpException](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Exception/HttpException.php "Symfony\Component\HttpKernel\Exception\HttpException"), Symfony will use the appropriate HTTP status code. Otherwise, the response will have a 500 HTTP status code:
```php
// this exception ultimately generates a 500 status error
throw new \Exception('Something went wrong!');
```
## Request
 The most common way to create a request is to base it on the current PHP global variables with  [createFromGlobals()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_createFromGlobals "Symfony\Component\HttpFoundation\Request::createFromGlobals()"):
```
use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();
```

which is almost equivalent to the more verbose, but also more flexible,  [__construct()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method___construct "Symfony\Component\HttpFoundation\Request::__construct()")  call:
```
$request = new Request(
    $_GET,
    $_POST,
    [],
    $_COOKIE,
    $_FILES,
    $_SERVER
);
```
Finally, the raw data sent with the request body can be accessed using  [getContent()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_getContent "Symfony\Component\HttpFoundation\Request::getContent()"):
```php
$content = $request->getContent();
```
For instance, this may be useful to process an XML string sent to the application by a remote service using the HTTP POST method.

If the request body is a **JSON** string, it can be accessed using  [toArray()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_toArray "Symfony\Component\HttpFoundation\Request::toArray()"):
```php
$data = $request->toArray();
```
The  [create()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_create "Symfony\Component\HttpFoundation\Request::create()")  method creates a request based on a URI, a method and some parameters (the query parameters or the request ones depending on the HTTP method); and of course, you can also override all other variables as well (by default, Symfony creates sensible defaults for all the PHP global variables).

Based on such a request, you can override the PHP global variables via  [overrideGlobals()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_overrideGlobals "Symfony\Component\HttpFoundation\Request::overrideGlobals()"):
```php
$request->overrideGlobals();
```
 ### [Serving Files](https://symfony.com/doc/current/components/http_foundation.html#serving-files "Permalink to this headline")

When sending a file, you must add a  **`Content-Disposition`**  header to your response.

### [Streaming a Response](https://symfony.com/doc/current/components/http_foundation.html#streaming-a-response "Permalink to this headline")

The  [StreamedResponse](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/StreamedResponse.php "Symfony\Component\HttpFoundation\StreamedResponse")  class allows you to stream the Response back to the client. The response content is represented by a PHP callable instead of a string:
```php
use Symfony\Component\HttpFoundation\StreamedResponse;

$response = new StreamedResponse();
$response->setCallback(function () {
    var_dump('Hello World');
    flush();
    sleep(2);
    var_dump('Hello World');
    flush();
});
$response->send();
```

The  `flush()`  function does not flush buffering. If  `ob_start()`  has been called before or the  `output_buffering`  `php.ini`  option is enabled, you must call  `ob_flush()`  before  `flush()`.

Additionally, PHP isn't the only layer that can buffer output. Your web server might also buffer based on its configuration. Some servers, such as nginx, let you disable buffering at the config level or by adding a special HTTP header in the response:
```php
// disables FastCGI buffering in nginx only for this response
$response->headers->set('X-Accel-Buffering', 'no');
```

### [Serving Files](https://symfony.com/doc/current/components/http_foundation.html#serving-files "Permalink to this headline")

When sending a file, you must add a  `Content-Disposition`  header to your response. While creating this header for basic file downloads is straightforward, using non-ASCII filenames is more involved. The  [makeDisposition()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/HeaderUtils.php#method_makeDisposition "Symfony\Component\HttpFoundation\HeaderUtils::makeDisposition()")  abstracts the hard work behind a simple API:

```php
use Symfony\Component\HttpFoundation\HeaderUtils;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\ResponseHeaderBag;

$fileContent = ...; // the generated file content
$response = new Response($fileContent);

$disposition = HeaderUtils::makeDisposition(
    HeaderUtils::DISPOSITION_ATTACHMENT,
    'foo.pdf'
);

$response->headers->set('Content-Disposition', $disposition);
```

Alternatively, if you are serving a static file, you can use a  [BinaryFileResponse](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/BinaryFileResponse.php "Symfony\Component\HttpFoundation\BinaryFileResponse"):

```php
use Symfony\Component\HttpFoundation\BinaryFileResponse;

$file = 'path/to/file.txt';
$response = new BinaryFileResponse($file);
```

The  `BinaryFileResponse`  will automatically handle  `Range`  and  `If-Range`  headers from the request. It also supports  `X-Sendfile`  (see for  [nginx](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/)  and  [Apache](https://tn123.org/mod_xsendfile/)). To make use of it, you need to determine whether or not the  `X-Sendfile-Type`  header should be trusted and call  [trustXSendfileTypeHeader()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/BinaryFileResponse.php#method_trustXSendfileTypeHeader "Symfony\Component\HttpFoundation\BinaryFileResponse::trustXSendfileTypeHeader()")  if it should:
```php
BinaryFileResponse::trustXSendfileTypeHeader();
```

The  `BinaryFileResponse`  will only handle  `X-Sendfile`  if the particular header is present. For Apache, this is not the default case.

To add the header use the  `mod_headers`  Apache module and add the following to the Apache configuration:
```xml
<IfModule mod_xsendfile.c>
  # This is already present somewhere...
  XSendFile on
  XSendFilePath ...some path...

  # This needs to be added:
  <IfModule mod_headers.c>
    RequestHeader set X-Sendfile-Type X-Sendfile
  </IfModule>
</IfModule>
```

With the  `BinaryFileResponse`, you can still set the  `Content-Type`  of the sent file, or change its  `Content-Disposition`:
```php
// ...
$response->headers->set('Content-Type', 'text/plain');
$response->setContentDisposition(
    ResponseHeaderBag::DISPOSITION_ATTACHMENT,
    'filename.txt'
);
```

It is possible to delete the file after the response is sent with the  **[deleteFileAfterSend()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/BinaryFileResponse.php#method_deleteFileAfterSend "Symfony\Component\HttpFoundation\BinaryFileResponse::deleteFileAfterSend()")**  method. Please note that this will not work when the  `X-Sendfile`  header is set.

**If the size of the served file is unknown (e.g. because it's being generated on the fly, or because a PHP stream filter is registered on it, etc.), you can pass a  `Stream`  instance to  `BinaryFileResponse`. This will disable  `Range`  and  `Content-Length`  handling, switching to chunked encoding instead**:
```php
use Symfony\Component\HttpFoundation\BinaryFileResponse;
use Symfony\Component\HttpFoundation\File\Stream;

$stream = new Stream('path/to/stream');
$response = new BinaryFileResponse($stream);
```
> If you _just_ created the file during this same request, the file _may_ be sent without any content. This may be due to cached file stats that return zero for the size of the file. To fix this issue, call `clearstatcache(true, $file)` with the path to the binary file.

Symfony offers two methods to interact with this preference:

-   [preferSafeContent()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php#method_preferSafeContent "Symfony\Component\HttpFoundation\Request::preferSafeContent()");
-   [setContentSafe()](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Response.php#method_setContentSafe "Symfony\Component\HttpFoundation\Response::setContentSafe()");

The following example shows how to detect if the user agent prefers "safe" content:
```php
if ($request->preferSafeContent()) {
    $response = new Response($alternativeContent);
    // this informs the user we respected their preferences
    $response->setContentSafe();

    return $response;
```
# Routing:
When your application receives a request, it calls a  [controller action](https://symfony.com/doc/current/controller.html)  to generate the response. **The routing configuration defines** which action to run for each incoming URL. It also provides other useful features, like generating SEO-friendly URLs (e.g.  `/read/intro-to-symfony`  instead of  `index.php?article_id=57`).

**[Symfony recommends attributes](https://symfony.com/doc/current/best_practices.html#best-practice-controller-annotations) because it's convenient to put the route and controller in the same place.**
The kernel can act as a controller too, which is especially useful for small applications that use Symfony as a microframework.

> If you define multiple PHP classes in the same file, Symfony only loads the routes of the first class, ignoring all the other routes.

The route name (`blog_list`) is not important for now, but it will be essential later when **[generating URLs](https://symfony.com/doc/current/routing.html#routing-generating-urls)**. **You only have to keep in mind that each route name must be unique in the application.**

HTML forms only support `GET` and `POST` methods. If you're calling a route with a different method from an HTML form, add a hidden field called `_method` with the method to use (e.g. `<input type="hidden" name="_method" value="PUT">`). If you create your forms with [Symfony Forms](https://symfony.com/doc/current/forms.html) this is done automatically for you when the [framework.http_method_override](https://symfony.com/doc/current/reference/configuration/framework.html#configuration-framework-http_method_override) option is `true`.

### [Matching Expressions](https://symfony.com/doc/current/routing.html#matching-expressions "Permalink to this headline")
```php
    #[Route(
        '/contact',
        name: 'contact',
        condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'",
        // expressions can also include config parameters:
        // condition: "request.headers.get('User-Agent') matches '%app.allowed_browsers%'"
    )]

    #[Route(
        '/posts/{id}',
        name: 'post_show',
        // expressions can retrieve route parameter values using the "params" variable
        condition: "params['id'] < 1000"
    )]
```
The value of the  `condition`  option is an expression using any valid  [expression language syntax](https://symfony.com/doc/current/reference/formats/expression_language.html)  and can use any of these variables created by Symfony:

| Title| Description |
|---|----|
|`context`| An instance of  [RequestContext](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Routing/RequestContext.php "Symfony\Component\Routing\RequestContext"), which holds the most fundamental information about the route being matched.|
|`request`|The  [Symfony Request](https://symfony.com/doc/current/components/http_foundation.html#component-http-foundation-request)  object that represents the current request.|
|`params`| An array of matched  [route parameters](https://symfony.com/doc/current/routing.html#routing-route-parameters)  for the current route.|
|env(string $name)|Returns the value of a variable using  [Environment Variable Processors](https://symfony.com/doc/current/configuration/env_var_processors.html)|
|service(string $alias)|Returns a routing condition service.|

First, add the `#[AsRoutingConditionService]` attribute or `routing.condition_service` tag to the services that you want to use in route conditions:
```php
use Symfony\Bundle\FrameworkBundle\Routing\Attribute\AsRoutingConditionService;
use Symfony\Component\HttpFoundation\Request;

#[AsRoutingConditionService(alias: 'route_checker')]
class RouteChecker
{
    public function check(Request $request): bool
    {
        // ...
    }
}
```
Then, use the  `service()`  function to refer to that service inside conditions:
```bash
// Controller (using an alias):
#[Route(condition: "service('route_checker').check(request)")]
// Or without alias:
#[Route(condition: "service('Ap\\\Service\\\RouteChecker').check(request)")]
```
Behind the scenes, expressions are compiled down to raw PHP. Because of this, using the `condition` key causes no extra overhead beyond the time it takes for the underlying PHP to execute.
```bash
php bin/console debug:router
```
and 
```bash
php bin/console router:match /lucky/number/8
```
To fix this, add some validation to the  `{page}`  parameter using the  `requirements`  option:
```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    #[Route('/blog/{page}', name: 'blog_list', requirements: ['page' => '\d+'])]
    public function list(int $page): Response
    {
        // ...
    }

    #[Route('/blog/{slug}', name: 'blog_show')]
    public function show($slug): Response
    {
        // ...
    }
}
```
The `requirements` option defines the [PHP regular expressions](https://www.php.net/manual/en/book.pcre.php) that route parameters must match for the entire route to match. In this example, `\d+` is a regular expression that matches a _digit_ of any length.
> The [Requirement](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Routing/Requirement/Requirement.php "Symfony\Component\Routing\Requirement\Requirement") enum contains a collection of commonly used regular-expression constants such as digits, dates and UUIDs which can be used as route parameter requirements.

Parameters also support [PCRE Unicode properties](https://www.php.net/manual/en/regexp.reference.unicode.php), which are escape sequences that match generic character types. For example, `\p{Lu}` matches any uppercase character in any language, `\p{Greek}` matches any Greek characters, etc.

If you prefer, requirements can be inlined in each parameter using the syntax `{parameter_name<requirements>}`. This feature makes configuration more concise, but it can decrease route readability when requirements are complex:
```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    #[Route('/blog/{page<\d+>}', name: 'blog_list')]
    public function list(int $page): Response
    {
        // ...
    }
}
```

> You can have more than one optional parameter (e.g. `/blog/{slug}/{page}`), but everything after an optional parameter must be optional. For example, `/{page}/blog` is a valid path, but `page` will always be required (i.e. `/blog` will not match this route).

If you want to always include some default value in the generated URL (for example to force the generation of  `/blog/1`  instead of  `/blog`  in the previous example) add the  `!`  character before the parameter name:  `/blog/{!page}`

As it happens with requirements, default values can also be inlined in each parameter using the syntax  `{parameter_name?default_value}`. This feature is compatible with inlined requirements, so you can inline both in a single parameter:
```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    #[Route('/blog/{page<\d+>?1}', name: 'blog_list')]
    public function list(int $page): Response
    {
        // ...
    }
}
```
> To give a `null` default value to any parameter, add nothing after the `?` character (e.g. `/blog/{page?}`). If you do this, don't forget to update the types of the related controller arguments to allow passing `null` values (e.g. replace `int $page` by `?int $page`).

### Priority:
```php
    #[Route('/blog/list', name: 'blog_list', priority: 2)]
```
### Param converters:
```bash
	composer require sensio/framework-extra-bundle
```
code:
```php
// src/Controller/BlogController.php
namespace App\Controller;

use App\Entity\BlogPost;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    // ...

    #[Route('/blog/{slug}', name: 'blog_show')]
    public function show(BlogPost $post): Response
    {
        // $post is the object whose slug matches the routing parameter

        // ...
    }
}
```
### [Special Parameters](https://symfony.com/doc/current/routing.html#special-parameters "Permalink to this headline")

In addition to your own parameters, routes can include any of the following special parameters created by Symfony:

|title| description|
|---|---|
|`_controller`|This parameter is used to determine which controller and action is executed when the route is matched.|
|`_format`|The matched value is used to set the "request format" of the  `Request`  object. This is used for such things as setting the  `Content-Type`  of the response (e.g. a  `json`  format translates into a  `Content-Type`  of  `application/json`).|
|`_fragment`|Used to set the fragment identifier, which is the optional last part of a URL that starts with a  `#`  character and is used to identify a portion of a document.|
|`_locale`|Used to set the  [locale](https://symfony.com/doc/current/translation.html#translation-locale-url)  on the request.|

code:
```php
// src/Controller/ArticleController.php
namespace App\Controller;

// ...
class ArticleController extends AbstractController
{
    #[Route(
        path: '/articles/{_locale}/search.{_format}',
        locale: 'en',
        format: 'html',
        requirements: [
            '_locale' => 'en|fr',
            '_format' => 'html|xml',
        ],
    )]
    public function search(): Response
    {
    }
}
```
### [Extra Parameters](https://symfony.com/doc/current/routing.html#extra-parameters "Permalink to this headline")

In the  **`defaults`**  option of a route you can optionally define parameters not included in the route configuration. This is useful to pass extra arguments to the controllers of the routes

### [Slash Characters in Route Parameters](https://symfony.com/doc/current/routing.html#slash-characters-in-route-parameters "Permalink to this headline")

Route parameters can contain any values except the  `/`  slash character, because that's the character used to separate the different parts of the URLs. For example, if the  `token`  value in the  `/share/{token}`  route contains a  `/`  character, this route won't match.

A possible solution is to change the parameter requirements to be more permissive:
```php
    #[Route('/share/{token}', name: 'share', requirements: ['token' => '.+'])]

```
> If the route defines several parameters and you apply this permissive regular expression to all of them, you might get unexpected results. For example, if the route definition is  `/share/{path}/{token}`  and both  `path`  and  `token`  accept  `/`, then  `token`  will only get the last part and the rest is matched by  `path`.

> If the route includes the special  `{_format}`  parameter, you shouldn't use the  `.+`  requirement for the parameters that allow slashes. For example, if the pattern is  `/share/{token}.{_format}`  and  `{token}`  allows any character, the  `/share/foo/bar.json`  URL will consider  `foo/bar.json`  as the token and the format will be empty. This can be solved by replacing the  `.+`  requirement by  `[^.]+`  to allow any character except dots.

## [Route Aliasing](https://symfony.com/doc/current/routing.html#route-aliasing "Permalink to this headline")

Route alias allow you to have multiple name for the same route

### [Deprecating Route Aliases](https://symfony.com/doc/current/routing.html#deprecating-route-aliases "Permalink to this headline")
```php
$routes->alias('new_route_name', 'original_route_name')
    // this outputs the following generic deprecation message:
    // Since acme/package 1.2: The "new_route_name" route alias is deprecated. You should stop using it, as it will be removed in the future.
    ->deprecate('acme/package', '1.2', '')

    // you can also define a custom deprecation message (%alias_id% placeholder is available)
    ->deprecate(
        'acme/package',
        '1.2',
        'The "%alias_id%" route alias is deprecated. Do not use it anymore.'
    )
;
```
## [Route Groups and Prefixes](https://symfony.com/doc/current/routing.html#route-groups-and-prefixes "Permalink to this headline")
```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/blog', requirements: ['_locale' => 'en|es|fr'], name: 'blog_')]
class BlogController extends AbstractController
{
    #[Route('/{_locale}', name: 'index')]
    public function index(): Response
    {
        // ...
    }

    #[Route('/{_locale}/posts/{slug}', name: 'show')]
    public function show(string $slug): Response
    {
        // ...
    }
}
```
If any of the prefixed routes defines an empty path, Symfony adds a trailing slash to it. In the previous example, an empty path prefixed with `/blog` will result in the `/blog/` URL. If you want to avoid this behavior, set the `trailing_slash_on_root` option to `false` (**this option is not available when using PHP attributes or annotations**):
```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('../../src/Controller/', 'annotation')
        // the second argument is the $trailingSlashOnRoot option
        ->prefix('/blog', false)

        // ...
    ;
};
```
get route name and parameter:
```php
        $routeName = $request->attributes->get('_route');
        $routeParameters = $request->attributes->get('_route_params');
```
You can get this information in services too injecting the  `request_stack`  service to  [get the Request object in a service](https://symfony.com/doc/current/service_container/request.html). In templates, use the  [Twig global app variable](https://symfony.com/doc/current/templates.html#twig-app-variable)  to get the current route and its attributes:
```php
{% set route_name = app.current_route %}
{% set route_parameters = app.current_route_parameters %}
```

> The  `app.current_route`  and  `app.current_route_parameters`  variables were introduced in Symfony 6.2. Before you had to access  `_route`  and  `_route_params`  request attributes using  `app.request.attributes.get()`.

### [Redirecting to URLs and Routes Directly from a Route](https://symfony.com/doc/current/routing.html#redirecting-to-urls-and-routes-directly-from-a-route "Permalink to this headline")

Use the  `RedirectController`  to redirect to other routes and URLs:
```php
// config/routes.php
use App\Controller\DefaultController;
use Symfony\Bundle\FrameworkBundle\Controller\RedirectController;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->add('doc_shortcut', '/doc')
        ->controller(RedirectController::class)
         ->defaults([
            'route' => 'doc_page',
            // optionally you can define some arguments passed to the route
            'page' => 'index',
            'version' => 'current',
            // redirections are temporary by default (code 302) but you can make them permanent (code 301)
            'permanent' => true,
            // add this to keep the original query string parameters when redirecting
            'keepQueryParams' => true,
            // add this to keep the HTTP method when redirecting. The redirect status changes:
            // * for temporary redirects, it uses the 307 status code instead of 302
            // * for permanent redirects, it uses the 308 status code instead of 301
            'keepRequestMethod' => true,
        ])
    ;

    $routes->add('legacy_doc', '/legacy/doc')
        ->controller(RedirectController::class)
         ->defaults([
            // this value can be an absolute path or an absolute URL
            'path' => 'https://legacy.example.com/doc',
            // redirections are temporary by default (code 302) but you can make them permanent (code 301)
            'permanent' => true,
        ])
    ;
};
```
#### [Redirecting URLs with Trailing Slashes](https://symfony.com/doc/current/routing.html#redirecting-urls-with-trailing-slashes "Permalink to this headline")

Historically, URLs have followed the UNIX convention of adding trailing slashes for directories (e.g.  `https://example.com/foo/`) and removing them to refer to files (`https://example.com/foo`). Although serving different contents for both URLs is OK, nowadays it's common to treat both URLs as the same URL and redirect between them.

Symfony follows this logic to redirect between URLs with and without trailing slashes (but only for  `GET`  and  `HEAD`  requests):
|title| 1 | 2|
|---|---|---|
|Route URL|If the requested URL is  `/foo`|If the requested URL is  `/foo/`|
|`/foo`|It matches (`200`  status response)|It makes a  `301`  redirect to  `/foo`|
|`/foo/`|It makes a  `301`  redirect to  `/foo/`|It matches (`200`  status response)|

The value of the `host` option can include parameters (which is useful in multi-tenant applications) and these parameters can be validated too with `requirements`:
```php
// src/Controller/MainController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class MainController extends AbstractController
{
    #[Route(
        '/',
        name: 'mobile_homepage',
        host: '{subdomain}.example.com',
        defaults: ['subdomain' => 'm'],
        requirements: ['subdomain' => 'm|mobile'],
    )]
    public function mobileHomepage(): Response
    {
        // ...
    }

    #[Route('/', name: 'homepage')]
    public function homepage(): Response
    {
        // ...
    }
}
```
> You can also use the inline defaults and requirements format in the `host` option: `{subdomain<m|mobile>?m}.example.com`

## [Localized Routes (i18n)](https://symfony.com/doc/current/routing.html#localized-routes-i18n "Permalink to this headline")

If your application is translated into multiple languages, each route can define a different URL per each  [translation locale](https://symfony.com/doc/current/translation.html#translation-locale). This avoids the need for duplicating routes, which also reduces the potential bugs:
```php
// src/Controller/CompanyController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class CompanyController extends AbstractController
{
    #[Route(path: [
        'en' => '/about-us',
        'nl' => '/over-ons'
    ], name: 'about_us')]
    public function about(): Response
    {
        // ...
    }
}
```
When using PHP attributes for localized routes, you have to use the `path` named parameter to specify the array of paths.
> When the application uses full "language + territory" locales (e.g.  `fr_FR`,  `fr_BE`), if the URLs are the same in all related locales, routes can use only the language part (e.g.  `fr`) to avoid repeating the same URLs.

A common requirement for internationalized applications is to prefix all routes with a locale. This can be done by defining a different prefix for each locale (and setting an empty prefix for your default locale if you prefer it):

```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('../../src/Controller/', 'annotation')
        ->prefix([
            // don't prefix URLs for English, the default locale
            'en' => '',
            'nl' => '/nl',
        ])
    ;
};
```
Another common requirement is to host the website on a different domain according to the locale. This can be done by defining a different host for each locale.
```php
// config/routes/annotations.php
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;
return function (RoutingConfigurator $routes) {
    $routes->import('../../src/Controller/', 'annotation')
        ->host([
            'en' => 'https://www.example.com',
            'nl' => 'https://www.example.nl',
        ])
    ;
};
```
#### Stateless route
```php
    #[Route('/', name: 'homepage', stateless: true)]
```
Now, if the session is used, the application will report it based on your  `kernel.debug`  parameter:

-   `enabled`: will throw an  [UnexpectedSessionUsageException](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpKernel/Exception/UnexpectedSessionUsageException.php "Symfony\Component\HttpKernel\Exception\UnexpectedSessionUsageException")  exception
-   `disabled`: will log a warning

If you don't set the route name explicitly with the `name` option, Symfony generates an **automatic name based on the controller and action**.

If you pass to the  `generateUrl()`  method some parameters that are not part of the route definition, they are included in the generated URL **as a query string**:
```php
$this->generateUrl('blog', ['page' => 2, 'category' => 'Symfony']);
// the 'blog' route only defines the 'page' parameter; the generated URL is:
// /blog/2?category=Symfony
```

> While objects are converted to string when used as placeholders, they are not converted when used as extra parameters. So, if you're passing an object (e.g. an Uuid) as value of an extra parameter, you need to explicitly convert it to a string:
> ```  $this->generateUrl('blog', ['uuid' => (string) $entity->getUuid()]); ```

url in service:
```php
    public function __construct(private UrlGeneratorInterface $router)
    {
    }
```
### [Generating URLs in Commands](https://symfony.com/doc/current/routing.html#generating-urls-in-commands "Permalink to this headline")

Generating URLs in commands works the same as  [generating URLs in services](https://symfony.com/doc/current/routing.html#routing-generating-urls-in-services). The only difference is that commands are not executed in the HTTP context. Therefore, if you generate absolute URLs, you'll get  `http://localhost/`  as the host name instead of your real host name.

The solution is to configure the  `default_uri`  option to define the "request context" used by commands when they generate URLs:
```yaml
# config/packages/routing.yaml
framework:
    router:
        # ...
        default_uri: 'https://example.org/my/path/'
```
if route exist
```php
use Symfony\Component\Routing\Exception\RouteNotFoundException;

// ...

try {
    $url = $this->router->generate($routeName, $routeParameters);
} catch (RouteNotFoundException $e) {
    // the route is not defined...
}
```
force https:
```yaml
# config/services.yaml
parameters:
    router.request_context.scheme: 'https'
    asset.request_context.secure: true
```
also
```php
    #[Route('/login', name: 'login', schemes: ['https'])]
```
# Twig
Twig syntax is based on these three constructs:

-   `{{ ... }}`, used to display the content of a variable or the result of evaluating an expression;
-   `{% ... %}`, used to run some logic, such as a conditional or a loop;
-   `{# ... #}`, used to add comments to the template (unlike HTML comments, these comments are not included in the rendered page).

**filters**  modify content before being rendered, like the  `upper`  filter to uppercase contents:
```
{{ title|upper }}
```

## create template 
First, you need to create a new file in the `templates/` directory to store the template contents.

```php        // get the user information and notifications somehow
        $userFirstName = '...';
        $userNotifications = ['...', '...'];

        // the template path is the relative file path from `templates/`
        return $this->render('user/notifications.html.twig', [
            // this array defines the variables passed to the template,
            // where the key is the variable name and the value is the variable value
            // (Twig recommends using snake_case variable names: 'foo_bar' instead of 'fooBar')
            'user_first_name' => $userFirstName,
            'notifications' => $userNotifications,
        ]);
```
## Template Naming

Symfony recommends the following for template names:

-   Use  [snake case](https://en.wikipedia.org/wiki/Snake_case)  for filenames and directories (e.g.  `blog_posts.html.twig`,  `admin/default_theme/blog/index.html.twig`, etc.);
-   Define two extensions for filenames (e.g.  `index.html.twig`  or  `blog_posts.xml.twig`) being the first extension (`html`,  `xml`, etc.) the final format that the template will generate.

Although templates usually generate HTML contents, they can generate any text-based format. That's why the two-extension convention simplifies the way templates are created and rendered for multiple formats.

## Variables:
When using the  `foo.bar`  notation, Twig tries to get the value of the variable in the following order:

1.  `$foo['bar']`  (array and element);
2.  `$foo->bar`  (object and public property);
3.  `$foo->bar()`  (object and public method);
4.  `$foo->getBar()`  (object and  _getter_  method);
5.  `$foo->isBar()`  (object and  _isser_  method);
6.  `$foo->hasBar()`  (object and  _hasser_  method);
7.  If none of the above exists, use  `null`  (or throw a  `Twig\Error\RuntimeError`  exception if the  [strict_variables](https://symfony.com/doc/current/reference/configuration/twig.html#config-twig-strict-variables)  option is enabled).

This allows to evolve your application code without having to change the template code (you can start with array variables for the application proof of concept, then move to objects with methods, etc.)

## URL:
Use the  `path()`  Twig function to link to these pages and pass the ***route name*** as the first argument and the route parameters as the optional second argument:
```twig
<a href="{{ path('blog_index') }}">Homepage</a>

{# ... #}

{% for post in blog_posts %}
    <h1>
        <a href="{{ path('blog_post', {slug: post.slug}) }}">{{ post.title }}</a>
    </h1>

    <p>{{ post.excerpt }}</p>
{% endfor %}
```
The `path()` function generates relative URLs. If you need to generate absolute URLs (for example when rendering templates for emails or RSS feeds), use the `url()` function, which takes the same arguments as `path()` (e.g. `<a href="{{ url('blog_index') }}"> ... </a>`).

## Linking to CSS, JavaScript and Image Assets:

```bash
composer require symfony/asset
```
You can now use the  `asset()`  function:
```php
{# the image lives at "public/images/logo.png" #}
<img src="{{ asset('images/logo.png') }}" alt="Symfony!"/>

{# the CSS file lives at "public/css/blog.css" #}
<link href="{{ asset('css/blog.css') }}" rel="stylesheet"/>

{# the JS file lives at "public/bundles/acme/js/loader.js" #}
<script src="{{ asset('bundles/acme/js/loader.js') }}"></script>
```

> The `asset()` function supports various cache busting techniques via the [version](https://symfony.com/doc/current/reference/configuration/framework.html#reference-framework-assets-version), [version_format](https://symfony.com/doc/current/reference/configuration/framework.html#reference-assets-version-format), and [json_manifest_path](https://symfony.com/doc/current/reference/configuration/framework.html#reference-assets-json-manifest-path) configuration options.

If you need **absolute URLs for assets**, use the `absolute_url()` Twig function as follows:
```php
<img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!"/>

<link rel="shortcut icon" href="{{ absolute_url('favicon.png') }}">
```
## [The App Global Variable](https://symfony.com/doc/current/templates.html#the-app-global-variable "Permalink to this headline")

Symfony creates a context object that is injected into every Twig template automatically as a variable called  `app`. It provides access to some application information:
```twig
<p>Username: {{ app.user.username ?? 'Anonymous user' }}</p>
{% if app.debug %}
    <p>Request method: {{ app.request.method }}</p>
    <p>Application Environment: {{ app.environment }}</p>
{% endif %}
```
|title|usage|
|---|---|
|app.user|The  [current user object](https://symfony.com/doc/current/security.html#create-user-class)  or  `null`  if the user is not authenticated.|
|app.request|The  [Request](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Request.php "Symfony\Component\HttpFoundation\Request")  object that stores the current  [request data](https://symfony.com/doc/current/components/http_foundation.html#accessing-request-data)  (depending on your application, this can be a  [sub-request](https://symfony.com/doc/current/components/http_kernel.html#http-kernel-sub-requests)  or a regular request).|
|app.session|The [Session](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/HttpFoundation/Session/Session.php "Symfony\Component\HttpFoundation\Session\Session") object that represents the current [user's session](https://symfony.com/doc/current/session.html) or `null` if there is none.|
|app.flashes|An array of all the [flash messages](https://symfony.com/doc/current/session.html#flash-messages) stored in the session. You can also get only the messages of some type (e.g. `app.flashes('notice')`).|
|app.environment|The name of the current  [configuration environment](https://symfony.com/doc/current/configuration.html#configuration-environments)  (`dev`,  `prod`, etc).|
|app.debug|True if in  [debug mode](https://symfony.com/doc/current/configuration/front_controllers_and_kernel.html#debug-mode). False otherwise.|
|app.token|A  [TokenInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Security/Core/Authentication/Token/TokenInterface.php "Symfony\Component\Security\Core\Authentication\Token\TokenInterface")  object representing the security token.|
|app.current_route|The name of the route associated with the current request or `null` if no request is available (equivalent to `app.request.attributes.get('_route')`)|
|app.current_route_parameters|An array with the parameters passed to the route of the current request or an empty array if no request is available (equivalent to `app.request.attributes.get('_route_params')`)|

### [Global Variables](https://symfony.com/doc/current/templates.html#global-variables "Permalink to this headline")

Twig allows you to automatically inject one or more variables ***into all templates***. These global variables are defined in the  **`twig.globals`  option** inside the main Twig configuration file:
```yaml
# config/packages/twig.yaml
twig:
    # ...
    globals:
        ga_tracking: 'UA-xxxxx-x'
```

Now, the variable  `ga_tracking`  is available in all Twig templates, so you can use it without having to pass it explicitly from the controller or service that renders the template:
```twig
<p>The Google tracking code is: {{ ga_tracking }}</p>
```

In addition to static values, Twig global variables ***can also reference services*** from the  [service container](https://symfony.com/doc/current/service_container.html). The main drawback is that these services are not loaded lazily. In other words, as soon as Twig is loaded, your service is instantiated, even if you never use that global variable.

To define a service as a global Twig variable, prefix the service ID string with the  **`@`**  character, which is the usual syntax to  [refer to services in container parameters](https://symfony.com/doc/current/service_container.html#service-container-parameters):
```yaml
# config/packages/twig.yaml
twig:
    # ...
    globals:
        # the value is the service's id
        uuid: '@App\Generator\UuidGenerator'
```

Now you can use the  `uuid`  variable in any Twig template to access to the  `UuidGenerator`  service:
```twig
UUID: {{ uuid.generate }}
```

## [Rendering Templates](https://symfony.com/doc/current/templates.html#rendering-templates "Permalink to this headline")
```php
// src/Controller/ProductController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    public function index(): Response
    {
        // ...

        // the `render()` method returns a `Response` object with the
        // contents created by the template
        return $this->render('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);

        // the `renderView()` method only returns the contents created by the
        // template, so you can use those contents later in a `Response` object
        $contents = $this->renderView('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);

        return new Response($contents);
    }
}
```

If your controller does not extend from `AbstractController`, you'll need to [fetch services in your controller](https://symfony.com/doc/current/controller.html#controller-accessing-services) and use the `render()` method of the **`twig` service**.

Another option is to use the **`#[Template()]`** attribute on the controller method to define the template to render:
```php
// src/Controller/ProductController.php
namespace App\Controller;

use Symfony\Bridge\Twig\Attribute\Template;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    #[Template('product/index.html.twig')]
    public function index()
    {
        // ...

        // when using the #[Template()] attribute, you only need to return
        // an array with the parameters to pass to the template (the attribute
        // is the one which will create and return the Response object).
        return [
            'category' => '...',
            'promotions' => ['...', '...'],
        ];
    }
}
```
## Render in service:
```php
// src/Service/SomeService.php
namespace App\Service;

use Twig\Environment;

class SomeService
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function someMethod()
    {
        // ...

        $htmlContents = $this->twig->render('product/index.html.twig', [
            'category' => '...',
            'promotions' => ['...', '...'],
        ]);
    }
}
```
## [Rendering a Template Directly from a Route](https://symfony.com/doc/current/templates.html#rendering-a-template-directly-from-a-route "Permalink to this headline"):
```yaml
# config/routes.yaml
acme_privacy:
    path:          /privacy
    controller:    Symfony\Bundle\FrameworkBundle\Controller\TemplateController
    defaults:
        # the path of the template to render
        template:  'static/privacy.html.twig'

        # the response status code (default: 200)
        statusCode: 200

        # special options defined by Symfony to set the page cache
        maxAge:    86400
        sharedAge: 86400

        # whether or not caching should apply for client caches only
        private: true

        # optionally you can define some arguments passed to the template
        context:
            site_name: 'ACME'
            theme: 'dark'
```
## [Checking if a Template Exists](https://symfony.com/doc/current/templates.html#checking-if-a-template-exists "Permalink to this headline")

```php
use Twig\Environment;

class YourService
{
    // this code assumes that your service uses autowiring to inject dependencies
    // otherwise, inject the service called 'twig' manually
    public function __construct(Environment $twig)
    {
        $loader = $twig->getLoader();
    }
}

// ....
if ($loader->exists('theme/layout_responsive.html.twig')) {
    // the template exists, do something
    // ...
}
```
lint twig
```bash
php bin/console lint:twig
```

## [Reusing Template Contents](https://symfony.com/doc/current/templates.html#reusing-template-contents "Permalink to this headline")
### [Including Templates](https://symfony.com/doc/current/templates.html#including-templates "Permalink to this headline"):
If certain Twig code is repeated in several templates, you can extract it into a single "template fragment" and include it in other templates. Imagine that the following code to display the user information is repeated in several places

The `include()` **Twig function takes as argument the path of the template to include**. The included template has access to all the variables of the template that includes it (use the [with_context](https://twig.symfony.com/doc/3.x/functions/include.html) option to control this).
```twig
{# templates/blog/index.html.twig #}

{# ... #}
{{ include('blog/_user_profile.html.twig', {user: blog_post.author}) }}
```
### [Embedding Controllers](https://symfony.com/doc/current/templates.html#embedding-controllers "Permalink to this headline")

```twig
{# templates/base.html.twig #}

{# ... #}
<div id="sidebar">
    {# if the controller is associated with a route, use the path() or url() functions #}
    {{ render(path('latest_articles', {max: 3})) }}
    {{ render(url('latest_articles', {max: 3})) }}

    {# if you don't want to expose the controller with a public URL,
       use the controller() function to define the controller to execute #}
    {{ render(controller(
        'App\\Controller\\BlogController::recentArticles', {max: 3}
    )) }}
</div>
```
When using the `controller()` function, controllers are not accessed using a regular Symfony route but through a special URL used exclusively to serve those template fragments. Configure that special URL in the `fragments` option:
```yaml
# config/packages/framework.yaml
framework:
    # ...
    fragments: { path: /_fragment }
```
### [Template Inheritance and Layouts](https://symfony.com/doc/current/templates.html#template-inheritance-and-layouts "Permalink to this headline")
The [Twig block tag](https://twig.symfony.com/doc/3.x/tags/block.html) defines the page sections that ***can be overridden in the child templates***.

> they can be overridden by the third-level inheritance template

```twig
{% extends 'blog/layout.html.twig' %}
```

When using `extends`, a child template is forbidden to define template parts outside of a block. **The following code throws a `SyntaxError`**:
```twig
{# app/Resources/views/blog/index.html.twig #}
{% extends 'base.html.twig' %}

{# the line below is not captured by a "block" tag #}
<div class="alert">Some Alert</div>

{# the following is valid #}
{% block content %}My cool blog posts{% endblock %}
```

> Use the `twig.paths` option to configure those extra directories.

Using the above configuration, if your application renders for example the `layout.html.twig` template, Symfony will first look for `email/default/templates/layout.html.twig` and `backend/templates/layout.html.twig`. If any of those templates exists, Symfony will use it instead of using `templates/layout.html.twig`, which is probably the template you wanted to use.
```yaml
# config/packages/twig.yaml
twig:
    # ...
    paths:
        'email/default/templates': 'email'
        'backend/templates': 'admin'
```
Now, if you render the `layout.html.twig` template, Symfony will render the `templates/layout.html.twig` file. Use the special syntax `@` + namespace to refer to the other namespaced templates (e.g. `@email/layout.html.twig` and `@admin/layout.html.twig`).

If you [install packages/bundles](https://symfony.com/doc/current/setup.html#symfony-flex) in your application, they may include their own Twig templates (in the `Resources/views/` directory of each bundle).

## [Create the Extension Class](https://symfony.com/doc/current/templates.html#create-the-extension-class "Permalink to this headline")
```php
// src/Twig/AppExtension.php
namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            new TwigFilter('price', [$this, 'formatPrice']),
        ];
    }

    public function formatPrice($number, $decimals = 0, $decPoint = '.', $thousandsSep = ',')
    {
        $price = number_format($number, $decimals, $decPoint, $thousandsSep);
        $price = '$'.$price;

        return $price;
    }
}
```
If you want to create a function instead of a filter, define the `getFunctions()` method:
```php
// src/Twig/AppExtension.php
namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFunction;

class AppExtension extends AbstractExtension
{
    public function getFunctions()
    {
        return [
            new TwigFunction('area', [$this, 'calculateArea']),
        ];
    }

    public function calculateArea(int $width, int $length)
    {
        return $width * $length;
    }
}
```
> Next, register your class as a service and tag it with `twig.extension`.

## [Creating Lazy-Loaded Twig Extensions](https://symfony.com/doc/current/templates.html#creating-lazy-loaded-twig-extensions "Permalink to this headline")
```php
// src/Twig/AppExtension.php
namespace App\Twig;

use App\Twig\AppRuntime;
use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;

class AppExtension extends AbstractExtension
{
    public function getFilters()
    {
        return [
            // the logic of this filter is now implemented in a different class
            new TwigFilter('price', [AppRuntime::class, 'formatPrice']),
        ];
    }
}
```
Then, create the new `AppRuntime` class (it's not required but these classes are suffixed with `Runtime` by convention) and include the logic of the previous `formatPrice()` method:
```php
// src/Twig/AppRuntime.php
namespace App\Twig;

use Twig\Extension\RuntimeExtensionInterface;

class AppRuntime implements RuntimeExtensionInterface
{
    public function __construct()
    {
        // this simple example doesn't define any dependency, but in your own
        // extensions, you'll need to inject services using this constructor
    }

    public function formatPrice($number, $decimals = 0, $decPoint = '.', $thousandsSep = ',')
    {
        $price = number_format($number, $decimals, $decPoint, $thousandsSep);
        $price = '$'.$price;

        return $price;
    }
}
```
## [Setting Variables](https://twig.symfony.com/doc/3.x/templates.html#setting-variables "Permalink to this headline")

You can assign values to variables inside code blocks. Assignments use the  [set](https://twig.symfony.com/doc/3.x/tags/set.html)  tag:

```twig
{% set foo = 'foo' %}
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
```
## [Named Arguments](https://twig.symfony.com/doc/3.x/templates.html#named-arguments "Permalink to this headline")

```twig
{% for i in range(low=1, high=10, step=2) %}
    {{ i }},
{% endfor %}
```

Using named arguments makes your templates more explicit about the meaning of the values you pass as arguments, Named arguments also allow you to skip some arguments for which you don't want to change the default value:
```twig
{{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}

{# versus #}

{{ data|convert_encoding(from='iso-2022-jp', to='UTF-8') }}
```

## Filter:

### [`batch`](https://twig.symfony.com/doc/2.x/filters/batch.html#batch "Permalink to this headline"):
The  `batch`  filter "batches" items by returning a list of lists with the given number of items. A second parameter can be provided and used to fill in missing items:
```twig
{% set items = ['a', 'b', 'c', 'd'] %}

<table>
{% for row in items|batch(3, 'No item') %}
    <tr>
        {% for column in row %}
            <td>{{ column }}</td>
        {% endfor %}
    </tr>
{% endfor %}
</table>
```
and 

```html
<table>
    <tr>
        <td>a</td>
        <td>b</td>
        <td>c</td>
    </tr>
    <tr>
        <td>d</td>
        <td>No item</td>
        <td>No item</td>
    </tr>
</table>
```
### country_name

By default, the filter uses the current locale. You can pass it explicitly:
```
{# États-Unis #}
{{ 'US'|country_name('fr') }}
```
***Note*:** The  `country_name`  filter is part of the  `IntlExtension`  which is not installed by default. Install it first:
```bash
composer require twig/intl-extra
```
Then, on Symfony projects, install the  `twig/extra-bundle`:
```bash
composer require twig/extra-bundle
```
Otherwise, add the extension explicitly on the Twig environment:
```php
use Twig\Extra\Intl\IntlExtension;

$twig = new \Twig\Environment(...);
$twig->addExtension(new IntlExtension());
```

When using automatic escaping, Twig tries to not double-escape a variable when the automatic escaping strategy is the same as the one applied by the escape filter; but that does not work when using a variable as the escaping strategy:
```twig
{% set strategy = 'html' %}

{% autoescape 'html' %}
    {{ var|escape('html') }}   {# won't be double-escaped #}
    {{ var|escape(strategy) }} {# will be double-escaped #}
{% endautoescape %}
```
When using a variable as the escaping strategy, you should disable automatic escaping:
```twig
{% set strategy = 'html' %}

{% autoescape 'html' %}
    {{ var|escape(strategy)|raw }} {# won't be double-escaped #}
{% endautoescape %}
```

### [Custom Escapers](https://twig.symfony.com/doc/2.x/filters/escape.html#custom-escapers "Permalink to this headline")

You can define custom escapers by calling the  `setEscaper()`  method on the escaper extension instance. The first argument is the escaper name (to be used in the  `escape`  call) and the second one must be a valid PHP callable:
```php
$twig = new \Twig\Environment($loader);
$twig->getExtension(\Twig\Extension\EscaperExtension::class)->setEscaper('csv', 'csv_escaper');
```
### [`raw`](https://twig.symfony.com/doc/2.x/filters/raw.html#raw "Permalink to this headline")

The  `raw`  filter marks the value as being "safe", which means that in an environment with automatic escaping enabled this variable will not be escaped if  `raw`  is the last filter applied to it:
```twig
{% autoescape %}
    {{ var|raw }} {# var won't be escaped #}
{% endautoescape %}
```
### [`reduce`](https://twig.symfony.com/doc/2.x/filters/reduce.html#reduce "Permalink to this headline")

The  `reduce`  filter iteratively reduces a sequence or a mapping to a single value using an arrow function, so as to reduce it to a single value. The arrow function receives the return value of the previous iteration and the current value of the sequence or mapping:
```twig
{% set numbers = [1, 2, 3] %}

{{ numbers|reduce((carry, v) => carry + v) }}
{# output 6 #}
```
### [`slug`](https://twig.symfony.com/doc/2.x/filters/slug.html#slug "Permalink to this headline")

The  `slug`  filter transforms a given string into another string that only includes safe ASCII characters.

### [`spaceless`](https://twig.symfony.com/doc/2.x/filters/spaceless.html#spaceless "Permalink to this headline")
Use the  `spaceless`  filter to remove whitespace  _between HTML tags_, not whitespace within HTML tags or whitespace in plain text:
```twig
{{
    "<div>
        <strong>foo</strong>
    </div>
    "|spaceless }}

{# output will be <div><strong>foo</strong></div> #}
```

You can combine  `spaceless`  with the  `apply`  tag to apply the transformation on large amounts of HTML:
```twig
{% apply spaceless %}
    <div>
        <strong>foo</strong>
    </div>
{% endapply %}

{# output will be <div><strong>foo</strong></div> #}
```
### [`u`](https://twig.symfony.com/doc/2.x/filters/u.html#u "Permalink to this headline")
The  `u`  filter wraps a text in a Unicode object (a  [Symfony UnicodeString instance](https://symfony.com/doc/current/components/string.html)) that exposes methods to "manipulate" the string.

Let's see some common use cases.

Wrapping a text to a given number of characters:
```twig
{{ 'Symfony String + Twig = <3'|u.wordwrap(5) }}
Symfony
String
+
Twig
= <3
```
Truncating a string:
```twig
{{ 'Lorem ipsum'|u.truncate(8) }}
Lorem ip

{{ 'Lorem ipsum'|u.truncate(8, '...') }}
Lorem...
```

The  `truncate`  method also accepts a third argument to preserve whole words:
```twig
{{ 'Lorem ipsum dolor'|u.truncate(10, '...', false) }}
Lorem ipsum...
```
Converting a string to  _snake_  case or  _camelCase_:
```twig
{{ 'SymfonyStringWithTwig'|u.snake }}
symfony_string_with_twig

{{ 'symfony_string with twig'|u.camel.title }}
SymfonyStringWithTwig
```
You can also chain methods:
```twig
{{ 'Symfony String + Twig = <3'|u.wordwrap(5).upper }}
SYMFONY
STRING
+
TWIG
= <3
```
For large strings manipulation, use the  `apply`  tag:
```twig
{% apply u.wordwrap(5) %}
    Some large amount of text...
{% endapply %}
```
## Functions:
### [`attribute`](https://twig.symfony.com/doc/2.x/functions/attribute.html#attribute "Permalink to this headline")

The  `attribute`  function can be used to access a "dynamic" attribute of a variable:
```twig
{{ attribute(object, method) }}
{{ attribute(object, method, arguments) }}
{{ attribute(array, item) }}
```

In addition, the  `defined`  test can check for the existence of a dynamic attribute:
```twig
{{ attribute(object, method) is defined ? 'Method exists' : 'Method does not exist' }}
```
### [`parent`](https://twig.symfony.com/doc/2.x/functions/parent.html#parent "Permalink to this headline")

When a template uses inheritance, it's possible to render the contents of the parent block when overriding a block by using the  `parent`.
### [`source`](https://twig.symfony.com/doc/2.x/functions/source.html#source "Permalink to this headline")

The  `source`  function returns the content of a template without rendering it

## Tags:
### Block:
Block names must consist of alphanumeric characters, and underscores. The first char can't be a digit and dashes are not permitted.

### [`do`](https://twig.symfony.com/doc/2.x/tags/do.html#do "Permalink to this headline")
The  `do`  tag works exactly like the regular variable expression (`{{ ... }}`) just that it doesn't print anything:
```twig
{% do 1 + 2 %}
```
### [`embed`](https://twig.symfony.com/doc/2.x/tags/embed.html#embed "Permalink to this headline")

The  `embed`  tag combines the behavior of  [include](https://twig.symfony.com/doc/2.x/tags/include.html)  and  [extends](https://twig.symfony.com/doc/2.x/tags/extends.html). It allows you to include another template's contents, just like  `include`  does. But it also allows you to override any block defined inside the included template, like when extending a template.

### [`extends`](https://twig.symfony.com/doc/2.x/tags/extends.html#extends "Permalink to this headline")

The  `extends`  tag can be used to extend a template from another one.

> Note: Like PHP, Twig does not support ***multiple inheritance***. So you can only have one extends tag called per rendering. However, Twig supports horizontal  [reuse](https://twig.symfony.com/doc/2.x/tags/use.html).

### [`for`](https://twig.symfony.com/doc/2.x/tags/for.html#for "Permalink to this headline")

Loop over each item in a sequence. For example, to display a list of users provided in a variable called  `users`:
```twig
<h1>Members</h1>
<ul>
    {% for user in users %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```
> Note: The `loop.length`, `loop.revindex`, `loop.revindex0`, and `loop.last` variables are only available for PHP arrays, or objects that implement the `Countable` interface. They are also not available when looping with a condition.

Unlike in PHP, it's not **possible** to `break` or `continue` in a loop. You can however filter the sequence during iteration which allows you to skip items. The following example skips all the users which are not active:
```twig
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```
> Note: Using the  `loop`  variable within the condition is not recommended as it will probably not be doing what you expect it to. For instance, adding a condition like  `loop.index > 4`  won't work as the index is only incremented when the condition is true (so the condition will never match).

### [The `else` Clause](https://twig.symfony.com/doc/2.x/tags/for.html#the-else-clause "Permalink to this headline")

If no iteration took place because the sequence was empty, you can render a replacement block by using  `else`:
```twig
<ul>
    {% for user in users %}
        <li>{{ user.username|e }}</li>
    {% else %}
        <li><em>no user found</em></li>
    {% endfor %}
</ul>
```

### If
The rules to determine if an expression is  `true`  or  `false`  are the same as in PHP; here are the edge cases rules:
|title|description|
|---|---|
|Value|Boolean evaluation|
|empty string|false|
|numeric zero|false|
|NAN (Not A Number)|true|
|INF (Infinity)|true|
|whitespace-only string|true|
|string "0" or '0'|false|
|empty array|false|
|null|false|
|non-empty array|true|
|object|true|

### Use
The  `use`  statement tells Twig to import the blocks defined in  `blocks.html`  into the current template (it's like macros, but for blocks):
```twig
{# blocks.html #}

{% block sidebar %}{% endblock %}
```
### [`with`](https://twig.symfony.com/doc/2.x/tags/with.html#with "Permalink to this headline")

Use the  `with`  tag to create a new inner scope. Variables set within this scope are not visible outside of the scope:
```twig
{% with %}
    {% set foo = 42 %}
    {{ foo }} {# foo is 42 here #}
{% endwith %}
foo is not visible here any longer
```
# Forms
```bash
composer require symfony/form
```
## [Usage](https://symfony.com/doc/current/forms.html#usage "Permalink to this headline")

The recommended workflow when working with Symfony forms is the following:

1.  **Build the form**  in a Symfony controller or using a dedicated form class;
2.  **Render the form**  in a template so the user can edit and submit it;
3.  **Process the form**  to validate the submitted data, transform it into PHP data and do something with it (e.g. persist it in a database).

If your controller does not extend from `AbstractController`, you'll need to [fetch services in your controller](https://symfony.com/doc/current/controller.html#controller-accessing-services) and use the `createBuilder()` method of the `form.factory` service.

Form classes are [form types](https://symfony.com/doc/current/forms.html#form-types) that implement [FormTypeInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Form/FormTypeInterface.php "Symfony\Component\Form\FormTypeInterface"). However, it's better to extend from [AbstractType](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Form/AbstractType.php "Symfony\Component\Form\AbstractType"), which already implements the interface and provides some utilities;

The form class contains all the directions needed to create the task form. In controllers extending from the [AbstractController](https://symfony.com/doc/current/controller.html#the-base-controller-class-services), use the `createForm()` helper (otherwise, use the `create()` method of the `form.factory` service):

```php
    // ... default 

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Task::class,
        ]);
    }
```
Internally, the `render()` method calls `$form->createView()` to transform the form into a _form view_ instance.
```twig
{# templates/task/new.html.twig #}
{{ form(form) }}
```

## [Form Type Options Guessing](https://symfony.com/doc/current/forms.html#form-type-options-guessing "Permalink to this headline")

|title|description|
|---|---|
|required|The  `required`  option is guessed based on the validation rules (i.e. is the field  `NotBlank`  or  `NotNull`) or the Doctrine metadata (i.e. is the field  `nullable`). This is very useful, as your client-side validation will automatically match your validation rules.|
|maxlength|If the field is some sort of text field, then the  `maxlength`  option attribute is guessed from the validation constraints (if  `Length`  or  `Range`  is used) or from the  [Doctrine](https://symfony.com/doc/current/doctrine.html)  metadata (via the field's length).|

When editing an object via a form, all form fields are considered properties of the object. Any fields on the form that do not exist on the object will ***cause an exception to be thrown***.
set the `mapped` option to `false` in those fields:
```php
// ...
use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\FormBuilderInterface;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('task')
            ->add('dueDate')
            ->add('agreeTerms', CheckboxType::class, ['mapped' => false])
            ->add('save', SubmitType::class)
        ;
    }
}
```
## [Applying Themes to all Forms](https://symfony.com/doc/current/form/form_themes.html#applying-themes-to-all-forms "Permalink to this headline")
```yaml
# config/packages/twig.yaml
twig:
    form_themes: ['bootstrap_5_horizontal_layout.html.twig']
    # ...
```
single form:

```twig
{# this form theme will be applied only to the form of this template #}
{% form_theme form 'foundation_5_layout.html.twig' %}
{# apply multiple form themes but only to the form of this template #}

{% form_theme form with [
    'foundation_5_layout.html.twig',
    'form/my_custom_theme.html.twig'
] %}

{# ... #}
{% form_theme form.a_child_form 'form/my_custom_theme.html.twig' %}
```

## [Disabling Global Themes for Single Forms](https://symfony.com/doc/current/form/form_themes.html#disabling-global-themes-for-single-forms "Permalink to this headline")

Global form themes defined in the app are always applied to all forms, even those which use the  `form_theme`  tag to apply their own themes. You may want to disable this for example when creating an admin interface for a bundle which can be installed on different Symfony applications (and so you can't control what themes are enabled globally). To do that, add the  `only`  keyword after the list of form themes:

```twig
{% form_theme form with ['foundation_5_layout.html.twig'] only %}

{# ... #}
```
When using the  `only`  keyword, none of Symfony's built-in form themes (`form_div_layout.html.twig`, etc.) will be applied. In order to render your forms correctly, you need to either provide a fully-featured form theme yourself, or extend one of the built-in form themes with Twig's  `use`  keyword instead of  `extends`  to re-use the original theme contents.

```twig
{# templates/form/common.html.twig #}
{% use "form_div_layout.html.twig" %}

{# ... #}
```
## [Form Fragment Naming](https://symfony.com/doc/current/form/form_themes.html#form-fragment-naming "Permalink to this headline")

The naming of form fragments varies depending on your needs:

-   If you want to customize  **all fields of the same type**  (e.g. all  `<textarea>`) use the  `field-type_field-part`  pattern (e.g.  `textarea_widget`).
-   If you want to customize  **only one specific field**  (e.g. the  `<textarea>`  used for the  `description`  field of the form that edits products) use the  `_field-id_field-part`  pattern (e.g.  `_product_description_widget`).

## [Fragment Naming for All Fields of the Same Type](https://symfony.com/doc/current/form/form_themes.html#fragment-naming-for-all-fields-of-the-same-type "Permalink to this headline")

These fragment names follow the  `type_part`  pattern, where the  `type`  corresponds to the field  _type_  being rendered (e.g.  `textarea`,  `checkbox`,  `date`, etc) and the  `part`  corresponds to  _what_  is being rendered (e.g.  `label`,  `widget`, etc.)

A few examples of fragment names are:

-   `form_row`  - used by  [form_row()](https://symfony.com/doc/current/form/form_customization.html#reference-forms-twig-row)  to render most fields;
-   `textarea_widget`  - used by  [form_widget()](https://symfony.com/doc/current/form/form_customization.html#reference-forms-twig-widget)  to render a  `textarea`  field type;
-   `form_errors`  - used by  [form_errors()](https://symfony.com/doc/current/form/form_customization.html#reference-forms-twig-errors)  to render errors for a field;

The  `id`  attribute contains both the form name and the field name (e.g.  `product_price`). The form name can be set manually or generated automatically based on your form type name (e.g.  `ProductType`  equates to  `product`). If you're not sure what your form name is, look at the HTML code rendered for your form. **You can also define this value explicitly with the  `block_name`  option:**
```php
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options): void
{
    // ...

    $builder->add('name', TextType::class, [
        'block_name' => 'custom_name',
    ]);
}
```

In this example, the fragment name will be  `_product_custom_name_widget`  instead of the default  `_product_name_widget`.

> Symfony uses Twig blocks to render each part of a form.

A _theme_ is a Twig template with one or more of those **blocks** that you want to use when rendering a form.

#### [Custom Fragment Naming for Individual Fields](https://symfony.com/doc/current/form/form_themes.html#custom-fragment-naming-for-individual-fields "Permalink to this headline")

The  **`block_prefix`**  option allows form fields to define their own custom fragment name. This is mostly useful to customize some instances of the same field without having to  [create a custom form type](https://symfony.com/doc/current/form/create_custom_field_type.html):
```php
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options): void
{
    $builder->add('name', TextType::class, [
        'block_prefix' => 'wrapped_text',
    ]);
}
```
Now you can use  `wrapped_text_row`,  `wrapped_text_widget`, etc. as the block names.

#### [Fragment Naming for Collections](https://symfony.com/doc/current/form/form_themes.html#fragment-naming-for-collections "Permalink to this headline")

When using a  [collection of forms](https://symfony.com/doc/current/form/form_collections.html), you have several ways of customizing the collection and each of its entries. First, use the following blocks to customize each part of all form collections:
```twig
{% block collection_row %} ... {% endblock %}
{% block collection_label %} ... {% endblock %}
{% block collection_widget %} ... {% endblock %}
{% block collection_help %} ... {% endblock %}
{% block collection_errors %} ... {% endblock %}
```
You can also customize each **entry**(model) of all collections with the following blocks:
```twig
{% block collection_entry_row %} ... {% endblock %}
{% block collection_entry_label %} ... {% endblock %}
{% block collection_entry_widget %} ... {% endblock %}
{% block collection_entry_help %} ... {% endblock %}
{% block collection_entry_errors %} ... {% endblock %}
```
Finally, you can customize specific form collections instead of all of them. For example, consider the following complex example where a `TaskManagerType` has a collection of `TaskListType` which in turn has a collection of `TaskType`.

Then you get all the following customizable blocks (where  `*`  can be replaced by  `row`,  `widget`,  `label`, or  `help`):
```twig
{% block _task_manager_task_lists_* %}
    {# the collection field of TaskManager #}
{% endblock %}

{% block _task_manager_task_lists_entry_* %}
    {# the inner TaskListType #}
{% endblock %}

{% block _task_manager_task_lists_entry_tasks_* %}
    {# the collection field of TaskListType #}
{% endblock %}

{% block _task_manager_task_lists_entry_tasks_entry_* %}
    {# the inner TaskType #}
{% endblock %}

{% block _task_manager_task_lists_entry_tasks_entry_name_* %}
    {# the field of TaskType #}
{% endblock %}
```

### [Template Fragment Inheritance](https://symfony.com/doc/current/form/form_themes.html#template-fragment-inheritance "Permalink to this headline")

Each field type has a  _parent_  type (e.g. the parent type of  `textarea`  is  `text`, and the parent type of  `text`  is  `form`) and Symfony uses the fragment for the parent type if the base fragment doesn't exist.

When Symfony renders for example the errors for a textarea type, it looks first for a  `textarea_errors`  fragment before falling back to the  `text_errors`  and  `form_errors`  fragments.

The "parent" type of each field type is available in the  [form type reference](https://symfony.com/doc/current/reference/forms/types.html)  for each field type.

### [Creating a Form Theme in the same Template as the Form](https://symfony.com/doc/current/form/form_themes.html#creating-a-form-theme-in-the-same-template-as-the-form "Permalink to this headline")

This is recommended when doing customizations specific to a single form in your app, such as changing all  `<textarea>`  elements of a form or customizing a very special form field which will be handled with JavaScript.

You only need to add the special  `{% form_theme form _self %}`  tag to the same template where the form is rendered. This causes Twig to look inside the template for any overridden form blocks

## CSRF Protection
 ```bash
composer require symfony/security-csrf
```

```yaml
# config/packages/framework.yaml
framework:
    # ...
    csrf_protection: ~
```
By default Symfony adds the CSRF token in a hidden field called `_token`, but this can be customized on a form-by-form basis:
```php
// src/Form/TaskType.php
namespace App\Form;
// ...
use App\Entity\Task;
use Symfony\Component\OptionsResolver\OptionsResolver;

class TaskType extends AbstractType
{
    // ...

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class'      => Task::class,
            // enable/disable CSRF protection for this form
            'csrf_protection' => true,
            // the name of the hidden HTML field that stores the token
            'csrf_field_name' => '_token',
            // an arbitrary string used to generate the value of the token
            // using a different string for each form improves its security
            'csrf_token_id'   => 'task_item',
        ]);
    }

    // ...
}
```
You can also customize the rendering of the CSRF form field creating a custom  [form theme](https://symfony.com/doc/current/form/form_themes.html)  and using  `csrf_token`  as the prefix of the field (e.g. define  `{% block csrf_token_widget %} ... {% endblock %}`  to customize the entire form field contents).

## File:

```php
// src/Form/ProductType.php
namespace App\Form;

use App\Entity\Product;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\FileType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Validator\Constraints\File;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            // ...
            ->add('brochure', FileType::class, [
                'label' => 'Brochure (PDF file)',

                // unmapped means that this field is not associated to any entity property
                'mapped' => false,

                // make it optional so you don't have to re-upload the PDF file
                // every time you edit the Product details
                'required' => false,

                // unmapped fields can't define their validation using annotations
                // in the associated entity, so you can use the PHP constraint classes
                'constraints' => [
                    new File([
                        'maxSize' => '1024k',
                        'mimeTypes' => [
                            'application/pdf',
                            'application/x-pdf',
                        ],
                        'mimeTypesMessage' => 'Please upload a valid PDF document',
                    ])
                ],
            ])
            // ...
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Product::class,
        ]);
    }
}
```
## Data Transformers:
> When a form field has the `inherit_data` option set to `true`, data transformers are not applied to that field.

### [Example #1: Transforming Strings Form Data Tags from User Input to an Array](https://symfony.com/doc/current/form/data_transformers.html#example-1-transforming-strings-form-data-tags-from-user-input-to-an-array "Permalink to this headline")

Suppose you have a Task form with a tags  `text`  type:
```php
// src/Form/Type/TaskType.php
namespace App\Form\Type;

use Symfony\Component\Form\CallbackTransformer;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
// ...

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('tags', TextType::class);

        $builder->get('tags')
            ->addModelTransformer(new CallbackTransformer(
                function ($tagsAsArray) {
                    // transform the array to a string
                    return implode(', ', $tagsAsArray);
                },
                function ($tagsAsString) {
                    // transform the string back to an array
                    return explode(', ', $tagsAsString);
                }
            ))
        ;
    }

    // ...
}
```
The  `CallbackTransformer`  takes two callback functions as arguments. The first transforms the original value into a format that'll be used to render the field. The second does the reverse: it transforms the submitted value back into the format you'll use in your code.

> The `addModelTransformer()` method accepts _any_ object that implements [DataTransformerInterface](https://github.com/symfony/symfony/blob/6.2/src/Symfony/Component/Form/DataTransformerInterface.php "Symfony\Component\Form\DataTransformerInterface") - so you can create your own classes, instead of putting all the logic in the form (see the next section).


