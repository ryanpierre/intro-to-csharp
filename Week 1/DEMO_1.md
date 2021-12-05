## Things of note

- Compilation will prevent some of your normal TDD practices. 
- Making an interface first to satisfy tests can sometimes be helpful but in no way prescriptive. Like I can pass my tests with an interface and then I can use that interface and the errors it produces from inheritance as a guide to building the class.
- Initialize List class with values using new List<type> { "values", "go", "here" }
- Testing behaviour not implementation - we don't need to test the private methods, in fact we refactor them later to make sure the results we want are the same. I make the dummy statement that's always returned, then I refactor into private methods using my tests to know if I broke anything. I implicitly know my private methods work this way, while respecting the interface contract.

## Demo Structure 

- Demonstrate creating a new app for FizzBuzz and properly developing it in a TDD way
- Demonstrate tools in the IDE such as debugger, terminal output, test results, etc