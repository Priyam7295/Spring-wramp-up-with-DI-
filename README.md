
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

1. @Component Annotation
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
