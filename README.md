# Ampersand Samples

## Overview

[Ampersand](https://www.withampersand.com/) is a developer platform for building and maintaining user-facing integrations. This repository contains sample code and schema documentation for using the Ampersand platform.

## Ampersand Manifest Schema Documentation

This repository now includes comprehensive documentation for the Ampersand manifest schema (`amp.yaml`), which is the key configuration file that defines your integrations.

### What is an Ampersand Manifest?

A manifest file (`amp.yaml`) defines how your application integrates with various SaaS platforms through the Ampersand integration platform. It specifies:

- What data to read from external systems
- What data to write to external systems
- How to subscribe to events from external systems
- How to proxy API calls
- Authentication and scheduling configurations
- Object and field mapping across different providers

### Schema Documentation

The [SCHEMA.md](./SCHEMA.md) file provides a comprehensive reference for the Ampersand manifest schema, including:

- Detailed field descriptions
- Format requirements
- Common patterns and best practices
- Provider-specific considerations
- Examples explained

### Schema Versioning

The manifest schema follows semantic versioning:

- **Current version**: 1.0.0

See [VERSIONING.md](./VERSIONING.md) for more details about the versioning approach.

### JSON Schema Definition

For validation and tooling purposes, a formal JSON Schema definition is available:

- [ampersand-schema.json](./schema/ampersand-schema.json)

## Quick Start

Here's how to create a minimal Ampersand manifest:

1. **Create an `amp.yaml` file** with the following content:

```yaml
specVersion: 1.0.0
integrations:
  - name: my-first-integration
    provider: salesforce
    read:
      objects:
        - objectName: Contact
          destination: defaultWebhook
          schedule: "*/30 * * * *"
          requiredFields:
            - fieldName: FirstName
            - fieldName: LastName
            - fieldName: Email
```

2. **Deploy your integration** following the instructions in the [Ampersand documentation](https://docs.withampersand.com/).

## Sample Manifests

This repository contains sample manifest files for various providers:

- [Salesforce](./examples/salesforce/amp.yaml)
- [HubSpot](./examples/hubspot/amp.yaml) 
- [GitHub](./examples/github/amp.yaml)
- And many more in their respective directories

Each sample demonstrates the proper configuration for integrating with that specific provider.

## Additional Resources

For practical guides on how to use Ampersand, please refer to the official documentation:

- [Ampersand Documentation](https://docs.withampersand.com/)
- [Getting Started](https://docs.withampersand.com/docs/quickstart)
- [Read Actions](https://docs.withampersand.com/docs/read-actions)
- [Write Actions](https://docs.withampersand.com/docs/write-actions)
- [Subscribe Actions](https://docs.withampersand.com/subscribe-actions)
- [Proxy Actions](https://docs.withampersand.com/docs/proxy-actions)
- [Object and Field Mapping](https://docs.withampersand.com/object-and-field-mapping)

## Contributing

Enhancements to the samples and schema documentation are welcome! Please feel free to submit pull requests.

## License

This repository is licensed under the **MIT license**.

To read this license, please see [LICENSE](https://github.com/amp-labs/samples/blob/main/LICENSE).