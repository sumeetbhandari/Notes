To use the `networknt/json-schema-validator` library for validating JSON against a JSON Schema with conditional requirements in Java, you can follow these steps:

### Step 1: Add the required dependencies

First, add the `networknt/json-schema-validator` dependency to your project. If you are using Maven, add the following dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>1.0.76</version> <!-- Check for the latest version -->
</dependency>
```

If you are using Gradle, add the following line to your `build.gradle` file:

```gradle
implementation 'com.networknt:json-schema-validator:1.0.76'
```

### Step 2: Create the JSON Schema

Create a JSON Schema file `schema.json` with the required conditional logic.

**schema.json:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": ["Accepted", "Rejected"]
    },
    "rejectionCode": {
      "type": "string"
    },
    "additionalComments": {
      "type": "string"
    }
  },
  "required": ["decision"],
  "if": {
    "properties": {
      "decision": { "const": "Rejected" }
    }
  },
  "then": {
    "required": ["rejectionCode", "additionalComments"]
  }
}
```

### Step 3: Create JSON Data Examples

Create JSON data files to be validated against the schema.

**dataAccepted.json:**
```json
{
  "decision": "Accepted"
}
```

**dataRejected.json:**
```json
{
  "decision": "Rejected",
  "rejectionCode": "RC123",
  "additionalComments": "Some comments"
}
```

**dataInvalidRejected.json:**
```json
{
  "decision": "Rejected"
}
```

### Step 4: Write the Java Code for Validation

Create a Java class to perform the validation using the `networknt/json-schema-validator` library.

**JsonSchemaValidator.java:**

```java
import com.networknt.schema.JsonSchema;
import com.networknt.schema.JsonSchemaFactory;
import com.networknt.schema.ValidationMessage;
import com.networknt.schema.SpecVersion;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.InputStream;
import java.util.Set;

public class JsonSchemaValidator {

    public static void main(String[] args) {
        validateJson("/dataAccepted.json");
        validateJson("/dataRejected.json");
        validateJson("/dataInvalidRejected.json");
    }

    private static void validateJson(String jsonFilePath) {
        try {
            // Load the JSON schema
            InputStream schemaStream = JsonSchemaValidator.class.getResourceAsStream("/schema.json");
            JsonSchemaFactory schemaFactory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
            JsonSchema schema = schemaFactory.getSchema(schemaStream);

            // Load the JSON document
            InputStream jsonStream = JsonSchemaValidator.class.getResourceAsStream(jsonFilePath);
            ObjectMapper mapper = new ObjectMapper();
            JsonNode jsonNode = mapper.readTree(jsonStream);

            // Validate the JSON document against the schema
            Set<ValidationMessage> validationMessages = schema.validate(jsonNode);
            if (validationMessages.isEmpty()) {
                System.out.println(jsonFilePath + ": JSON is valid against the schema.");
            } else {
                System.out.println(jsonFilePath + ": JSON is not valid. Errors: " + validationMessages);
            }
        } catch (Exception e) {
            System.out.println(jsonFilePath + ": JSON is not valid. Error: " + e.getMessage());
        }
    }
}
```

### Explanation

1. **Schema Definition:**
   - The `decision` field must be a string with values "Accepted" or "Rejected".
   - If `decision` is "Rejected", then `rejectionCode` and `additionalComments` must be present and be strings.
   - The `if`, `then`, and `else` clauses are used to conditionally enforce the presence of `rejectionCode` and `additionalComments` based on the value of `decision`.

2. **Java Validation Code:**
   - The Java code reads the schema from `schema.json` and the JSON documents from separate files.
   - The `validateJson` method is used to validate different JSON files against the schema.
   - It prints a success message if the JSON is valid, and an error message if it is not valid.

### Running the Code

1. Create three JSON files `dataAccepted.json`, `dataRejected.json`, and `dataInvalidRejected.json` in the `resources` directory of your project.

**dataAccepted.json:**
```json
{
  "decision": "Accepted"
}
```

**dataRejected.json:**
```json
{
  "decision": "Rejected",
  "rejectionCode": "RC123",
  "additionalComments": "Some comments"
}
```

**dataInvalidRejected.json:**
```json
{
  "decision": "Rejected"
}
```

2. Place the `schema.json` file in the `resources` directory of your project.

3. Compile and run the `JsonSchemaValidator` class. The program will validate the JSON documents against the schema and print the results.

By following these steps, you can validate JSON documents with conditional requirements using Java and the `networknt/json-schema-validator` library.
