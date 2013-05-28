# Facades

- [Introduction](#introduction)
- [Explanation](#explanation)
- [Practical Usage](#practical-usage)
- [Creating Facades](#creating-facades)
- [Mocking Facades](#mocking-facades)

<a name="introduction"></a>
## Introduction

Facades are special classes that are designed to simplify your code. Laravel comes with many facades and you have probably been using them without even knowing it.

You may wish to create your own facades for your applications and packages. Let's go over the concept, development and usage of facade classes.

> **Note:** Before digging into facades, it is strongly recommended that you become very familiar with the Laravel [IoC container](/docs/ioc).

<a name="explanation"></a>
## Explanation

A facade is nothing more than a way to provide shorter syntax to methods on objects from the [IoC container](/docs/ioc).

In the context of a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class. Laravel's facades and yours will extend the `Facade` class.

Your facade class only needs to implement a single method, `getFacadeAccessor`. It's the `getFacadeAccessor` method's job to define what to resolve from the container. The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to the object resolved from the container.

<a name="practical-usage"></a>
## Practical Usage

In the examle below, a call is made to the Laravel cache system. In this case, it appears that the static method `get` is being called on the `Cache` class.

	$value = Cache::get('key');

However, if we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`.

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

The Cache class extends the base `Facade` class and defines a method `getFacadeAccessor()`. Remember, this method's job is to return the name of an IoC binding.

When a user references any static method on the `Cache` facade, Laravel resolves that IoC binding from the container and runs the requested method (in this case, `get`) against that object.

So, our `Cache::get` call could be re-written like so:

	$value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## Creating Facades

Creating a facade for your own application or package is simple. You only need 3 things:

- an IoC binding
- a facade class
- a facade alias configuration

Let's look at an example. Here, we have a class that can be referenced as `PaymentGateway\Payment`.

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

We need to be able to resolve this class from the IoC container. So, let's add a binding.

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

A great place to register this binding would be to create a new [service provider](/docs/ioc#service-providers) named `PaymentServiceProvider`, and add this binding to the `register` method. You can then configure Laravel to load your service provider from the `app/config/app.php` configuration file.

Next, we can create our own facade class.

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

Next, if we wish, we can add an alias for our facade to the `aliases` array in the `app/config/app.php` configuration file. Now, we can call the `process` method on an instance of the `Payment` class.

	Payment::process();

<a name="mocking-facades"></a>
## Mocking Facades

Unit testing is an important aspect of why facades work the way that they do. In fact, testability is the primary reason for facades to even exist. For more information, check out the [mocking facades](/docs/testing#mocking-facades) section of the documentation.