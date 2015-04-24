# Tests PowerMatcher

To ensure a stable system extensive tests have been written for the PowerMatcher

## Unit Tests

The following elements are extensively unit tested with JUnit.

* All Core elements
* All DataObjects and 
* Monitoring 
* Peak Shaving
* Runtime


## Integration Tests

* Remote Communication (Websockets)
* Adding/Removing Agents
* SessionManager
* Concurrency
* Oscillation

## Contributing Code & Unit tests

Make sure when implementing new features that the existing test cases don't fail. If they fail for a good reason mention so in the pull request.
Make sure you have added sufficient unit testing for your contribution; in some case possible a new integration test.

