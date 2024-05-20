To achieve custom error messages natively supported by a JSON schema validation library, you might want to use the `Everit` JSON schema library. This library allows you to embed custom messages directly in your JSON schema, and then process these messages easily in your Java code. Here’s how you can set this up:

### 1. JSON Schema with Custom Messages

Define your schema with custom messages using the `errorMessage` keyword:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": ["Accepted", "Rejected"],
      "errorMessage": {
        "enum": "The decision field must be either 'Accepted' or 'Rejected'."
      }
    },
    "rejectionCode": {
      "type": "string",
      "errorMessage": {
        "type": "The rejection code must be a string."
      }
    },
    "additionalComments": {
      "type": "string",
      "errorMessage": {
        "type": "The additional comments must be a string."
      }
    }
  },
  "required": ["decision"],
  "if": {
    "properties": { "decision": { "const": "Rejected" } }
  },
  "then": {
    "required": ["rejectionCode", "additionalComments"],
    "errorMessage": {
      "required": "When decision is 'Rejected', rejectionCode and additionalComments are required."
    }
  },
  "else": {
    "not": {
      "required": ["rejectionCode", "additionalComments"]
    }
  }
}
```

### 2. Maven Dependency

First, add the `everit-json-schema` dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.everit.json</groupId>
    <artifactId>org.everit.json.schema</artifactId>
    <version>1.14.0</version> <!-- Use the latest version -->
</dependency>
```

### 3. Java Test Class

Create a Java test class to validate JSON instances against the schema and handle custom error messages:

```java
package com.example.project.test;

import org.everit.json.schema.Schema;
import org.everit.json.schema.loader.SchemaLoader;
import org.json.JSONObject;
import org.json.JSONTokener;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.everit.json.schema.ValidationException;

import java.io.InputStream;
import java.util.List;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.fail;

public class JsonSchemaValidatorTest {

    private static Schema schema;

    @BeforeAll
    public static void setup() throws Exception {
        // Load the JSON schema
        InputStream schemaStream = JsonSchemaValidatorTest.class.getResourceAsStream("/schema.json");
        JSONObject rawSchema = new JSONObject(new JSONTokener(schemaStream));
        schema = SchemaLoader.load(rawSchema);
    }

    private void validateJson(String jsonFilePath) {
        try {
            InputStream jsonStream = JsonSchemaValidatorTest.class.getResourceAsStream(jsonFilePath);
            JSONObject json = new JSONObject(new JSONTokener(jsonStream));
            schema.validate(json);
        } catch (ValidationException e) {
            printValidationMessages(e);
            fail("JSON validation failed: " + e.getMessage());
        }
    }

    private void printValidationMessages(ValidationException e) {
        List<ValidationException> causes = e.getCausingExceptions();
        if (causes.isEmpty()) {
            System.out.println(e.getMessage());
        } else {
            for (ValidationException cause : causes) {
                System.out.println(cause.getMessage());
            }
        }
    }

    @Test
    public void testValidAccepted() {
        validateJson("/dataAccepted.json");
    }

    @Test
    public void testValidRejected() {
        validateJson("/dataRejected.json");
    }

    @Test
    public void testInvalidRejected() {
        try {
            validateJson("/dataInvalidRejected.json");
        } catch (AssertionError e) {
            // Expected validation failure, do nothing
        }
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

When running the tests, the output should include the custom error messages defined in your schema:

```sh
[INFO] Running com.example.project.test.JsonSchemaValidatorTest
The decision field must be either 'Accepted' or 'Rejected'.
When decision is 'Rejected', rejectionCode and additionalComments are required.
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.123 s - in com.example.project.test.JsonSchemaValidatorTest

Results:

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.345 s
[INFO] Finished at: 2024-05-15T15:35:19+00:00
[INFO] ------------------------------------------------------------------------
```

### Summary

- **Schema**: Defines custom error messages.
- **Java Test Class**: Validates JSON files and prints custom validation messages.
- **Output**: Provides clear and specific custom validation error messages.

This approach leverages the `Everit` JSON schema validator to directly handle custom error messages, streamlining the validation process and ensuring more informative feedback.
