# A Journey from AI to LLMs and MCP - 8 - 

## Free Resources  
- **[Free Apache Iceberg Course](https://hello.dremio.com/webcast-an-apache-iceberg-lakehouse-crash-course-reg.html?utm_source=ev_external_blog&utm_medium=influencer&utm_campaign=AItoLLMS&utm_content=alexmerced&utm_term=external_blog)**  
- **[Free Copy of “Apache Iceberg: The Definitive Guide”](https://hello.dremio.com/wp-apache-iceberg-the-definitive-guide-reg.html?utm_source=ev_external_blog&utm_medium=influencer&utm_campaign=AItoLLMS&utm_content=alexmerced&utm_term=external_blog)**  
- **[2025 Apache Iceberg Architecture Guide](https://medium.com/data-engineering-with-dremio/2025-guide-to-architecting-an-iceberg-lakehouse-9b19ed42c9de)**  
- **[How to Join the Iceberg Community](https://medium.alexmerced.blog/guide-to-finding-apache-iceberg-events-near-you-and-being-part-of-the-greater-iceberg-community-0c38ae785ddb)**  
- **[Iceberg Lakehouse Engineering Video Playlist](https://youtube.com/playlist?list=PLsLAVBjQJO0p0Yq1fLkoHvt2lEJj5pcYe&si=WTSnqjXZv6Glkc3y)**  
- **[Ultimate Apache Iceberg Resource Guide](https://medium.com/data-engineering-with-dremio/ultimate-directory-of-apache-iceberg-resources-e3e02efac62e)** 

# Resources in MCP — Serving Relevant Data Securely to LLMs

In the previous post, we explored the architecture of the **Model Context Protocol (MCP)**—a flexible, standardized way to connect LLMs to tools, data, and workflows. One of MCP’s most powerful capabilities is its ability to expose **resources** to language models in a structured, secure, and controllable way.

In this post, we’ll dive into:
- What MCP resources are
- How they’re discovered and accessed
- Text vs binary resources
- Dynamic templates and subscriptions
- Best practices for implementation and security

If you want to give LLMs real, relevant context from your systems—without compromising safety or control—**resources** are the foundation.

---

## 📦 What Are Resources in MCP?

**Resources** represent data that a model or client can read.

This might include:
- Local files (e.g. `file:///logs/server.log`)
- Database records (e.g. `postgres://db/customers`)
- Web content (e.g. `https://api.example.com/data`)
- Images or screenshots (e.g. `screen://localhost/monitor1`)
- Structured system data (e.g. logs, metrics, config files)

Each resource is identified by a **URI**, and can be **read**, **discovered**, and optionally **subscribed to** for updates.

---

## 🗂️ Resource Discovery

Clients can ask a server to list available resources using:
resources/list

csharp
Copy
Edit

The server responds with an array of structured metadata:

```json
{
  "resources": [
    {
      "uri": "file:///logs/app.log",
      "name": "Application Logs",
      "description": "Recent server logs",
      "mimeType": "text/plain"
    }
  ]
}
Clients (or users) can browse these like a menu, selecting what context to send to the model.

🔄 Resource Templates
In addition to static lists, servers can expose URI templates using RFC 6570 syntax:

json
Copy
Edit
{
  "uriTemplate": "file:///logs/{date}.log",
  "name": "Log by Date",
  "description": "Access logs by date (e.g., 2024-04-01)",
  "mimeType": "text/plain"
}
This allows dynamic access to parameterized content—great for APIs, time-based logs, or file hierarchies.

📖 Reading a Resource
To retrieve the content of a resource, clients use:

arduino
Copy
Edit
resources/read
With a payload like:

json
Copy
Edit
{
  "uri": "file:///logs/app.log"
}
The server responds with the content in one of two formats:

📝 Text Resource
json
Copy
Edit
{
  "contents": [
    {
      "uri": "file:///logs/app.log",
      "mimeType": "text/plain",
      "text": "Error: Timeout on request...\n"
    }
  ]
}
🖼️ Binary Resource (e.g. image, PDF)
json
Copy
Edit
{
  "contents": [
    {
      "uri": "screen://localhost/display1",
      "mimeType": "image/png",
      "blob": "iVBORw0KGgoAAAANSUhEUgAAA..."
    }
  ]
}
Clients can choose how and when to inject these into the model’s prompt, depending on MIME type and length.

🔔 Real-Time Updates
Resources aren’t static—they can change. MCP supports subscriptions to keep context fresh.

🔄 List Updates
If the list of resources changes, the server can notify the client with:

bash
Copy
Edit
notifications/resources/list_changed
Useful when new logs, files, or endpoints become available.

📡 Content Updates
Clients can subscribe to specific resource URIs:

bash
Copy
Edit
resources/subscribe
When the resource changes, the server sends:

bash
Copy
Edit
notifications/resources/updated
This is ideal for live logs, dashboards, or real-time documents.

🛡️ Security Best Practices
Exposing resources to models requires careful control. MCP includes flexible patterns for securing access:

✅ Best Practices for Server Developers
Validate all URIs: No open file reads!

Whitelist paths or endpoints for file access

Use descriptive names and MIME types to help clients filter content

Provide helpful descriptions for the LLM and user

Support URI templates for scalable access

Audit access and subscriptions

Avoid leaking secrets in content or metadata

🧱 Example: Safe Log Server
ts
Copy
Edit
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "file:///logs/app.log",
        name: "App Logs",
        mimeType: "text/plain"
      }
    ]
  };
});

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;

  if (!uri.startsWith("file:///logs/")) {
    throw new Error("Access denied");
  }

  const content = await readFile(uri); // Add sanitization here
  return {
    contents: [{
      uri,
      mimeType: "text/plain",
      text: content
    }]
  };
});
🧠 Why Resources Matter for AI Agents
LLMs are context-hungry. They reason better when they have:

Real-time logs

Source code

System metrics

API responses

By serving these as resources, MCP gives agents the data they need—on demand, with full user control, and without bloating prompt templates.

📌 Recap: Resources at a Glance
Feature	Description
URI-based identifiers	Unique path to each piece of content
Text & binary support	Suitable for logs, images, PDFs, etc.
Dynamic templates	Construct URIs on the fly
Real-time updates	Subscriptions for changing content
Secure access patterns	URI validation, MIME filtering, whitelisting
🔮 Coming Up Next: Tools in MCP — Giving LLMs the Power to Act
So far, we’ve shown how MCP feeds models with data. But what if we want the model to take action?

In the next post, we’ll explore tools in MCP:

How LLMs call functions safely

Tool schemas and invocation patterns

Real-world examples: shell commands, API calls, and more

With resources, LLMs become aware of their environment. With tools, they begin to act on it. The next stage of agent intelligence is just around the corner.

yaml
Copy
Edit

---

Let me know if you want **Blog 9: “Tools in MCP — Giving LLMs the Power to Act”** next, or if you'd like to include visual examples or diagrams for this post!





You said:
