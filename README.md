package com.example.jsonvalidator;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.schema.JsonSchema;
import com.networknt.schema.JsonSchemaFactory;
import com.networknt.schema.SpecVersion;
import com.networknt.schema.ValidationMessage;

import java.util.Set;
import java.util.stream.Collectors;

public class JSONSchemaValidatorUtility {

    private static final ObjectMapper mapper = new ObjectMapper();

    public static JsonSchema getSchemaFromString(String schemaString) throws Exception {
        JsonNode schemaNode = mapper.readTree(schemaString);
        JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V7);
        return factory.getSchema(schemaNode);
    }

    public static JsonNode getJsonNodeFromString(String jsonString) throws Exception {
        return mapper.readTree(jsonString);
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
        String schemaString = "{\n" +
                "  \"$schema\": \"http://json-schema.org/draft-07/schema#\",\n" +
                "  \"type\": \"array\",\n" +
                "  \"items\": {\n" +
                "    \"type\": \"object\",\n" +
                "    \"properties\": {\n" +
                "      \"Decision\": {\n" +
                "        \"type\": \"string\",\n" +
                "        \"enum\": [\"Accepted\", \"Rejected\"]\n" +
                "      },\n" +
                "      \"RejectionCode\": {\n" +
                "        \"type\": \"string\",\n" +
                "        \"nullable\": true\n" +
                "      },\n" +
                "      \"AdditionalComments\": {\n" +
                "        \"type\": \"string\",\n" +
                "        \"nullable\": true\n" +
                "      }\n" +
                "    },\n" +
                "    \"required\": [\"Decision\"],\n" +
                "    \"allOf\": [\n" +
                "      {\n" +
                "        \"if\": {\n" +
                "          \"properties\": { \"Decision\": { \"const\": \"Rejected\" } }\n" +
                "        },\n" +
                "        \"then\": {\n" +
                "          \"required\": [\"RejectionCode\", \"AdditionalComments\"]\n" +
                "        },\n" +
                "        \"else\": {\n" +
                "          \"properties\": {\n" +
                "            \"RejectionCode\": { \"const\": null },\n" +
                "            \"AdditionalComments\": { \"const\": null }\n" +
                "          }\n" +
                "        }\n" +
                "      }\n" +
                "    ]\n" +
                "  }\n" +
                "}";

        String jsonString = "[\n" +
                "  {\n" +
                "    \"Decision\": \"Accepted\",\n" +
                "    \"RejectionCode\": null,\n" +
                "    \"AdditionalComments\": null\n" +
                "  },\n" +
                "  {\n" +
                "    \"Decision\": \"Rejected\"\n" +
                "  }\n" +
                "]";

        JsonSchema schema = getSchemaFromString(schemaString);
        JsonNode jsonNode = getJsonNodeFromString(jsonString);
        Set<ValidationMessage> errors = validateJson(schema, jsonNode);
        printValidationMessages(errors);
    }
}

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <jackson.version>2.13.4</jackson.version>
        <networknt.json.schema.validator.version>1.0.73</networknt.json.schema.validator.version>
        <junit.version>5.8.2</junit.version>
    </properties>

    <dependencies>
        <!-- Jackson Databind for JSON processing -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>

        <!-- Networknt JSON Schema Validator -->
        <dependency>
            <groupId>com.networknt</groupId>
            <artifactId>json-schema-validator</artifactId>
            <version>${networknt.json.schema.validator.version}</version>
        </dependency>

        <!-- JUnit for testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
            </plugin>
        </plugins>
    </build>
</project>

