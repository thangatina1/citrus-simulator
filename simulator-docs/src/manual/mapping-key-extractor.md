## Mapping key extractor

The mapping key extractor implementation decides how to map incoming request message to simulator scenarios. Each incoming request
triggers a predefined scenario that generates the response message for the calling client. The simulator identifies the scenario based 
on a mapping key that is extracted from the incoming request.

There are multiple ways to identify the simulator scenario from incoming request messages:

* Message-Type: Each request message type (XML root QName) results in a separate simulator scenario
* REST request mappings: Identifies the scenario based on Http method and resource path on server
* SOAP Action: Each SOAP action value defines a simulator scenario
* Message Header: Any SOAP or Http message header value specifies a new simulator scenario
* XPath payload: An XPath expression is evaluated on the message payload to identify the scenario
* Json payload: An JsonPath expression is evaluated on the message payload to identify the scenario

Once the simulator scenario is identified with the respective mapping key the scenario get loaded and executed. All scenarios perform Citrus test logic in order
to provide a proper response messages as a result. This way the simulator is able to perform complex response generating logic with dynamic values and so on. 

The mentioned mapping key extraction strategies are implemented in these classes:

* **AnnotationRequestMappingKeyExtractor** Evaluates REST request mappings
* **SoapActionMappingKeyExtractor** Evaluates the SOAP action header
* **HeaderMappingKeyExtractor** Evaluates any message header
* **XPathPayloadMappingKeyExtractor** Evaluates a XPath expression on the message payload
* **JsonPayloadMappingKeyExtractor** Evaluates a JsonPath expression on the message payload

Of course you can also implement a custom mapping key extractor. Just implement the interface methods of that API and add the implementation to the simulator 
configuration as described later on in this document.

### Default behavior

The default mapping key logic extracts the message type of incoming requests. This is done by evaluating a Xpath expression on the request payload that uses the root element of the message as the
mapping key. Each message type gets its own simulator scenario.

Let's demonstrate that in a simple example. We know three different message types named **successMessage**, **warningMessage** and **errorMessage**. We create a simulator scenario for each of these message types with
respective naming. Given the following incoming requests the simulator will pick the matching scenario for execution. 

```
<successMessage>
    <text>This is a success message</text>
</successMessage>

<warningMessage>
    <text>This is a warning message</text>
</warningMessage>

<errorMessage>
    <text>This is a error message</text>
</errorMessage>
```

The simulator evaluates the root element name and maps the requests to the matching scenario. Each scenario implements different response generating logic so the simulator is able to respond to a **successMessage** in a different
way than for **errorMessage** types.

### Configuration

You can change the mapping key behavior by overwriting te default mapping key extractor in your simulator.

```java
public class SimulatorAdapter extends SimulatorRestAdapter {
    @Override
    public MappingKeyExtractor mappingKeyExtractor() {
        HeaderMappingKeyExtractor mappingKeyExtractor = new HeaderMappingKeyExtractor();
        mappingKeyExtractor.setHeaderName("X-simulator-scenario");
        return mappingKeyExtractor;
    } 
}
```

With the configuration above we use the *HeaderMappingKeyExtractor* implementation so the header name **X-simulator-scenario** gets evaluated for each incoming request message. 
Depending on that header value the matching scenario is executed as a result. As you can see the mapping key extractor is just a bean in the Spring application context. There is a default implementation but you can overwrite
this behavior very easy in the simulator adapter configuration.