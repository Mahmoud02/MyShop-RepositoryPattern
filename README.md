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
