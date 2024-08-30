# Static Keyword

# Static keyword in different Contexts

Static keyword is a bit different in different contexts, and it was a bit confusing for me in the
beginning, so let's see what `static` keyword means in contexts in `C++`.

## Global Members (Internal Linking)

When `static` keyword is used to declare a global variable it means that that variable is `internally
linked`. 

Let's see a small overview of internal vs external linking to understand, it's pretty much like
private and public access control in `classes` but for global members.
### Internal Linking

- In `internal linking` the variable/function is not visible to other files (translation units) so that
particular function is only for the file it is declared in, it cannot be accessed in other files.
- This means that, you can declare a variable with the same name in another file and the compiler
will treat those two variables with same name as different variables and will not throw ambiguity
error.
- So, generally internal linking allows you to have variables and functions with same name and 
signature in different files without causing conflicts and ambiguity error.

!!! info "Ambiguity error"
    Ambiguity error occurs when there are multiple declaration of same variable/functions and the
    compiler doesn't know which one to use, so it throws error.

**Consider this example, read the comments for better understanding**,

#### Foo.cpp File
```C++
// This variable can only be accessed inside Foo.cpp
static int InternalVariable = 10;

// This function can only be accessed inside Foo.cpp
static void InternalFunction()
{

}

// This variable can be accessed in any file
int ExternalVariable = 20;

// This function can be accessed in any file
void ExternalFunction() 
{
   
}
```

#### Bar.cpp File
```C++
// Externs allows to use the variable in Foo.cpp
extern int ExternalVariable; 

// This variable is different from the one in Foo.cpp, because of static keyword
// This variable can only be accessed inside Bar.cpp
static int InternalVariable = 20;

// This function is different from the one in Foo.cpp, because of static keyword
// This function can only be accessed inside Bar.cpp
static void InternalFunction()
{

}

/*
 * Throws error this is because, ExternalFunction is already declared in Foo.cpp and it does not
 * have a static keyword, which means it's externally linked and not internally linked, will explain
 * a bit more in detail below
 */
void ExternalFunction()
{

}
```

#### Inference from the Example

- Here, you can see that ExternalFunction throws error, this is because it does not have the static
keyword which means that it's externally linked. 

- Externally linked means, the ExternalFunction will be visible to all translation units and
when the linker cross-checks, it will see that ExternalFunction is defined twice, once in
`Foo.cpp` and another time in `Bar.cpp`.

- Since it is not internal, it means that both definitions of the ExternalFunction points to
the same function, and the compiler will not know which one to pick, so it throws ambiguity error


#### In Short

Internal keyword is like `private` access specifier for a file, and restricts the global
member to be defined or used only within the file.

## Class Members

- Static Keyword for class members can come in handy in many cases. 
- When a variable/function is declared with the keyword static, then that variable/function can
be accessed without needing to have an object for that class.
- To access the class member without any object, just use the scope resolution operator, this beautiful
thing - `::`
- Static functions can only access the static members of the class, it cannot access the non-static
members of the class even if it is public.
- Note that, to access the static class members, the member must be public.

**Consider the following example and read the comments for better understanding**

```C++
class Circle
{

public:
    
    // Can be access directly outside the class
    static float Pi;
    
    // Cannot be accessed outside the class, it is a class variable, only for the class object
    float ClassSpecificRadius = 10.f;
   
    // Cannot be accessed outside the class, this function can only be called with an object
    float CalculateDiameter()
    {    
        return 2.f * ClassSpecificRadius;
    }
    
    // Can be access directly outside the class
    // Does not throw error since we are only accessing the static member of the class which is Pi
    static float CalculateCircumferenceForRadius(const float Radius)
    {
        return 2.f * Pi * Radius;
    } 
    
    // Throws error since a static function is accessing a class variable, which is not allowed
    static float CalculateCircumference()
    {
        // Here, accessing Pi is fine, since it is a static variable
        // But accessing ClassSpecificRadius is not allowed since it is a class only variable
        return 2.f * Pi * ClassSpecificRadius;
    } 
};

int main()
{
    /* [Accessing static members with Scope Resolution operator] */
    
    // For non-integral type like float, static members must only be initialized outside
    Circle::Pi = 3.14f; // Valid access no error
    
    // Valid, does not throw error
    Circle::CalculateCircumferenceForRadius(20.f); 
    
    // Throws error, since we are trying to access a class variable without an object
    Circle::ClassSpecificRadius;
    
    // Throws error, we are trying to access a class function without an object
    Circle::CalculateDiameter();
    
    // Throws error, since the function uses a class member in it's definition
    Circle::CalculateCircumference();
    
     /* [Accessing class members with an object of the class] */
    
    // Creating an object for the class
    Circle CircleInstance = Circle();
    
    // Vaid access, since we are accessing the members only with an object
    CircleInstance.ClassSpecificRadius; 
    CircleInstance.CalculateDiameter();
}
```

### Things to Note

- Static members of a class has a scope of the program, so it is valid throughout the lifetime
of the program.
- Static members must only be defined once, other it results in ambiguity error, since they
are treated like global variables, linker does not like this at all.
- Static members only have one instance in the memory, unlike class members which are created
for each instance of the class created. This is a fundamental characteristic of static. 
members—they are associated with the class itself rather than any specific object.


## References :

1. `Game Engine Architecture` book, by Jason Gregory.
2. `Unreal Engine Source Code`, yeah I understood most concepts by using and reading Unreal Engine's
source code