### What Is Unit Testing?

A **unit test** is a piece of code that tests a small, isolated piece (a *unit*) of your application, typically a method or function, to ensure it behaves as expected.

---

### Benefits of Unit Testing

- Catches bugs early in development
- Makes code easier to refactor
- Serves as documentation for how the code works
- Encourages writing loosely coupled, testable code

---

### Unit Testing Frameworks in .NET

1. **xUnit** – modern, widely used
2. **NUnit** – similar to JUnit, very popular
3. **MSTest** – built into Visual Studio

Let’s go with **xUnit** (preferred by the .NET community).

---

### Getting Started with xUnit

**1. Install the xUnit NuGet package:**

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Moq
```

---

### Sample Class to Test

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Multiply(int a, int b) => a * b;
}
```

---

### Writing a Unit Test with xUnit

```csharp
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_ShouldReturnCorrectSum()
    {
        // Arrange
        var calc = new Calculator();

        // Act
        var result = calc.Add(2, 3);

        // Assert
        Assert.Equal(5, result);
    }

    [Fact]
    public void Multiply_ShouldReturnCorrectProduct()
    {
        var calc = new Calculator();
        var result = calc.Multiply(4, 5);
        Assert.Equal(20, result);
    }
}
```

---

### Running the Tests

```bash
dotnet test
```

---

### What is the Arrange-Act-Assert pattern?

The **Arrange-Act-Assert** pattern is a simple and effective way to structure unit tests. It divides the test into three distinct parts:

- **Arrange**: You prepare the necessary objects, variables, or data needed for the test.
- **Act**: You perform the operation or method call that you want to test.
- **Assert**: You verify that the result is what you expect.

### Example

Here’s an example of a unit test that uses this pattern with a simple `Calculator` class:

#### Calculator Class

```csharp
public class Calculator
{
    public int Add(int x, int y)
    {
        return x + y;
    }
}
```

#### Test Case Using Arrange-Act-Assert

```csharp
using Xunit;

public class CalculatorTests
{
    [Fact]
    public void Add_ShouldReturnSumOfTwoNumbers()
    {
        // Arrange
        var calculator = new Calculator();
        int a = 5;
        int b = 7;

        // Act
        int result = calculator.Add(a, b);

        // Assert
        Assert.Equal(12, result);
    }
}
```

### Explanation

- **Arrange**: We set up the `calculator` object and initialized the values `a = 5` and `b = 7`.
- **Act**: We invoked the `Add` method to compute the sum.
- **Assert**: We checked that the result equals the expected value `12`.

> **Note:** Unit tests are designed to verify that a function or method produces the expected output for a given input. The focus is not on how the function works internally but on its observable behavior, which allows you to confidently test and refactor code without breaking existing functionality.



### String Test Example in Unit Testing & Fluent Assertion

Let’s assume you have a class that performs string manipulations, such as trimming whitespaces and checking if a string contains certain characters.

#### Example String Method

```csharp
public class StringHelper
{
    public string TrimWhitespace(string input)
    {
        return input.Trim();
    }

    public bool ContainsWord(string input, string word)
    {
        return input.Contains(word);
    }
}
```

In this example:
- `TrimWhitespace`: Removes leading and trailing whitespace from a string.
- `ContainsWord`: Checks if a word exists within a string.

### Unit Test Without FluentAssertions

Here’s how we could write basic unit tests for these methods using **xUnit** and the Arrange-Act-Assert pattern:

```csharp
using Xunit;

public class StringHelperTests
{
    [Fact]
    public void TrimWhitespace_ShouldRemoveLeadingAndTrailingSpaces()
    {
        // Arrange
        var helper = new StringHelper();
        string input = "  Hello World  ";

        // Act
        string result = helper.TrimWhitespace(input);

        // Assert
        Assert.Equal("Hello World", result);
    }

    [Fact]
    public void ContainsWord_ShouldReturnTrueWhenWordExists()
    {
        // Arrange
        var helper = new StringHelper();
        string input = "Hello, world!";
        string word = "world";

        // Act
        bool result = helper.ContainsWord(input, word);

        // Assert
        Assert.True(result);
    }
}
```

These tests verify that:
1. The whitespace is trimmed correctly from the input string.
2. The `ContainsWord` method correctly checks if the word exists in the input string.

---

### Using FluentAssertions

**FluentAssertions** is a library that provides more expressive and readable assertions. It allows you to write assertions in a more human-readable format.

#### Adding FluentAssertions

First, install **FluentAssertions** via NuGet:

```bash
dotnet add package FluentAssertions
```

Now, let’s rewrite the same tests using **FluentAssertions**.

#### Refactored Unit Tests with FluentAssertions

```csharp
using FluentAssertions;
using Xunit;

public class StringHelperTests
{
    [Fact]
    public void TrimWhitespace_ShouldRemoveLeadingAndTrailingSpaces()
    {
        // Arrange
        var helper = new StringHelper();
        string input = "  Hello World  ";

        // Act
        string result = helper.TrimWhitespace(input);

        // Assert
        result.Should().Be("Hello World");
    }

    [Fact]
    public void ContainsWord_ShouldReturnTrueWhenWordExists()
    {
        // Arrange
        var helper = new StringHelper();
        string input = "Hello, world!";
        string word = "world";

        // Act
        bool result = helper.ContainsWord(input, word);

        // Assert
        result.Should().BeTrue();
    }
}
```

### Why Use FluentAssertions?

1. **More Readable**: The syntax is easier to read and understand, making tests look closer to natural language. For example, `result.Should().Be("Hello World")` is more expressive than `Assert.Equal("Hello World", result)`.

2. **Better Error Messages**: When a test fails, FluentAssertions provides more descriptive error messages, making it easier to identify what went wrong.

3. **Advanced Features**: FluentAssertions supports a wide range of assertions like checking collections, exceptions, and more. For example, it supports `BeNull`, `BeEmpty`, `HaveCount`, `Contain`, etc.

#### Example of Advanced Assertions

```csharp
public void ShouldContainWordAndHaveCorrectLength()
{
    // Arrange
    var helper = new StringHelper();
    string input = "Hello, world!";

    // Act
    bool result = helper.ContainsWord(input, "world");

    // Assert
    result.Should().BeTrue();
    input.Should().Contain("world");
    input.Length.Should().BeGreaterThan(10);
}
```

This example demonstrates multiple assertions for the same string, and all of them use FluentAssertions.

---

### Summary

- **Unit Test**: You test a string manipulation method by verifying if it behaves as expected for different inputs.
- **FluentAssertions**: Makes the assertions more readable and provides better error messages. It’s a helpful tool for writing clean, expressive tests.
