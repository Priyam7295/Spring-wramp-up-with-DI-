---
### Table of Contents:-
1. Annotations
2. Beans and its lifecycle and IOC
3. Dependency Injection

---
# 1. Annotations 

### 1. @Controller, @RestController (@Controller + @ResponseBody)
The annotations @Controller and @RestController help Spring identify which classes contain request-handling logic, when the request comes to the application .
```
@Controller 
public class UserController {

    @GetMapping("/user/{id}")
    // here ResponseBody is not used, it means the return will not be String, but will be a view
    //It’s typically used for traditional web applications where the response is a rendered view (
    public String getUser(@PathVariable Long id, Model model) {
        // Simulate fetching a user from a service
        User user = new User(id, "John Doe");
        
        // Add user to the model for the view
        model.addAttribute("user", user);
        
        // Return the view name (Thymeleaf template)
        return "user";
    }

    @GetMapping("/user/data/{id}")
    @ResponseBody
    public User getUserData(@PathVariable Long id) {
        // Return raw data (JSON) with @ResponseBody
        return new User(id, "Jane Doe");
    }
```

```
// UserRestController.java
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@RestController
public class UserRestController {

    @GetMapping("/api/user/{id}")
    public User getUser(@PathVariable Long id) {
        // Simulate fetching a user from a service
        return new User(id, "John Doe");
    }

    @PostMapping("/api/user")
    public User createUser(@RequestBody User user) {
        // Simulate saving a user
        return user; // Return the created user
    }
}
```

### 2. @RequestMapping

```
@Controller
public class SampleController{
    @RequestMapping( path = "/api/user" , method = RequestMethod.GET )
    @ResponseBody
    public String sampleMethod(){
        return "Fetching something and returning"
    }
}
```

### 3. @GetMapping ( short version for @RequestMapping( path = "/api/user" , method = RequestMethod.GET ) ) , @PostMapping
```
@Controller
public class SampleController{
    @GetMapping(path="/api/user")
    @ResponseBody
    public String sampleMethod(){
        return "Fetching something and returning"
    }
}
```
#### NOTE:- we can also write the Mapping at the controller layer, if the initials for all the methos inside are same. ex- 

```
@RestController
@RequestMapping(path="/api/user/")
public class SampleController{
    @GetMapping(path="/history")
    public History getHistor(){
    }

   @PostMapping("/createUser")
    public User createUser(@RequestBody User user) {
        // Simulate saving a user
        return user; // Return the created user
    }
}
```

### 4. @RequestParam

Request coming from -
**https://Ui.com/api/fetchUser?firstName=priyam&lastName=raj&age=23**

```
@RestController
@RequestMapping(value = "/api")
public class SampleController {

    @GetMapping(path = "/fetchUser")
    public String getUserDetails(
        @RequestParam(name = "firstName") String firstName,
        @RequestParam(name = "lastName", required = false) String lastName,
        @RequestParam(name = "age") int age) {
        
        return "fetching and returning user details based on first name = " + firstName + 
               ", lastName = " + lastName + " and age is = " + age;
    }
}

```

- Spring automatically type cast primitive data types while doing the above, for ex - from the url, when we store the age inside the controller,  it gets automatically casted to int data type.
- But if we want to do cast, then we need to do it using custom type converter.
- ex- Whatever my first name comes ,before we assign the value to the method param here ```@RequestParam(name = "firstName") String firstName ``` we want to do some custom logic, we need to do it using property editor

```

class FirstNamePropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        // Convert the input to uppercase and remove any extra spaces
        String processedName = text.trim().toUpperCase();
        setValue(processedName);
    }
}

@RestController
@RequestMapping(value = "/api")
public class SampleController {

    // It runs first, and checks if any of the params require any pre-processing
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        // Register custom editor for firstName parameter
        // Parameters: (return type, parameter name, editor class)
        binder.registerCustomEditor(String.class, "firstName", new FirstNamePropertyEditor());
    }

    @GetMapping(path = "/fetchUser")
    public String getUserDetails(
        @RequestParam(name = "firstName") String firstName,
        @RequestParam(name = "lastName", required = false) String lastName,
        @RequestParam(name = "age") int age) {
        
        return "fetching and returning user details based on first name = " + firstName + 
               ", lastName = " + lastName + " and age is = " + age;
        }
    }
}
```
### 5. @PathVariable
Request coming from -
**https://Ui.com/api/fetchUser/12131

```
@RestController
@RequestMapping(value = "/api")
public class SampleController {
      @GetMapping("/fetchUser/{userId}")
      public String returnUser(@PathVariable(value = "firstName") String firstName ){
          return "fetching user " + fetchName;
      } 
}
```

### 6. @RequestBody

```
@RestController
@RequestMapping(value = "/api/user")
public class SampleController{
      @PostMapping("/saveUser")
      public String getUserDetails(@RequestBody User user){
          // 1. create user
          // 2. return "user created" + user; 
      }
}

```

