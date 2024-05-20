Here's the utility class for JSON schema validation using the `networknt/json-schema-validator` library. This class will load the schema, validate JSON data against it, and print out detailed error messages if the validation fails.

### JSONSchemaValidatorUtility.java

```java
package com.example.jsonvalidator;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.schema.JsonSchema;
import com.networknt.schema.JsonSchemaFactory;
import com.networknt.schema.SpecVersion;
import com.networknt.schema.ValidationMessage;

import java.io.InputStream;
import java.util.Set;

public class JSONSchemaValidatorUtility {

    private static final ObjectMapper mapper = new ObjectMapper();

    public static JsonSchema getSchema(String schemaPath) throws Exception {
        InputStream schemaStream = JSONSchemaValidatorUtility.class.getResourceAsStream(schemaPath);
        JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
        return factory.getSchema(schemaStream);
    }

    public static JsonNode getJsonNode(String jsonPath) throws Exception {
        InputStream jsonStream = JSONSchemaValidatorUtility.class.getResourceAsStream(jsonPath);
        return mapper.readTree(jsonStream);
    }

    public static Set<ValidationMessage> validateJson(JsonSchema schema, JsonNode jsonNode) {
        return schema.validate(jsonNode);
    }

    public static void printValidationMessages(Set<ValidationMessage> validationMessages) {
        if (validationMessages.isEmpty()) {
            System.out.println("Validation passed: No errors found.");
        } else {
            System.out.println("Validation failed with the following errors:");
            validationMessages.forEach(error -> System.out.println("Validation error: " + error.getMessage()));
        }
    }

    public static void main(String[] args) throws Exception {
        JsonSchema schema = getSchema("/schema.json");

        JsonNode validAcceptedJson = getJsonNode("/validAcceptedDecision.json");
        Set<ValidationMessage> errors = validateJson(schema, validAcceptedJson);
        printValidationMessages(errors);

        JsonNode validRejectedJson = getJsonNode("/validRejectedDecision.json");
        errors = validateJson(schema, validRejectedJson);
        printValidationMessages(errors);

        JsonNode invalidAcceptedJson = getJsonNode("/invalidAcceptedDecision.json");
        errors = validateJson(schema, invalidAcceptedJson);
        printValidationMessages(errors);

        JsonNode invalidRejectedJson = getJsonNode("/invalidRejectedDecision.json");
        errors = validateJson(schema, invalidRejectedJson);
        printValidationMessages(errors);
    }
}
```

### Explanation

1. **Package Declaration**: The utility class is placed in the `com.example.jsonvalidator` package. Adjust the package name as needed for your project structure.
2. **ObjectMapper Instance**: A static instance of `ObjectMapper` is used to read JSON data.
3. **getSchema Method**: This method loads the JSON schema from a file path.
4. **getJsonNode Method**: This method reads JSON data from a file path and converts it into a `JsonNode`.
5. **validateJson Method**: This method validates the JSON data against the provided schema and returns the validation messages.
6. **printValidationMessages Method**: This method prints the validation messages. It indicates whether validation passed or failed and lists the errors if there are any.
7. **main Method**: The main method demonstrates how to use the utility class to validate different JSON files against the schema and print the validation messages.

### Running the Utility

1. **Directory Structure**:
   - Ensure your project follows a standard Maven directory structure:
     ```
     src
     └── main
         ├── java
         │   └── com
         │       └── example
         │           └── jsonvalidator
         │               └── JSONSchemaValidatorUtility.java
         └── resources
             ├── schema.json
             ├── validAcceptedDecision.json
             ├── validRejectedDecision.json
             ├── invalidAcceptedDecision.json
             └── invalidRejectedDecision.json
     ```

2. **Running the Main Method**:
   - You can run the `main` method of `JSONSchemaValidatorUtility` from your IDE or using the `java` command in the terminal after building the project. This will validate the JSON files against the schema and print the validation results.

This utility class, combined with the test cases provided earlier, offers a comprehensive approach to validating JSON data against a schema with detailed custom error messages.
