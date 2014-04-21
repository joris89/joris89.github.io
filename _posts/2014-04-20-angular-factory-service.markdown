---
layout: post
title:  "Factories, Services and Providers in Angular JS"
date:   2014-04-21 01:21:36
---

When I started to build my first application with AngularJS, I never knew whether I should use a factory, a provider or a service. So I started to look how the guys of this framework implemented those 3 services. I realized, that the factories and services are based on the provider. When you understand the provider, you will understand the factory and the service as well. So let's begin with the provider:

<hr />

<h3>Provider</h3>

A basic provider looks like this:
{% highlight javascript %}
var myApp = angular.module('myApp', []);

myApp.provider('helloProvider', function() {
	this.name = '';

	var setName = function(name) {
		this.name = name;
	}

	var sayHello = function() {
		return "Hello " + this.name;
	}

    this.$get = function () {
        return {
        	sayHello: sayHello,
        	setName: setName
        };
    };
});
{% endhighlight %}

And this is how it looks like in AngularJS:
{% highlight javaScript %}
function provider(name, provider_) {
	assertNotHasOwnProperty(name, 'service');
	if (isFunction(provider_) || isArray(provider_)) {
  		provider_ = providerInjector.instantiate(provider_);
	}
	if (!provider_.$get) {
  		throw $injectorMinErr(
	  		'pget', 
	  		"Provider '{0}' must define $get factory method.", name
  		);
	}
	return providerCache[name + providerSuffix] = provider_;
}
{% endhighlight %}

<h3>Factory</h3>

A factory looks like this:
{% highlight javaScript %}
var myApp = angular.module('myApp', []);

myApp.factory('helloFactory', function() {
    return {
        sayHello: function(name) {
            return 'Hello ' + name + '!';
        }
    }
});

{% endhighlight %}

And again, the sourcecode in AngularJS:
{% highlight javaScript %}
function factory(name, factoryFn) 
{ 
	return provider(name, { 
		$get: factoryFn 
	}); 
}
{% endhighlight %}

So basically a factory is a provider but it only implements the $get method.

<h3>Service</h3>

Again, at first a simple example:
{% highlight javaScript %}
var myApp = angular.module('myApp', []);

myApp.service('helloService', function() {
	this.sayHello = function(name) {
		return 'Hello ' + name + '!';
	};
});
{% endhighlight %}

Sourcecode in AngularJS:
{% highlight javaScript %}
function service(name, constructor) {
	return factory(name, ['$injector', function($injector) {
		return $injector.instantiate(constructor);
	}]);
}
{% endhighlight %}

A service instantiates a 'class' and everything that uses this service will get the same instance!

<hr />

And this is how you use this 3 services in you controller:
{% highlight javaScript %}
var myApp = angular.module('myApp');

myApp.controller('helloController', function(
		$scope, 
		helloProvider, 
		helloFactory, 
		helloService
	) {
		helloProvider.setName('world');

		$scope.helloFromProvider = helloProvider.sayHello();
		$scope.helloFromFactory = helloFactory.sayHello('world!');
		$scope.helloFromService = helloService.sayHello('world!');
	}
);
{% endhighlight %}

<hr />

For the future, when you don't know which one to choose, just remember the following:

<ul>
	<li>
		If you want to call your service like normal function, implement a factory.
	</li>
	<li>
		If you want your service to be called with the 'new' operator, use a service.
	</li>
	<li>
		If you need more then that, choose the provider. It's the most configurable of all three. 
	</li>
</ul>
