# SimpleSoft - DependencyInjection
Library that helps to register services into the `IServiceCollection`, Microsoft's dependency injection facade, by enabling the developer to scan assemblies for service attributes or configurators.

## Reasons
Some of the reasons to use this little library:

1. Enables assembly scan;
2. Use `IServiceConfigurator` to aggregate related registrations;

## Installation 
This library can be installed via [NuGet](https://www.nuget.org/packages/SimpleSoft.DependencyInjection/) package. Just run the following command:

```powershell
Install-Package SimpleSoft.DependencyInjection -Pre
```

## Compatibility

This library is compatible with the folowing frameworks:

* .NET Framework 4.5
* .NET Standard 1.0

## Tipical usage
```csharp
    using System;
    using Microsoft.Extensions.DependencyInjection;
    using SimpleSoft.DependencyInjection;

    public class Program
    {
        public static void Main(string[] args)
        {
            var provider =
                new ServiceCollection()
                    .AddServicesFrom<Program>()
                    .BuildServiceProvider();

            foreach (var service in provider.GetServices<IProvider>())
            {
                Console.WriteLine(service);
            }

            Console.WriteLine(provider.GetRequiredService<IFacebookProvider>());
            Console.WriteLine(provider.GetRequiredService<IGoogleProvider>());
        }
    }

    public interface IProvider { }

    public abstract class Provider : IProvider
    {
        private readonly ProviderSettings _settings;

        protected Provider(ProviderSettings settings)
        {
            _settings = settings;
        }

		public override string ToString()
        {
            return $"{{ Id : '{_settings.Id:D}' }}";
        }
    }

    public class ProviderSettings
    {
        public Guid Id { get; set; } = Guid.NewGuid();
    }

    public interface IFacebookProvider : IProvider { }

    public interface IGoogleProvider : IProvider { }
    
    //	will be added as a transient service for IGoogleProvider and IProvider
    [Service(ServiceLifetime.Transient)]
    public class GoogleProvider : Provider, IGoogleProvider
    {
        public GoogleProvider(ProviderSettings settings) : base(settings)
        {

        }
    }
    
    //	Will be added as a singleton service for IFacebookProvider and FacebookProvider
    [Service(TypesToRegister = new[] {typeof(FacebookProvider), typeof(IFacebookProvider)})]
    public class FacebookProvider : Provider, IFacebookProvider
    {
        public FacebookProvider(ProviderSettings settings) : base(settings)
        {

        }
    }
    
    //	Will also be loaded
    public class ServiceConfigurator : IServiceConfigurator
    {
        public void Configure(IServiceCollection services)
        {
            services.AddTransient(k => new ProviderSettings
            {
                Id = Guid.NewGuid()
            });
        }
    }
```


