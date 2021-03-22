# EQL

A noted based on [The Maximal Graph](https://youtu.be/IS3i3DTUnAI) video.

## Motivation

In a typical SSR, an url like `customers/2` is sent to the server. The server parses the input, fetches the data and render the HTML. There are two things involved: `which` and `what`. Which is the reference to one or many data. What is the data to be retrieved and may have three forms: stored data, computed data and related entities. For a data API, the client may request different `what`. The REST API complects `which` and `what` thus is not a good fit. A client should be able to specify which and what recursively. The query syntax should be descriptive and independent of implementations. GraphQL is an example that has a custom language and typed schema.

GraphQL doesn't fit Clojure because Clojure is property-based, not type-based.
