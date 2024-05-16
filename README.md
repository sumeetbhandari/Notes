Let's update the JSON Schema and the Java program to meet the new requirements:

1. The `decision` field must be a string and can only be "Accepted" or "Rejected".
2. If `decision` is "Rejected", then both `rejectionCode` and `additionalComments` fields must be present and be strings.
3. If `decision` is "Accepted", then `rejectionCode` and `additionalComments` should not be required and can be absent.

### JSON Schema (schema.json)

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

### JSON Data Examples

**Valid JSON when decision is "Accepted":**
```json
{
  "decision": "Accepted"
}
```

**Valid JSON when decision is "Rejected":**
```json
{
  "decision": "Rejected",
  "rejectionCode": "RC123",
  "additionalComments": "Some comments"
}
```

**Invalid JSON when decision is "Rejected" but missing required fields:**
```json
{
  "decision": "Rejected"
}
```

### Java Code for Validation (JsonSchemaValidator.java)

```java
import org.everit.json.schema.Schema;
import org.everit.json.schema.loader.SchemaLoader;
import org.json.JSONObject;
import org.json.JSONTokener;

import java.io.InputStream;

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
            JSONObject rawSchema = new JSONObject(new JSONTokener(schemaStream));
            Schema schema = SchemaLoader.load(rawSchema);

            // Load the JSON document
            InputStream jsonStream = JsonSchemaValidator.class.getResourceAsStream(jsonFilePath);
            JSONObject jsonObject = new JSONObject(new JSONTokener(jsonStream));

            // Validate the JSON document against the schema
            schema.validate(jsonObject);
            System.out.println(jsonFilePath + ": JSON is valid against the schema.");
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

By following these steps, you can validate JSON documents with conditional requirements using Java and the `everit-org/json-schema` library.
