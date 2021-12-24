# Repository-Pattern
Data Access Patterns: Repository Pattern


This is an extremely common pattern to apply in front of your data layer. It will encapsulate the logic to communicate with your data layer so that the consumer of your repository doesn't have to worry about if you're using an entity framework. If you're communicating with a file system, if you're using in hibernate or any other

what if the application does not leverage the repository pattern

    1- the application lacks testability
    2- the controller is now tightly coupled with the data access layer.
    3- code duplication when we want to leverage the same data access somewhere else
    4-  not very good for maintainability

Benefits of the repository pattern

    1- the controller is now separated from our data layer
    2- It's easy for us to write a test without having any side effects
    3- we can also modify and extend entities inside the repository before they are passed back to our consumer
    4- a shareable abstraction, resulting in less duplication of code

Example on Generic repository

we will apply the generic repository to decouple the controller from the data layer And in order for us to do that, we're first going to introduce the interface of a repository. a repository is in charge of all the crud operations, which is create, read, update and delete. Our repository is going to allow us to add an entity it's going to allow us to update on the entity, and it's going to allow us to retrieve a particular entity. Based on the identifiers I also want a way for us to retrieve all the items as well as find a particular item based on some search criteria. This parameter passed to our fine function known as the predicate, will be an expression that will evaluate if a particular item should be returned or not from the underlying data store. And of course, we also want to be able to save the changes that we might have done to our different entities. So this here is the interface that our controller is now going to be working with instead of directly communicating with our data access layer,
```CSharp
public interface IRepository<T>
    {
        T Add(T entity);
        T Update(T entity);
        T Get(Guid id);
        IEnumerable<T> All();
        IEnumerable<T> Find(Expression<Func<T, bool>> predicate);
        void SaveChanges();
    }
```
it's more than okay to introduce more than one repository in an application. In reality, though, a lot of these different repositories have the same type of code for retrieving, getting, finding, and listing all the items. So how about we introduce a generic repository that introduces these base functionality that all of the different repositories can leverage? And then if we have particular implementations for other types of repositories, we can introduce that in a subclass. So we'll introduce a generic repository that is going to implement our interface, And we also want to make sure that we only work with reference types. So we were going to say here that is required to be a class. So now we can proceed to implement this interface. In order for us to implement these methods, we, of course, need to work with our data context. So this is pretty much the only place in the application where we will know about our shopping context, and we're going to make sure that we pass our shopping context into the constructor off our repository. So now that we set up one of these repositories, we will know that we can work with our shopping context, which is our underlying data structure. It could as well have been the SQL Connection and SQL Command or in hibernate the consumer off our generic repository won't know the difference. So let's go ahead and implement the methods.
```CSharp
public abstract class GenericRepository<T>
        : IRepository<T> where T : class
    {
        protected ShoppingContext context;

        public GenericRepository(ShoppingContext context)
        {
            this.context = context;
        }

        public virtual T Add(T entity)
        {
            return context
                .Add(entity)
                .Entity;
        }

        public virtual IEnumerable<T> Find(Expression<Func<T, bool>> predicate)
        {
            return context.Set<T>()
                .AsQueryable()
                .Where(predicate).ToList();
        }

        public virtual T Get(Guid id)
        {
            return context.Find<T>(id);
        }

        public virtual IEnumerable<T> All()
        {
            return context.Set<T>()
                .ToList();
        }

        public virtual T Update(T entity)
        {
            return context.Update(entity)
                .Entity;
        }

        public void SaveChanges()
        {
            context.SaveChanges();
        }
    }
```
Extend our repository

let's proceed to add on implementation for a particular entity that's based on our generic repository. We're going to start off with our product repository. This is going to inherit from our generic repository, and this particular repository will now work with our product entity. This means that we need to pass our shopping context into our product repository, which is then passed on to the base class, which is our generic repository. Now, we just out of the box got all the methods for adding updating, getting, finding, and saving the changes for a product. So one of the things that I want to implement in the concrete implementations of our generic repository is I want to override how we update the products because I wanna make sure that we can first find the particular product and then simply change the values that we have approved to change in the repository

```CSharp
public class ProductRepository : GenericRepository<Product>
    {
        public ProductRepository(ShoppingContext context) : base(context)
        {

        }
        public override Product Update(Product entity)
        {
            var product = context.Products
                .Single(p => p.ProductId == entity.ProductId);

            product.Price = entity.Price;
            product.Name = entity.Name;

            return base.Update(product);
        }
    }
```
Consuming a Repository

we have the interface that represents the way that we communicate with a repository. Our controller can use this in order free to know how to grab data without having to care about how the data is represented. If we're using in hibernate entity framework or storing this to a file on disk, the consumer no longer has to know anything about the underlying data structure. We then introduce the generic repository that allows us to generically work with our data context to reduce the amount of duplication of code inside our concrete implementations off our repositories. So now let's go ahead and go to our Web application and refactor this to make use of our repositories. We're going to proceed to go into our Product controller where we using DI to load the productRepository of type IRepository<Product>

```CSharp
public class ProductController : Controller
    {
        private readonly IRepository<Product> productRepository;
        public ProductController(IRepository<Product> productRepository)
        {
            this.productRepository = productRepository;
        }
        public IActionResult Index()
        {
            var orders = productRepository.Find(product => product.Price > 20);
            return View(orders);
        }
    }
```
so we need to register the IRepository<Product> in ther startup class as

```CSharp

public void ConfigureServices(IServiceCollection services)
        {
            services.Configure<CookiePolicyOptions>(options =>
            {
                // This lambda determines whether user consent for non-essential cookies is needed for a given request.
                options.CheckConsentNeeded = context => true;
                options.MinimumSameSitePolicy = SameSiteMode.None;
            });

            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

            services.AddTransient<ShoppingContext>();
            services.AddTransient<IRepository<Product>, ProductRepository>();
        })
```
the outcomes

    1- code for accessing data can now be shared
    2- data access in encapsulated
    3- the consumer (controller) no longer know about how data is accessed

Next Unit of Work Pattern
