# MyShop-RepositoryPattern
 Repository Pattern will encapsulate the logic to communicate with your data layer so that the consumer of your repository don't have to worry about if you're using entity 
 framework.If you're communicating with a file system, if you're using in hibernate or any other oh rms or ways to communicate with your database.

## This Reop is A quick Introduction To Repository Pattern.
 1. The repository pattern can be applied in any type of application. 
 2. It doesn't have to be a Web application. It doesn't have to be a .NET application 
 3. Applying the repository pattern means that we introduce a layer that encapsulate our data access code.
 4. This means that our repository will know how totern simply communicate with entity framework and hibernate, or our file on disk or another SQL Server or other types off database is. 
 


## First, let's see an App implementation without a Repository Pattern.

1. If you don't have asp.net core experience, Controller and its action method handles incoming browser requests, retrieves necessary model data, and returns appropriate responses.
2. Entity Framework is an Object/Relational Mapping (O/RM) framework. It is an enhancement to ADO.NET that gives developers an automated mechanism for accessing & storing the data in the database. 

```c#
    public class OrderController : Controller
    {
        //An instance of DbContext represents a session with the database which can be used to query and save instances of  entities to a database.
        private ShoppingContext context;

        public OrderController()
        {
            context = new ShoppingContext();
        }
        
        //Here the controller method communicates with the DataAccess layer directly to retrieve the data.
        public IActionResult Index()
        {
            var orders = context.Orders
                .Include(order => order.LineItems)
                .ThenInclude(lineItem => lineItem.Product)
                .Where(order => order.OrderDate > DateTime.UtcNow.AddDays(-1)).ToList();

            return View(orders);
        }
        // again we communicate directly wi the DataAccess layer.
        public IActionResult Create()
        {
            var products = context.Products.ToList();

            return View(products);
        }
        //again we communicate directly wi the DataAccess layer
        [HttpPost]
        public IActionResult Create(CreateOrderModel model)
        {
            if (!model.LineItems.Any()) return BadRequest("Please submit line items");

            if (string.IsNullOrWhiteSpace(model.Customer.Name)) return BadRequest("Customer needs a name");

            var customer = new Customer
            {
                Name = model.Customer.Name,
                ShippingAddress = model.Customer.ShippingAddress,
                City = model.Customer.City,
                PostalCode = model.Customer.PostalCode,
                Country = model.Customer.Country
            };

            var order = new Order
            {
                LineItems = model.LineItems
                    .Select(line => new LineItem { ProductId = line.ProductId, Quantity = line.Quantity })
                    .ToList(),

                Customer = customer
            };

            context.Orders.Add(order);

            context.SaveChanges();

            return Ok("Order Created");
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }
``` 
## 
<img src="https://user-images.githubusercontent.com/18700494/109553599-e4346e80-7adb-11eb-81ec-006b8d2982a2.png" />

## So, what is the problem with that approach?
1. The controller is tightly coupled with the data access layer.
2. The controller has to know about the O.R M or the way that we work with our database. 
3. it is difficult to write a test for the controller without side effects.
4. once we want to leverage the same data access somewhere else in the application, we'll have to duplicate our code, and that's not very good for maintain ability.

## So, let see how Repository Pattern will help us to solve these problems!
<img src="https://user-images.githubusercontent.com/18700494/109556132-0ed3f680-7adf-11eb-8922-e5a899e9943c.png" />

1.  Applying the repository pattern simply means that we introduce a layer that encapsulate our data access code.
2.  This means that our repository will know how to communicate with, for instance, entity framework and hibernate, or our file on disk or another SQL Server or other types off database is.
3.  This means that the controller can now leverage our repository to ask for data without having to worry about how that data is fetched from our data store. 
4.  It's easy for us to write a test without having any side effects.
5.  Introducing this repository also now means that we have a share a ble abstraction, resulting in less duplication of code. 
6.   A repository is simply an abstraction that encapsulate your data access , making your code testable, reusable as well as maintainable.  

## Implementation 
1. We apply Three- layers Architecture in this Project, the repository classes will be here [MyShop.Infrastructure].
2. Three-tier architecture means dividing the project into three layers User Interface Layer, Business Layer, and Data Layer where we separate logic, data, and user interface into three divisions. 
3. No Problem, if you don't know it, you can read about it later. We will only focus on Repository Pattern.
4. So we have three different projects. We have our web application, which contains our model views and controllers.
5. we have an infrastructure project which will contain the data access as well as services 
6. The third is our domain project. 

#### 1-Create IRepository Interface.
1. This interface will simply dictate the contract of a repository. 
2. we're going to set up a generic interface.
3. Our repository is going to allow us to add an entity it's going to allow us to update on entity, and it's going to allow us to retrieve a particular entity.
4. we're going to introduce a concrete implementation off this interface which are controller can use. 
```c#
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
#### 2-Create Generic Repository .
1. a lot of  different repositories have the same type of code for retrieving, getting, finding and listing all the items.
2. So how about we introduce a generic repository that introduces these base functionality that all of the different repositories can leverage?
3. And if we have particular implementations for other types of repositories, we can introduce that in a subclass.
4. We're going to make sure that we pass our shopping context into the constructor of our repository.
```c#
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
```
### 3-Add Sub Classes
1. The sub classes off generic repository can add custom behavior to each of the methods before data is passed to the database or return to the caller.
2. So we could override each of the methods inside our generic repository to introduce this custom behavior in our concrete implementations of our repositories.
3. In order for us to be able to override those methods, they all need to be marked as virtual.
4. This just means that we have the capability off overriding that particular method, but we don't have to. 
```c#
public class ProductRepository : GenericRepository<Product>
    {
        public ProductRepository(ShoppingContext context) : base(context)
        {

        }
        // here we override Update Method 
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
### 4-Use Repository in Controller
1.We have the interface that represents the way that we communicate with a repository.
2.Our controller can use this in order to fetch the  data without having to care about how the data is represented.
3. If we're using in hibernate entity framework or storing this to a file on disk, the Controller  no longer has to know anything about the underlying  structure.
4. We then introduce the generic repository that allows us to generically work with our data context to reduce the amount of duplication of code inside our concrete implementations off our repositories.
```c#
public class CustomerController : Controller
    {
       //we use Di to inject the object
        private readonly IRepository<Customer> repository;

        public CustomerController(IRepository<Customer> repository)
        {
            this.repository = repository;
        }

        public IActionResult Index(Guid? id)
        {
            if (id == null)
            {
                var customers = repository.All();

                return View(customers);
            }
            else
            {
                var customer = repository.Get(id.Value);

                return View(new[] { customer });
            }
        }
    }
    
    // in startup 
     public void ConfigureServices(IServiceCollection services)
        {
           //...      
            services.AddTransient<IRepository<Customer>, CustomerRepository>();
            services.AddTransient<IRepository<Order>, OrderRepository>();
            services.AddTransient<IRepository<Product>, ProductRepository>();
        }
```
