
# JSON serializers for Ktos

The purpose of this document is to assess Kotlinx Serialization, GSON and Jackon as a JSON encoder to a Ktor service.

## Ktor Kotlinx Serialization (kotlinx.serialization)

### Pros
* Native Kotlin support: as it is designed by JetBrains, fully idiomatic Kotlin with support for Kotlin data classes, sealed classes, and inline classes.
* Lightweight & fast: lightweight compared to Jackson and GSON, as it's designed with Kotlin multiplatform in mind.
* Compile-time serialization: utilizes compile-time-generated serializers, which reduces runtime reflection overhead, making serialization and deserialization faster.
* Ktor native integration: native integration with Ktor.
* Supports multiple formats: JSON, Protobuf, CBOR, and others.
* Null safety: excellent Kotlin null safety handling.

### Cons
* Less mature: still evolving compared to Jackson. Some advanced features and customizations might be missing or less flexible.
* Limited ecosystem: fewer plugins, extensions, and third-party integrations compared to Jackson.
* Strict schemas: less flexible when dealing with dynamic or loosely typed JSON.
* Learning curve: some advanced customizations can be less intuitive compared to Jackson.

### Latency Impact
The absence of runtime reflection improves performance, making it suitable for low-latency services.

## GSON

### Pros
* Mature & stable: widely adopted, battle-tested, and supported by Google.
* Flexible: supports dynamic JSON and allows custom serializers.
* Easy to use: simple API.
* No external dependencies: pure Java library with no additional transitive dependancies.

### Cons
* Reflection-based: uses runtime reflection, which can introduce latency overhead.
* Slower performance: generally slower than Jackson and kotlinx.serialization in benchmarks.
* No Kotlin-specific optimizations: lacks Kotlin-specific features (like handling Kotlin-specific types) without extra adapters.
* Memory footprint: can be larger due to reflection and object allocations.
* Limited streaming: lacks efficient streaming support compared to Jackson.

### Latency Impact

Reflection overhead and slower parsing can be bottlenecks in high-throughput or low-latency environments.

## Jackson

### Pros
* Feature-rich: very mature, widely adopted, extensive ecosystem and community support.
* Highly configurable: supports annotations and custom serializers/deserializers.
* Streaming API: supports efficient streaming, which reduces memory and latency.
* Fast: one of the fastest JSON serializers/deserializers in Java ecosystem.

### Cons
* Reflection at runtime: uses runtime reflection, though caching and bytecode tricks reduce overhead.
* Larger library: bigger footprint compared to kotlinx.serialization.
* A more complex API: can be more challenging to configure properly.
* Initialization cost: some overhead on startup due to introspection and caching.

### Latency Impact

Faster than GSON, and with a streaming API, it can be optimized for low-latency scenarios. However, it still involves some runtime reflection overhead, unlike kotlinx.serialization.

## Recommendation for Low-Latency

Ktor Kotlinx Serialization is the best option.
Its compile-time serialization with no reflection overhead fits low-latency requirements well.
