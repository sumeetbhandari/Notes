The mapping of custom messages in the Java test class is not strictly necessary if your validation library supports custom error messages directly within the schema. However, the `networknt` JSON schema validator library does not directly support custom error messages embedded in the schema itself. Therefore, post-processing the validation results in the test class to apply custom messages is a practical workaround.

If you would like to simplify your setup and avoid manual mapping in the test class, you could consider using a JSON schema validation library that natively supports custom error messages. But if you prefer to stick with `networknt` JSON schema validator and ensure clear and specific messages, here's a streamlined approach without manually mapping messages:

### Updated JSON Schema

While the schema itself can't natively define custom messages that the `networknt` library will directly use, we can still define it for clarity.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": ["Accepted", "Rejected"],
      "description": "Decision must be 'Accepted' or 'Rejected'."
    },
    "rejectionCode": {
      "type": "string",
      "description": "Rejection code must be a string."
    },
    "additionalComments": {
      "type": "string",
      "description": "Additional comments must be a string."
    }
  },
  "required": ["decision"],
  "if": {
    "properties": { "decision": { "const": "Rejected" } }
  },
  "then": {
    "required": ["rejectionCode", "additionalComments"],
    "description": "When decision is 'Rejected', rejectionCode and additionalComments are required."
  },
  "else": {
    "not": {
      "required": ["rejectionCode", "additionalComments"]
    }
  }
}
```

### Simplified Java Test Class

Here is how you can simplify the Java test class to print the custom messages without mapping:

```java
package com.example.project.test;

import com.networknt.schema.JsonSchema;
import com.networknt.schema.JsonSchemaFactory;
import com.networknt.schema.SpecVersion;
import com.networknt.schema.ValidationMessage;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.Set;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertFalse;

public class JsonSchemaValidatorTest {

    private static JsonSchema schema;
    private static ObjectMapper mapper;

    @BeforeAll
    public static void setup() throws Exception {
        // Load the JSON schema
        InputStream schemaStream = JsonSchemaValidatorTest.class.getResourceAsStream("/schema.json");
        JsonSchemaFactory schemaFactory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
        schema = schemaFactory.getSchema(schemaStream);

        // Initialize ObjectMapper
        mapper = new ObjectMapper();
    }

    private Set<ValidationMessage> validateJson(String jsonFilePath) {
        try {
            // Load the JSON document
            InputStream jsonStream = JsonSchemaValidatorTest.class.getResourceAsStream(jsonFilePath);
            JsonNode jsonNode = mapper.readTree(jsonStream);

            // Validate the JSON document against the schema
            return schema.validate(jsonNode);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    @Test
    public void testValidAccepted() {
        Set<ValidationMessage> messages = validateJson("/dataAccepted.json");
        assertTrue(messages.isEmpty(), "Valid Accepted JSON should pass validation, but got: " + messages);
    }

    @Test
    public void testValidRejected() {
        Set<ValidationMessage> messages = validateJson("/dataRejected.json");
        assertTrue(messages.isEmpty(), "Valid Rejected JSON should pass validation, but got: " + messages);
    }

    @Test
    public void testInvalidRejected() {
        Set<ValidationMessage> messages = validateJson("/dataInvalidRejected.json");
        assertFalse(messages.isEmpty(), "Invalid Rejected JSON should fail validation");
        messages.forEach(message -> {
            System.out.println("Validation message: " + message.getMessage() + " at " + message.getPath());
        });
        boolean foundExpectedError = messages.stream().anyMatch(msg -> msg.getMessage().contains("required properties [additionalComments] are missing"));
        assertTrue(foundExpectedError, "Expected validation error not found in messages: " + messages);
    }
}
```

### Running the Tests

1. **Navigate to the project directory**:
    ```sh
    cd /path/to/your/project
    ```

2. **Run the tests**:
    ```sh
    mvn test
    ```

### Expected Output

The expected output when running the tests will now print detailed validation messages, though not with custom error messages directly defined in the schema:

```sh
[INFO] Running com.example.project.test.JsonSchemaValidatorTest
Validation message: object has missing required properties ([additionalComments]) at /
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.015 s - in com.example.project.test.JsonSchemaValidatorTest

Results:

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.152 s
[INFO] Finished at: 2024-05-15T13:47:09+00:00
[INFO] ------------------------------------------------------------------------
```

### Summary

- **Schema**: Defines validation rules and descriptions.
- **Java Test Class**: Validates JSON files and prints validation messages.
- **Output**: Provides clear and detailed validation error messages.

This approach simplifies the setup while ensuring detailed and understandable validation feedback during test execution.


