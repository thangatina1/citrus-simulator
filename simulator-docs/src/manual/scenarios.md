## Simulator scenarios

The simulator provides the response generating logic by defining one to many scenarios that get executed based on the incoming request. The different scenarios on the simulator 
describe different response messages and stand for individual simulation logic. Each scenario is capable of receiving and validating the incoming request message. Based on that the scenario 
is in charge of constructing a proper response message.
 
First of all the scenario gets a name under which mapping strategies can identify the scenario. This name is very important when it comes to mapping incoming requests to scenarios. Besides that
the scenario is a normal Java class that implements following interface *SimulatorScenario*

```java
package com.consol.citrus.simulator.scenario;

/**
 * @author Christoph Deppisch
 */
public interface SimulatorScenario {

    ScenarioEndpoint scenario();
}
```

The simulator scenario has to provide message receive and send operations for the simulator endpoint. Fortunately there are default implementations that you can inherit from:
 
* **SimulatorRestScenario** 
* **SimulatorWebServiceScenario** 
* **SimulatorJmsScenario** 
* **SimulatorEndpointScenario** 

Each of these base scenario classes provide comfortable access to the endpoint so we can just add receive and send logic for generating the response message.

```java
@Scenario("Hello")
public class HelloScenario extends SimulatorRestScenario {

    @Override
    protected void configure() {
        scenario()
            .receive()
            .payload("<Hello xmlns=\"http://citrusframework.org/schemas/hello\">" +
                        "Say Hello!" +
                     "</Hello>");

        scenario()
            .send()
            .payload("<HelloResponse xmlns=\"http://citrusframework.org/schemas/hello\">" +
                        "Hi there!" +
                     "</HelloResponse>");
    }
}
``` 

The scenario above is annotated with **@Scenario** for defining a scenario name. There is one single **configure** method to be implemented.
We can use Citrus Java DSL methods in the method body in order to receive the incoming request and send back a proper response message. Of course we can use the full Citrus power here
in order to construct different message payloads such as XML, JSON, PLAINTEXT and so on.

So we could also extract dynamic values from the request in order to reuse those in our response message:
 
```java
@Scenario("Hello")
public class HelloScenario extends SimulatorRestScenario {

    @Override
    protected void configure() {
        scenario()
            .receive()
            .payload("<Hello xmlns=\"http://citrusframework.org/schemas/hello\">" +
                        "<user>@ignore@</user>" +
                     "</Hello>")
            .extractFromPayload("/Hello/user", "userName");

        scenario()
            .send()
            .payload("<HelloResponse xmlns=\"http://citrusframework.org/schemas/hello\">" +
                        "<text>Hi there ${userName}!</text>" +
                     "</HelloResponse>");
    }
}
``` 

In the receive operation the user name value is extracted to a test variable ***${userName}*. In the response we are able to use this variable in order to greet the user by name. This way
we can use the Citrus test power for generating dynamic response messages. Of course this mechanism works for XML, Json and Plaintext payloads.

Now you are ready to write different scenarios that generate different response messages for the calling client.