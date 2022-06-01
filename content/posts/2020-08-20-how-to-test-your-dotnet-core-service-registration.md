---
title: How to Test Your .NET Core Service Registration
date: 2020-08-09 11:17 -500
draft: false
---

One of the most common runtime exceptions or errors I face using Dot Net
Core is that a service is not registered for Dependency Injection. This
post focuses on the Microsoft.Extensions.DependencyInjection library.
Registering dependencies is a required step anytime a new service created
and I often forget. I have found a way to easily test and ensure services
are generated properly from the service provider (also known as the 
dependency injection container).

Below is a sample extension method used by a class library to register 
its services in the Applications container. This is fairly common 
practice with class libraries. I often forget to add to these as the 
number of services increase.

```cs
namespace MyApp.Services
{
	public static class RegisterServices
    {
    	 public static IServiceCollection UseTextServices(this IServiceCollection services)
         {
             services.AddScoped<ITextService,TextService>();
         	 return services;
         }
    }
}
```

Let's take a look at a way to test and ensure we generate our service
from the provider without error. The method below builds a ServiceProvider
object using our extension method. After the service provider is built,
we can use the GetService<T>()  to retrieve our service and ensure it 
returns the correct type. When using the GetService<T>() method, you 
want to use the interface like you would when implementing through a
constructor. 

```cs
namespace MyApp.Test
{
	public class RegisterServicesTest
    {
        [Fact]
        public void TestServiceRegistration()
        {
        	ServiceProvider provider = new ServiceCollection()
                .UseTextServices()
                .BuildServiceProvider();
          
            Assert.IsType<TextService>(provider.GetService<ITextService>());
        }
    }
}
```

Let's take a look at a way to test and ensure we generate our service
from the provider without error. The method below builds a 
ServiceProvider object using our extension method. After the service
provider is built, we can use the GetService<T>()  to retrieve our
service and ensure it returns the correct type. When using the
GetService<T>() method, you want to use the interface like you
would when implementing through a constructor. 

```cs
namespace MyApp.Test
{
	public class RegisterServicesTest
    {
        [Fact]
        public void TestServiceRegistration()
        {
        	ServiceProvider provider = new ServiceCollection()
                .UseTextServices()
                .BuildServiceProvider();
          
            Assert.IsType<TextService>(provider.GetService<ITextService>());
        }
    }
}
```

This not only tests that services are registered, but anything those
services depend on and so on. I would recommend adding unit tests to
ensure your dependency injection provider is setup properly.