### 7. @ResponseEntity 
- Till now, we know controller and RestController
- If in @Controller, we donot use @ResponseBody at the method level, by default spring will try to render it as a viewName(like.jsp , so if method return "Hello", it will try to render Hello.jsp)
- ResponseEntity = It represents the entire HTTP **response**
- @ResponseBody talks about just body
- @ResponseEntity talks about body ,statusCode,Header.

```
@RestController
@RequestMapping("/api")
public class UserController {

    // 1. Basic ResponseEntity with just body and status
    @GetMapping("/user")
    public ResponseEntity<String> getUser() {
        return new ResponseEntity<>("User details", HttpStatus.OK);
        // or return ResponseEntity.ok("User details");
    }

    // 2. ResponseEntity with custom headers
    @GetMapping("/user-with-header")
    public ResponseEntity<String> getUserWithHeader() {
        HttpHeaders headers = new HttpHeaders();
        headers.add("Custom-Header", "Custom-Value"); // and returning to the uI
        
        return new ResponseEntity<>(
            "User with custom header", 
            headers, 
            HttpStatus.OK
        );
    }

    // 3. Different HTTP Status codes
    @PostMapping("/create")
    public ResponseEntity<String> createUser() {
        return new ResponseEntity<>("User Created", HttpStatus.CREATED); // 201
    }

    // 4. With Object response
    @GetMapping("/user-object")
    public ResponseEntity<User> getUserObject() {
        User user = new User("John", "Doe", 25);
        return ResponseEntity.ok(user);
    }

    // 6. Error scenario
    @GetMapping("/error-demo")
    public ResponseEntity<String> getError() {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body("Something went wrong");
    }
}

```

- In case of @RestController, internally spring will create ResponseEntity, and so we can just write ```public String methodName```, instead of ``` public ResponseEntity<String> methodName ```
- In case of @Controller, we need to explicity add the ResponseEntity, since spring is not sure whether the .jsp page will be returned, or the data directly to the client by using @ResponeDody
---

# 2. Bean and its LifeCycle | Inversion Of Control(IOC)
- Bean is a java Object, that is handled by Spring Container(IOC)
- IOC container contains all the objects created and also managing it throughout it's lifecycle.

Two ways of creating a bean
- @Component Annotation
- @Bean Annotation

### 1. @Component Annotation
- " Convention over configuration " approach
- Means spring boot will try to auto configure based on conventions, reducing the need for explicit configurations
- @Components, @Service etc, all tells spring to create a bean and internally use it.

```
@Component
public class User{
    String userName;
    String email;

    // getter and setter
}
```

- So here, spring will use ```new User()``` [i.e, using default constructor] to create an instance(object) of this class, and store it .

But what now-
```
@Component
public class User{
    String userName;
    String email;

    // Here we created our own constructor, so no default constructor is there
    public User(string username, String email){
        this.userName = username;
        this.email = email;
    }
    // getter and setter
}
```
- if we use @Component, then our app will fail, spring will not be able to create an instance of the object .
- So @Beans come into the picture, where we provide configurations.

### 2. @Bean Annotations

2.1 User class
```
public class User{
    String userName;
    String email;

    // Here we created our own constructor, so no default constructor is there
    public User(string username, String email){
        this.userName = username;
        this.email = email;
    }
    // getter and setter
}
```
2.2 Creating configuration - @Configuration
- This annotation tells spring that, here there will be certain bean methods, for that you need to create bean also.
```
@Configuration
public class AppConfig{
    @Bean
    public User createUserBean(){
        return new User("Priyam","priyaammm@amazon.com");
    }
}
```
- When are Beans created?
- NOTE- @Bean , by default has Singleton Scope
- 1. Eagerness - When we start application. Ex- Beans with singleton scope are eagerly initialized 
  2. Lazy - Some Beans are created lazilly, meaning when they actaully need . Ex- Beans with scopes like Prototype are lazily initialized 
  3. Even though the Bean with singleton Scope (with @Lazy Annotation), then the Bean gets created Lazily
 
### 3. Lifecycle of a Bean

[Application Start] -> [IOC Container started] -> [Construct Bean (by IOC scanning for @Component and @Bean(inside @Configuration))] -> [Inject Dependency into Constructed Bean] -> [@PostContruct] -> [Use the Bean] -> [@PreDestroy] -> [Bean Destroy(after we close the application)] , Finally IOC closed, all beans destroyed . 

#### Exmaple on DI
If Bean is not found, spring will create it and then inject it

```
@Component
// Be default singleton scope, hence eagerly initialized
public class User{

    @Autowired
    Order order; // Dependency Injection happens here, spring inject the Order Bean here, by creating that

    public User(){
        System.println.out("User initiated");
    }
}
```

```
@Lazy // scope is lazy, will get initialzed, only when needed
@Component
public class Order{
    public Order(){
        System.println.out("Order initiated");
    }
}
```
---
# 3. Dependency Injection
What is Dependency Injection
- We can make a class independent of it's dependencies
-  @Autowired - first look for bean of required type, if not create it and inject it

Ways of Dependency Injection:-
1. Field Injection
2. Setter Injection
3. Constructor Injection


