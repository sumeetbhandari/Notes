Here's how you can write Java test cases using the `networknt/json-schema-validator` library for JSON schema validation, including assertions on validation messages. This example includes the Maven setup and test cases.

### Maven Setup

First, make sure your Maven `pom.xml` includes the necessary dependencies:

```xml
<properties>
    <json.schema.validator.version>1.0.72</json.schema.validator.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>json-schema-validator</artifactId>
            <version>${json.schema.validator.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>com.networknt</groupId>
        <artifactId>json-schema-validator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### JSON Schema

Here's the JSON schema with custom error messages:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "Decision": {
      "type": "string",
      "enum": ["Accepted", "Rejected"],
      "errorMessage": "Decision must be either 'Accepted' or 'Rejected'."
    },
    "RejectionCode": {
      "type": "string",
      "errorMessage": "RejectionCode is required when Decision is 'Rejected'."
    },
    "AdditionalComments": {
      "type": "string",
      "errorMessage": "AdditionalComments are required when Decision is 'Rejected'."
    }
  },
  "required": ["Decision"],
  "if": {
    "properties": { "Decision": { "const": "Rejected" } }
  },
  "then": {
    "required": ["RejectionCode", "AdditionalComments"]
  },
  "else": {
    "properties": {
      "RejectionCode": { "type": "null" },
      "AdditionalComments": { "type": "null" }
    }
  }
}
```

### Java Test Cases

Here's a set of Java test cases using JUnit 5:

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

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertFalse;

public class JsonSchemaValidatorTest {

    private JsonSchema getSchema(String schemaPath) throws Exception {
        InputStream schemaStream = getClass().getResourceAsStream(schemaPath);
        JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
        return factory.getSchema(schemaStream);
    }

    private JsonNode getJsonNode(String jsonPath) throws Exception {
        InputStream jsonStream = getClass().getResourceAsStream(jsonPath);
        ObjectMapper mapper = new ObjectMapper();
        return mapper.readTree(jsonStream);
    }

    @Test
    public void testValidAcceptedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/validAcceptedDecision.json");

        Set<ValidationMessage> errors = schema.validate(jsonNode);
        assertTrue(errors.isEmpty(), "Expected no validation errors, but found: " + errors);
    }

    @Test
    public void testValidRejectedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/validRejectedDecision.json");

        Set<ValidationMessage> errors = schema.validate(jsonNode);
        assertTrue(errors.isEmpty(), "Expected no validation errors, but found: " + errors);
    }

    @Test
    public void testInvalidAcceptedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/invalidAcceptedDecision.json");

        Set<ValidationMessage> errors = schema.validate(jsonNode);
        assertFalse(errors.isEmpty(), "Expected validation errors, but found none");
        errors.forEach(error -> System.out.println("Validation error: " + error.getMessage()));
    }

    @Test
    public void testInvalidRejectedDecision() throws Exception {
        JsonSchema schema = getSchema("/schema.json");
        JsonNode jsonNode = getJsonNode("/invalidRejectedDecision.json");

        Set<ValidationMessage> errors = schema.validate(jsonNode);
        assertFalse(errors.isEmpty(), "Expected validation errors, but found none");
        errors.forEach(error -> System.out.println("Validation error: " + error.getMessage()));
    }
}
```

### Example JSON Files

Place these JSON files in the `resources` directory:

**validAcceptedDecision.json**:
```json
{
  "Decision": "Accepted",
  "RejectionCode": null,
  "AdditionalComments": null
}
```

**validRejectedDecision.json**:
```json
{
  "Decision": "Rejected",
  "RejectionCode": "RC001",
  "AdditionalComments": "Some comments"
}
```

**invalidAcceptedDecision.json**:
```json
{
  "Decision": "Accepted",
  "RejectionCode": "RC001",
  "AdditionalComments": "Some comments"
}
```

**invalidRejectedDecision.json**:
```json
{
  "Decision": "Rejected"
}
```

### Explanation

1. **Setup and Dependencies**: The `pom.xml` file includes the necessary dependencies for JSON schema validation and JUnit 5 for testing.
2. **Schema with Custom Error Messages**: The JSON schema is defined with custom error messages for different properties and conditions.
3. **Java Test Cases**: The test cases load the schema and JSON files, perform validation, and assert the results, printing validation messages for errors.

These tests ensure that the schema validation behaves as expected, and the custom error messages provide clear feedback on validation failures.
