---
mapped_pages:
  - https://www.elastic.co/guide/en/ecs-logging/java/current/_structured_logging_with_log4j2.html
---

# Structured logging with log4j2 [_structured_logging_with_log4j2]

By leveraging log4j2’s `MapMessage` or even by implementing your own `MultiformatMessage` with JSON support, you can add additional fields to the resulting JSON.

Example:

```java
logger.info(new StringMapMessage()
    .with("message", "Hello World!")
    .with("foo", "bar"));
```

If Jackson is on the classpath, you can also use an `ObjectMessage` to add a custom object the resulting JSON.

```java
logger.info(new ObjectMessage(myObject));
```

The `myObject` variable refers to a custom object which can be serialized by a Jackson `ObjectMapper`.

Using either will merge the object at the top-level (not nested under `message`) of the log event if it is a JSON object. If it’s a string, number boolean, or array, it will be converted into a string and added as the `message` property. This conversion avoids mapping conflicts as `message` is typed as a string in the Elasticsearch mapping.


## Tips [_tips]

We recommend using existing [ECS fields](ecs://reference/ecs-field-reference.md).

If there is no appropriate ECS field, consider prefixing your fields with `labels.`, as in `labels.foo`, for simple key/value pairs. For nested structures, consider prefixing with `custom.`. This approach protects against conflicts in case ECS later adds the same fields but with a different mapping.


## Gotchas [_gotchas]

A common pitfall is how dots in field names are handled in Elasticsearch and how they affect the mapping. In recent Elasticsearch versions, the following JSON structures would result in the same index mapping:

```json
{
  "foo.bar": "baz"
}
```

```json
{
  "foo": {
    "bar": "baz"
  }
}
```

The property `foo` would be mapped to the [Object datatype](elasticsearch://reference/elasticsearch/mapping-reference/object.md).

This means that you can’t index a document where `foo` would be a different datatype, as in shown in the following example:

```json
{
  "foo": "bar"
}
```

In that example, `foo` is a string. Trying to index that document results in an error because the data type of `foo` can’t be object and string at the same time.

