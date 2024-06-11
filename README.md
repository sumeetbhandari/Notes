If the schema is an array, you'll need to adjust the validation and the tests to handle an array of JSON objects. Here's how you can modify the utility class and the test cases to accommodate this scenario:

### Example Schema (Array of Objects)

Let's assume the schema defines an array of objects where each object follows the previously mentioned structure.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "Decision": {
        "type": "string",
        "enum": ["Accepted", "Rejected"]
      },
      "RejectionCode": {
        "type": "string"
      },
      "AdditionalComments": {
        "type": "string"
      }
    },
    "required": ["Decision"],
    "allOf": [
      {
        "if": {
          "properties": { "Decision": { "const": "Rejected" } }
        },
        "then": {
          "required": ["RejectionCode", "AdditionalComments"]
        },
        "else": {
          "properties": {
            "RejectionCode": { "const": null },
            "AdditionalComments": { "const": null }
          }
        }
      }
    ]
  }
}
```

### Updated Utility Class

#### JSONSchemaValidatorUtility.java

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
import java.util.stream.Collectors;

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
            Set<String> formattedMessages = validationMessages.stream()
                .map(error -> error.getMessage().replace("$.","").trim())
                .collect(Collectors.toSet());
            formattedMessages.forEach(error -> System.out.println("Validation error: " + error));
        }
    }

    public static void main(String[] args) throws Exception {
        JsonSchema schema = getSchema("/schema.json");

        JsonNode validAcceptedJson = getJsonNode("/validAcceptedDecisionArray.json");
        Set<ValidationMessage> errors = validateJson(schema, validAcceptedJson);
        printValidationMessages(errors);

        JsonNode validRejectedJson = getJsonNode("/validRejectedDecisionArray.json");
        errors = validateJson(schema, validRejectedJson);
        printValidationMessages(errors);

        JsonNode invalidAcceptedJson = getJsonNode("/invalidAcceptedDecisionArray.json");
        errors = validateJson(schema, invalidAcceptedJson);
        printValidationMessages(errors);

        JsonNode invalidRejectedJson = getJsonNode("/invalidRejectedDecisionArray.json");
        errors = validateJson(schema, invalidRejectedJson);
        printValidationMessages(errors);
    }
}
```

### Example JSON Files

**validAcceptedDecisionArray.json**:
```json
[
  {
    "Decision": "Accepted",
    "RejectionCode": null,
    "AdditionalComments": null
  }
]
```

**validRejectedDecisionArray.json**:
```json
[
  {
    "Decision": "Rejected",
    "RejectionCode": "RC001",
    "AdditionalComments": "Some comments"
  }
]
```

**invalidAcceptedDecisionArray.json**:
```json
[
  {
    "Decision": "Accepted",
    "RejectionCode": "RC001",
    "AdditionalComments": "Some comments"
  }
]
```

**invalidRejectedDecisionArray.json**:
```json
[
  {
    "Decision": "Rejected"
  }
]
```

### Updated JUnit Test Cases

#### JsonSchemaValidatorTest.java

```java
import com.networknt.schema.JsonSchema;
import com.networknt.schema.JsonSchemaFactory;
import com.networknt.schema.SpecVersion;
import com.networknt.schema.ValidationMessage;
import org.junit.jupiter.api.Test;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.InputStream;
import java.util.Set;
import java.util.stream.Collectors;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertFalse;

public class JsonSchemaValidatorTest {

    private static JsonSchema getSchema(String schemaPath) throws Exception {
        InputStream schemaStream = JsonSchemaValidatorTest.class.getResourceAsStream(schemaPath);
        JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
        return factory.getSchema(schemaStream);
    }

    private static JsonNode getJsonNode(String jsonPath) throws Exception {
        InputStream jsonStream = JsonSchemaValidatorTest.class.getResourceAsStream(jsonPath);
        ObjectMapper mapper = new ObjectMapper();
        return mapper.readTree(jsonStream);
    }

    private static Set<String> validateJson(JsonSchema schema, JsonNode jsonNode) {
        Set<ValidationMessage> errors = schema.validate(jsonNode);
        return errors.stream()
                .map(error -> error.getMessage().replace("$.","").trim())
                .collect(Collectors.toSet());
    }

    @Test
    public void testValidAcceptedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/validAcceptedDecisionArray.json");

        Set<String> errors = validateJson(schema, jsonNode);
        assertTrue(errors.isEmpty(), "Expected no validation errors, but found: " + errors);
    }

    @Test
    public void testValidRejectedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/validRejectedDecisionArray.json");

        Set<String> errors = validateJson(schema, jsonNode);
        assertTrue(errors.isEmpty(), "Expected no validation errors, but found: " + errors);
    }

    @Test
    public void testInvalidAcceptedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/invalidAcceptedDecisionArray.json");

        Set<String> errors = validateJson(schema, jsonNode);
        assertFalse(errors.isEmpty(), "Expected validation errors, but found none");

        // Assert specific error messages
        assertTrue(errors.contains("RejectionCode: must be null"), "Expected error message about RejectionCode being null, but found: " + errors);
        assertTrue(errors.contains("AdditionalComments: must be null"), "Expected error message about AdditionalComments being null, but found: " + errors);
    }

    @Test
    public void testInvalidRejectedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/invalidRejectedDecisionArray.json");

        Set<String> errors = validateJson(schema, jsonNode);
        assertFalse(errors.isEmpty(), "Expected validation errors, but found none");

        // Assert specific error messages
        assertTrue(errors.contains("RejectionCode: required key [RejectionCode] not found"), "Expected error message about RejectionCode being missing, but found: " + errors);
        assertTrue(errors.contains("AdditionalComments: required key [AdditionalComments] not found"), "Expected error message about AdditionalComments being missing, but found: " + errors);
    }
}
```

### Explanation

1. **Utility Class**: Updated to validate an array of JSON objects.
2. **JSON Files**: Adjusted to contain arrays of JSON objects.
3. **JUnit Test Cases**: 
   - Ensure the validation handles arrays of objects.
   - Specific assertions for error messages are updated accordingly.

This setup will allow you to validate JSON arrays against the schema and handle validation messages appropriately.
