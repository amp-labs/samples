# Ampersand Manifest Specification Reference

## Table of Contents

- [Introduction](#introduction)
- [Schema Versioning](#schema-versioning)
- [Top-Level Structure](#top-level-structure)
- [Integration Configuration](#integration-configuration)
- [Authentication Configuration](#authentication-configuration)
- [Read Configuration](#read-configuration)
  - [Read Object Configuration](#read-object-configuration)
  - [Field Configuration](#field-configuration)
  - [Backfill Configuration](#backfill-configuration)
- [Write Configuration](#write-configuration)
- [Subscribe Configuration](#subscribe-configuration)
  - [Event Configuration](#event-configuration)
  - [Filtering and Transformation](#filtering-and-transformation)
- [Proxy Configuration](#proxy-configuration)
- [Object and Field Mapping](#object-and-field-mapping)
  - [Cross-Provider Standardization](#cross-provider-standardization)
  - [Custom Field Mapping](#custom-field-mapping)
  - [Mapping Examples](#mapping-examples)
- [Common Patterns](#common-patterns)
  - [Required vs. Optional Fields](#required-vs-optional-fields)
  - [Field Selection Prompts](#field-selection-prompts)
  - [Scheduling Patterns](#scheduling-patterns)
- [Webhook Configuration](#webhook-configuration)
- [Provider-Specific Considerations](#provider-specific-considerations)
- [Examples](#examples)

## Introduction

An Ampersand manifest (`amp.yaml`) is a YAML configuration file that defines how your application integrates with various SaaS platforms through the Ampersand integration platform. The manifest follows a structured schema that enables you to specify:

- What data to read from external systems
- What data to write to external systems
- How to subscribe to events from external systems
- How to proxy API calls
- Authentication and scheduling configurations
- Object and field mapping across different providers

This document provides a comprehensive reference for the Ampersand manifest schema, including detailed field descriptions, examples, and best practices.

## Schema Versioning

The Ampersand manifest schema follows semantic versioning with the format `MAJOR.MINOR.PATCH`:

```yaml
specVersion: 1.0.0
```

- **MAJOR**: Incremented for incompatible changes that require manifest updates
- **MINOR**: Incremented for new features added in a backward-compatible manner
- **PATCH**: Incremented for backward-compatible bug fixes and clarifications

When a new schema version is released:
- Breaking changes are only introduced in MAJOR version increments
- Each version of the schema has its own JSON Schema definition in `/schema/vX/`
- The repository includes migration guides for moving between MAJOR versions

Current supported versions:
- 1.0.0: Initial stable release (current)

## Top-Level Structure

The Ampersand manifest starts with a top-level structure that specifies the schema version and a list of integrations:

```yaml
specVersion: 1.0.0
integrations:
  - name: integration1
    provider: provider1
    # ... integration 1 configuration
  - name: integration2
    provider: provider2
    # ... integration 2 configuration
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `specVersion` | String | Yes | The version of the manifest specification. Format: `MAJOR.MINOR.PATCH`. Currently only `1.0.0` is supported. |
| `integrations` | Array | Yes | An array of integration configurations. Each element defines a complete integration with a specific provider. |

### Notes:
- Only one `specVersion` is allowed per manifest file.
- The `integrations` array must contain at least one integration configuration.
- Multiple integrations can reference the same provider with different configurations.

## Integration Configuration

Each integration defines a connection to a specific provider (e.g., Salesforce, HubSpot). This section contains the basic metadata about the integration:

```yaml
name: my-integration
displayName: My Integration
provider: salesforce
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | String | Yes | A unique identifier for this integration within your manifest. Use lowercase alphanumeric characters, hyphens and underscores only. Maximum length: 64 characters. |
| `displayName` | String | No | A human-readable name for this integration. This name is displayed in the UI and logs. If not provided, `name` is used. Maximum length: 100 characters. |
| `provider` | String | Yes | The provider/platform for this integration. Must be one of the supported providers (e.g., `salesforce`, `hubspot`, `intercom`). Case-sensitive. |

### Naming Conventions:

- `name`: Use descriptive, unique names that identify the purpose of the integration. Examples: `salesforce-contacts-sync`, `hubspot-crm-integration`, `github-issues-tracker`.
- `displayName`: Use proper capitalization and spacing for better readability in UIs. Examples: "Salesforce Contacts Sync", "HubSpot CRM Integration", "GitHub Issues Tracker".

### Provider Support:

The `provider` field must match one of the supported providers. Common providers include:

- `salesforce`: Salesforce CRM
- `hubspot`: HubSpot CRM and marketing
- `intercom`: Intercom customer messaging
- `github`: GitHub repositories and issues
- `jira`: Jira project management
- `apollo`: Apollo.io sales intelligence
- `aha`: Aha! product management
- `ashby`: Ashby recruiting
- `attio`: Attio relationship management
- `brevo`: Brevo marketing
- `close`: Close CRM
- `gong`: Gong sales intelligence
- `heyreach`: HeyReach LinkedIn automation
- `instantly`: Instantly email automation
- `iterable`: Iterable marketing
- `kit`: Kit CRM
- `klaviyo`: Klaviyo marketing
- `marketo`: Marketo marketing
- `monday`: Monday.com project management
- `outreach`: Outreach sales
- `pipedrive`: Pipedrive CRM
- `salesloft`: SalesLoft sales engagement
- `smartlead`: SmartLead lead generation
- `stripe`: Stripe payments
- `zoho`: Zoho CRM
- `zendesk`: Zendesk customer service

## Read Configuration

The `read` section defines what data to read from the provider. This section is optional - if your integration only writes data, subscribes to events, or uses proxy, you can omit this section.

```yaml
read:
  objects:
    - objectName: contact
      destination: defaultWebhook
      schedule: "*/30 * * * *"
      requiredFields:
        - fieldName: firstname
        - fieldName: lastname
      optionalFields:
        - fieldName: phone
      backfill:
        defaultPeriod:
          fullHistory: true
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objects` | Array | Yes | An array of objects to read from the provider. Each element defines a specific object type (e.g., contacts, leads) to read. |

### Read Object Configuration

Each object in the `objects` array defines a specific data type to read from the provider:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objectName` | String | Yes | The name of the object to read (e.g., `contact`, `lead`, `account`). This must match an object type supported by the provider's API. Case-sensitive. |
| `mapToName` | String | No | An optional name mapping for this object. Used to standardize object names across different providers. |
| `mapToDisplayName` | String | No | An optional display name mapping for this object. Used for UI display. |
| `destination` | String | Yes | The webhook destination for the read data. This determines where the data will be sent after it's read. Must match a webhook configured in your Ampersand account. |
| `schedule` | String | Yes | A cron schedule for reading the data. Defines how frequently the data should be read. Uses standard cron syntax. Examples: `*/10 * * * *` (every 10 minutes), `0 0 * * *` (daily at midnight). |
| `requiredFields` | Array | Yes | Fields that are always read for every customer. These fields will always be included in the data that's read. |
| `optionalFields` | Array | No | Optional fields that can be included. Customers can choose which of these fields to include. |
| `optionalFieldsAuto` | String | No | Set to `all` to automatically include all available fields. This can be used instead of explicitly listing optional fields. |
| `backfill` | Object | No | Configuration for backfilling historical data. Defines how historical data should be loaded. |

#### Object Name Conventions

The `objectName` must match the API object name in the provider's system. Some common examples:

- Salesforce: `contact`, `lead`, `account`, `opportunity`
- HubSpot: `contacts`, `companies`, `deals`, `tickets`
- GitHub: `repos`, `issues`, `pulls`, `users`
- Intercom: `contacts`, `companies`, `conversations`, `teams`

#### Destination Configuration

The `destination` field must match a webhook configured in your Ampersand account. Common patterns include:

- Provider-specific webhooks: `salesforceWebhook`, `hubspotWebhook`, `intercomWebhook`
- Generic webhooks: `defaultWebhook`, `dataWebhook`, `analyticsWebhook`

#### Schedule Formatting

The `schedule` field uses standard cron syntax with five fields:

```
┌─────────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌─────────── day of month (1 - 31)
│ │ │ ┌───────── month (1 - 12)
│ │ │ │ ┌─────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
│ │ │ │ │
* * * * *
```

Common patterns:
- `*/10 * * * *`: Every 10 minutes
- `*/30 * * * *`: Every 30 minutes
- `0 * * * *`: Every hour at minute 0
- `0 0 * * *`: Every day at midnight
- `0 12 * * *`: Every day at noon
- `0 0 * * 1`: Every Monday at midnight

### Field Configuration

Fields can be specified in two ways:

1. Simple field name:
```yaml
- fieldName: firstname
```

2. Mapped field with display name and prompt:
```yaml
- mapToName: priority
  mapToDisplayName: Priority Score
  prompt: Which field do you use to track the priority of a lead?
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fieldName` | String | Yes* | The name of the field in the provider's API. Must match a field available in the provider's API for the specified object. Case-sensitive. |
| `mapToName` | String | No* | An optional name mapping for this field. Used to standardize field names across different providers. |
| `mapToDisplayName` | String | No* | An optional display name mapping for this field. Used for UI display. |
| `prompt` | String | No* | An optional prompt to show when configuring this field. Helps users understand what the field is for. |

*Either `fieldName` or all of `mapToName`, `mapToDisplayName`, and `prompt` are required.

#### Field Naming Conventions

The `fieldName` must match the API field name in the provider's system. Some common examples:

- Salesforce: `Id`, `FirstName`, `LastName`, `Email`, `Phone`
- HubSpot: `firstname`, `lastname`, `email`, `phone`, `company`
- GitHub: `id`, `name`, `full_name`, `owner`, `private`
- Intercom: `id`, `name`, `email`, `phone`, `created_at`

### Backfill Configuration

The `backfill` section configures how historical data is loaded:

```yaml
backfill:
  defaultPeriod:
    fullHistory: true
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `defaultPeriod` | Object | No | The default time period for backfilling. |
| `defaultPeriod.fullHistory` | Boolean | No | Whether to backfill the full history (`true`) or just recent data (`false`). Default: `false`. |

#### Backfill Considerations:

- Setting `fullHistory: true` may result in longer initial sync times, especially for large datasets.
- Some providers may have API rate limits that affect backfill performance.
- By default, if `backfill` is not specified, only recent data will be loaded.

## Write Configuration

The `write` section defines what data can be written to the provider. This section is optional - if your integration only reads data, subscribes to events, or uses proxy, you can omit this section.

```yaml
write:
  objects:
    - objectName: contact
      inheritMapping: true
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objects` | Array | Yes | An array of objects that can be written to. Each element defines a specific object type (e.g., contacts, leads) that can be written to. |

### Write Object Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objectName` | String | Yes | The name of the object to write to. Must match an object type supported by the provider's API. Case-sensitive. |
| `inheritMapping` | Boolean | No | Whether to inherit mappings from read configuration. If `true`, field mappings defined in the read configuration will be used for writing. Default: `false`. |

#### Write Object Considerations:

- The `objectName` must match an object type that can be written to in the provider's API.
- Not all objects that can be read can also be written to.
- Some providers have read-only objects or fields.
- When `inheritMapping: true` is set, field mappings defined in the read configuration will be used for writing, which simplifies configuration for bidirectional syncs.

## Subscribe Configuration

The `subscribe` section defines how to subscribe to events from the provider. This section is optional - if your integration only reads or writes data or uses proxy, you can omit this section.

```yaml
subscribe:
  objects:
    - objectName: account
      destination: accountWebhook
      createEvent:
        enabled: always
      updateEvent:
        enabled: always
        watchFieldsAuto: all  # Alternatively, use requiredWatchFields
      deleteEvent:
        enabled: always
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objects` | Array | Yes | An array of objects to subscribe to. Each element defines a specific object type to receive events for. |

### Subscribe Object Configuration

Each object in the `objects` array defines a specific object type to subscribe to:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `objectName` | String | Yes | The name of the object to subscribe to. Must match an object type supported by the provider. Case-sensitive. |
| `destination` | String | Yes | The webhook destination for the event data. This determines where the event data will be sent. Must match a webhook configured in your Ampersand account. |
| `inheritFieldsAndMapping` | Boolean | Yes | Whether to inherit field mappings from read configuration. For Salesforce, this must be set to `true`. |
| `createEvent` | Object | No | Configuration for create events. |
| `updateEvent` | Object | No | Configuration for update events. |
| `deleteEvent` | Object | No | Configuration for delete events. |

### Event Configuration

Each event type can be configured with specific options:

1. **Create Event Configuration**:
```yaml
createEvent:
  enabled: always
```

2. **Update Event Configuration**:
```yaml
updateEvent:
  enabled: always
  watchFieldsAuto: all  # Watch all fields for changes
```

Alternatively:
```yaml
updateEvent:
  enabled: always
  requiredWatchFields:  # Watch specific fields
    - name
    - email
    - phone
```

3. **Delete Event Configuration**:
```yaml
deleteEvent:
  enabled: always
```

#### Subscribe Considerations:

- Currently, subscribe actions are only supported for Salesforce and HubSpot.
- For Salesforce, an admin needs to install the integration.
- You must also have a read action specified for the object you wish to subscribe to.
- The `inheritFieldsAndMapping` field must be set to `true` for Salesforce.
- Only one of `requiredWatchFields` or `watchFieldsAuto` should be provided for update events.

## Proxy Configuration

The `proxy` section configures API call proxying. This allows direct access to the provider's API through Ampersand's proxy. This section is optional - if your integration only reads or writes data or subscribes to events, you can omit this section.

```yaml
proxy:
  enabled: true
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `enabled` | Boolean | No | Whether proxy is enabled for this integration. Default: `false`. |

#### Proxy Considerations:

- Enabling proxy allows direct API calls to the provider through Ampersand.
- This is useful for operations not covered by the standard read/write/subscribe functionality.
- All proxy calls are authenticated using the integration's credentials.
- API rate limits of the provider still apply.
- Proxy requests may be logged by Ampersand for security and debugging purposes.

## Object and Field Mapping

Object and field mapping is a powerful feature in Ampersand that allows you to standardize your data model across different providers. This is particularly useful when integrating with multiple providers and wanting to present a unified data model to your application.

### Cross-Provider Standardization

When working with multiple providers, you often need to map different provider-specific object and field names to a common data model. For example:

- Salesforce's `Account` and HubSpot's `company` might both represent the same concept in your data model.
- Salesforce's `FirstName` and HubSpot's `firstname` might both represent the same field.

Ampersand's mapping capabilities allow you to define a standard data model and map provider-specific objects and fields to it.

### Custom Field Mapping

Field mapping allows you to:

1. **Standardize field names**: Map different provider-specific field names to a consistent naming convention.
2. **Handle custom fields**: Map custom fields from providers to your standardized data model.
3. **Transform data formats**: Standardize how data is formatted across different providers.
4. **Simplify integration**: Present a consistent API to your application, regardless of the underlying provider.

### Mapping Examples

#### Object Mapping Example:

```yaml
# Salesforce
- objectName: Account
  mapToName: company
  mapToDisplayName: Company

# HubSpot
- objectName: company
  mapToName: company
  mapToDisplayName: Company
```

This maps both Salesforce's `Account` and HubSpot's `company` to a standardized `company` object in your data model.

#### Field Mapping Example:

```yaml
# Salesforce Contact
- objectName: Contact
  requiredFields:
    - fieldName: FirstName
      mapToName: first_name
    - fieldName: LastName
      mapToName: last_name
    - fieldName: Email
      mapToName: email_address

# HubSpot Contact
- objectName: contact
  requiredFields:
    - fieldName: firstname
      mapToName: first_name
    - fieldName: lastname
      mapToName: last_name
    - fieldName: email
      mapToName: email_address
```

This maps fields from both providers to a standardized data model with `first_name`, `last_name`, and `email_address` fields.

#### Field Selection with Prompts:

When different customers use different field names for the same concept, you can use prompts to help them select the appropriate fields:

```yaml
- mapToName: status
  mapToDisplayName: Status
  prompt: Pick the label that you use to track the status of a lead.

- mapToName: priority
  mapToDisplayName: Priority
  prompt: Which field do you use to track the priority of this task?
```

#### Inheritance in Write Configuration:

For bidirectional syncs, you can inherit mappings from the read configuration:

```yaml
read:
  objects:
    - objectName: Contact
      mapToName: contact
      requiredFields:
        - fieldName: FirstName
          mapToName: first_name
        - fieldName: LastName
          mapToName: last_name

write:
  objects:
    - objectName: Contact
      inheritMapping: true  # Inherits the mappings from the read configuration
```

### Mapping Best Practices:

1. **Consistent naming**: Use a consistent naming convention for your standardized data model (e.g., lowercase with underscores).
2. **Descriptive prompts**: When using field selection prompts, make them clear and descriptive.
3. **Inheritance**: Use `inheritMapping: true` for bidirectional syncs to avoid duplication.
4. **Documentation**: Document your data model and mapping decisions.

## Common Patterns

### Required vs. Optional Fields

Fields can be categorized as required (always included) or optional (customer can choose):

```yaml
requiredFields:
  - fieldName: firstname
  - fieldName: lastname
  - fieldName: email
optionalFields:
  - fieldName: phone
  - fieldName: address
  - fieldName: company
  - fieldName: title
```

Alternatively, you can automatically include all available fields:

```yaml
requiredFields:
  - fieldName: firstname
  - fieldName: lastname
  - fieldName: email
optionalFieldsAuto: all
```

#### Field Selection Considerations:

- Required fields are always included in the data that's read.
- Optional fields allow customers to choose which fields to include.
- Using `optionalFieldsAuto: all` includes all available fields beyond the required ones.
- Be mindful of API rate limits and data volume when including many optional fields.

### Field Selection Prompts

When mapping fields, you can provide prompts to help users select the appropriate fields. This is especially useful when different customers use different field names for the same concept:

```yaml
- mapToName: status
  mapToDisplayName: Status
  prompt: Pick the label that you use to track the status of a lead.

- mapToName: priority
  mapToDisplayName: Priority
  prompt: Which field do you use to track the priority of this task?

- mapToName: linkedin_url
  mapToDisplayName: LinkedIn URL
  prompt: Which do field do you use for the LinkedIn URL for the contact?
```

### Scheduling Patterns

Reads are scheduled using cron syntax. Here are some common patterns:

| Pattern | Description |
|---------|-------------|
| `*/10 * * * *` | Every 10 minutes |
| `*/30 * * * *` | Every 30 minutes |
| `0 * * * *` | Every hour at minute 0 |
| `0 0 * * *` | Every day at midnight |
| `0 12 * * *` | Every day at noon |
| `0 0 * * 1` | Every Monday at midnight |
| `0 0 1 * *` | First day of each month at midnight |
| `0 0/12 * * *` | Every 12 hours (at midnight and noon) |

#### Scheduling Considerations:

- Higher frequency (e.g., every 10 minutes) provides more up-to-date data but increases API usage.
- Lower frequency (e.g., daily) reduces API usage but data may be less current.
- Consider the provider's API rate limits when setting schedules.
- For less frequently updated data (e.g., user profiles), a lower frequency may be sufficient.
- For rapidly changing data (e.g., chat messages), a higher frequency is recommended.

## Webhook Configuration

Webhooks are used to receive data from Ampersand. Each read, write, or subscribe operation can specify a destination webhook where the data will be sent.

```yaml
destination: salesforceWebhook
```

The `destination` field must match a webhook configured in your Ampersand account. You can configure webhooks in the Ampersand Console.

### Webhook Types:

1. **Destination Webhooks**: Used for read operations. The data read from the provider is sent to this webhook.
2. **Event Webhooks**: Used for subscribe operations. Events from the provider are sent to this webhook.
3. **Response Webhooks**: Used for write operations. The response from the write operation is sent to this webhook.

### Webhook Configuration in Ampersand Console:

When configuring a webhook in the Ampersand Console, you need to specify:

- **Name**: A unique identifier for the webhook (e.g., `salesforceWebhook`).
- **URL**: The URL where the webhook data will be sent.
- **Headers**: Any headers that should be included with the webhook request.
- **Authentication**: Optional authentication for the webhook.

### Webhook Payload Structure:

The payload of a webhook depends on the operation:

1. **Read Operation Payload**:
```json
{
  "provider": "salesforce",
  "objectType": "Contact",
  "operation": "read",
  "timestamp": "2023-05-01T12:34:56Z",
  "data": [
    {
      "id": "001ABC123",
      "firstName": "John",
      "lastName": "Doe",
      "email": "john.doe@example.com"
    },
    // ...
  ]
}
```

2. **Subscribe Operation Payload**:
```json
{
  "provider": "salesforce",
  "eventType": "Contact.created",
  "timestamp": "2023-05-01T12:34:56Z",
  "data": {
    "id": "001ABC123",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com"
  }
}
```

3. **Write Operation Response Payload**:
```json
{
  "provider": "salesforce",
  "objectType": "Contact",
  "operation": "write",
  "timestamp": "2023-05-01T12:34:56Z",
  "status": "success",
  "data": {
    "id": "001ABC123"
  }
}
```

### Webhook Considerations:

- Webhook endpoints must be publicly accessible.
- Consider implementing authentication for your webhook endpoints.
- Webhook requests may be retried if the initial request fails.
- Webhook endpoints should respond with a 2xx status code to acknowledge receipt.
- Consider implementing idempotent processing for webhook events to handle potential duplicates.

## Provider-Specific Considerations

Different providers have different API structures, limitations, and best practices. Here are some provider-specific considerations:

### Salesforce

- Object names are case-sensitive (e.g., `Contact`, `Lead`, `Account`).
- Field names are also case-sensitive (e.g., `FirstName`, `LastName`, `Email`).
- Salesforce has API rate limits that vary by edition and license type.
- Custom fields in Salesforce have the `__c` suffix (e.g., `Custom_Field__c`).

### HubSpot

- Object names are lowercase (e.g., `contact`, `company`, `deal`).
- Field names are also lowercase (e.g., `firstname`, `lastname`, `email`).
- HubSpot has API rate limits that reset daily.
- Custom fields in HubSpot are prefixed with their object type (e.g., `contact.custom_field`).

### GitHub

- API paths often include the repository name (e.g., `repos/{owner}/{repo}/issues`).
- GitHub has API rate limits that reset hourly.
- Authentication requires appropriate scopes for the operations being performed.

### Intercom

- Intercom uses ID-based references extensively.
- Intercom has API rate limits that vary by plan.
- Certain operations require specific permissions.

## Examples

### Read Action Example

```yaml
specVersion: 1.0.0
integrations:
  - name: salesforce-read
    provider: salesforce
    read:
      objects:
        - objectName: contact
          destination: contactWebhook
          schedule: "*/30 * * * *"
          requiredFields:
            - fieldName: firstname
            - fieldName: lastname
            - fieldName: email
          optionalFields:
            - fieldName: phone
          backfill:
            defaultPeriod:
              days: 30
```

### Write Action Example

```yaml
specVersion: 1.0.0
integrations:
  - name: salesforce-write
    provider: salesforce
    write:
      objects:
        - objectName: contact
          inheritMapping: true
```

### Subscribe Action Example

```yaml
specVersion: 1.0.0
integrations:
  - name: salesforce-subscribe
    provider: salesforce
    subscribe:
      objects:
        - objectName: account
          destination: accountWebhook
          inheritFieldsAndMapping: true
          createEvent:
            enabled: always
          updateEvent:
            enabled: always
            watchFieldsAuto: all
          deleteEvent:
            enabled: always
    read:  # Required for subscribe actions
      objects:
        - objectName: account
          destination: accountWebhook
          requiredFields:
            - fieldName: id
            - fieldName: name
            - fieldName: billingcity
              mapToName: city
          backfill:
            defaultPeriod:
              fullHistory: true
```

### Proxy Action Example

```yaml
specVersion: 1.0.0
integrations:
  - name: salesforce-proxy
    provider: salesforce
    proxy:
      enabled: true
```

For more examples, see the examples in this repository.