** We only have to use Constructor Injection. why ?**

#### 1. Field Injection
   - Dependency is set into the field of class directly.
```
@Component
// Be default singleton scope, hence eagerly initialized
public class User{

    @Autowired
    Order order; // Dependency Injection happens here, spring inject the Order Bean here, by creating that

    public User(){
        System.println.out("User initiated");
    }
}
```

```
@Lazy // scope is lazy, will get initialzed, only when needed
@Component
public class Order{
    public Order(){
        System.println.out("Order initiated");
    }
}
```
Advantage
- Very simple and easy to use

Disadvantages
- Can't be used with Immutable field.
- Like can't use ```public final Order order```;
- **Chances of null Pointer Exception**
  Scenario 1 - Some part of code creating object like this
  ```
  @Autowired
    private User user;
  ```
  Scenario 2 - Some part of code creating object like this
  
  ```
  User newObj = new User();
  // so here order is null, as dependency injection is not happening here
  
  ```
- Unit Testing Mock issue - Since the injection happens automatically by spring, there is not way to mock it. Ex- cannot mock ```order``` here.

#### 2. Setter Injection
- Dependency is set into the feild using the setter method
- 
```
@Service
public class OrderService {
    private PaymentService paymentService;         // Required injection
    private InventoryService inventoryService;     // Required injection
    private NotificationService notificationService; // Optional injection
    private boolean isInitialized = false;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @Autowired
    public void setInventoryService(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }

    @Autowired(required = false)
    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}

```

es2- in controller layer example
```
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private OrderService orderService; // Cannot be final, since we are setting it

    @Autowired
    public void setOrderService(OrderService orderService) { // Setter injection
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        // Using OrderService
        Order order = orderService.processOrder(request);
        return ResponseEntity.ok(order);
    }
}

public class orderService{
    orderService(){
        // constructor called
    }
}
```

Advantages:-
- Dependency can be changed any time after the object creation by using the setter method
- Can directly pass mock object in the setter, no need to use mockito

Disadvantages:-
- Field can't be marked as final.
- Since once object is created, it will see @Autowired, and then spring will do injection here by seeing the Bean in the IOC
- Hard to understand and maintain

#### 3. Constructor Injection

- At the time of creating the object, all the dependencies are resolved (injected)
```
@Service  
public class OrderService {
    private final PaymentService paymentService;  // final field
    private final InventoryService inventoryService;

    public OrderService(PaymentService paymentService) {  // Constructor injection
        this.paymentService = paymentService;
    }

    @Autowired
    public OrderService(InventoryService inventoryService){
        this.inventoryService = inventoryService;
    }
}

public class OrderService{
    
}
```

ex2- Multiple Injection
```
@Service  
public class OrderService {
    private final PaymentService paymentService;  
    private final InventoryService inventoryService;

    @Autowired
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {  
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}

```

Advantage:-
1. Can use final
2. During object creation, every dependency is resolved and injected and ready to work
3. Fails fast - If any dependency is missed, then it will fail during compilation time itself, rather than in runtime in case of other 2 methods.
4. Unit Testing becomes easy -  We can pass mock objects in the contructor


#### Common Issues dealing with dependencies
1. Circular Dependency
2. Unsatisfied Dependency

#### 1. Circular Dependency
- As same suggests, if class A has dependency B, and B has A.
```
// Circular Dependency Problem ❌
@Component
public class A {
    private final B b;

    @Autowired
    public A(B b) {  // Needs B
        this.b = b;
    }
}

@Component
public class B {
    private final A a;

    @Autowired
    public B(A a) {  // Needs A
        this.a = a;
    }
}
// This will fail because each needs the other to be created first!
```

- Try to refactor code
- Use @Lazy on @Autowired component [Hacky]
- Using @PostConstruct [Hacky]
  
#### 2. Unsatisfied Dependency
- User class has one dependency order .
- Order is an interface, how the User class knows, which implementation of order to inject
- So applicaation will fail and output - unsatisfied Dependency
```
@Component
public class User{
    @Autowired
    User user; // using field injection for demo, can be any

    public User(){
        // created
    }

}

public interface Order{
    
}

@Component
@Primary
public class OnlineOrder implements Order{

}

@Component
public class OfflineOrder implements Order{

}

```

Solutions:-
1. @Primary Annotations to give priority to any one implementation of the Order interface.
2. @Qualifers

Ex- Using qualifier
```
// 1. First define interface
public interface Order {
    void process();
}

// 2. Implementations with qualifiers
@Component
@Qualifier("online")  // or can create custom qualifier
public class OnlineOrder implements Order {
    @Override
    public void process() {
        System.out.println("Processing online order");
    }
}

@Component
@Qualifier("offline")
public class OfflineOrder implements Order {
    @Override
    public void process() {
        System.out.println("Processing offline order");
    }
}

// 3. Using Qualifier in User class
@Component
public class User {
    private final Order order;

    @Autowired
    public User(@Qualifier("online") Order order) {  // Specify which implementation
        this.order = order;
    }
}
```
