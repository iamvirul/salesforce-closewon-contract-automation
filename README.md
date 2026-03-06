# CloseWon Contract Automation

Automatically send DocuSign contracts when Salesforce opportunities are marked as Closed-Won.

## Overview

This Ballerina integration listens to Salesforce opportunity change events and automatically triggers DocuSign contract creation and dispatch when an opportunity reaches the "Closed Won" stage and meets configured criteria.

**Key Technologies:**
- `ballerinax/salesforce` - Salesforce connector for event listening and data retrieval
- `ballerinax/docusign.dsesign` - Official DocuSign eSignature connector for envelope creation

## Features

- **Event-Driven Architecture**: Listens to Salesforce Platform Events for real-time opportunity updates
- **Configurable Business Rules**: Set minimum deal values and other criteria for contract dispatch
- **Flexible Signer Selection**: Choose from primary contact, billing contact, or other roles
- **Template Management**: Support multiple DocuSign templates based on product/deal type
- **Field Mapping**: Automatically pre-fill DocuSign fields from Salesforce opportunity data
- **CC Recipients**: Configure multiple CC recipients for contract notifications
- **Expiration Reminders**: Set expiration reminder days per template
- **Stage Updates**: Automatically update opportunity stage to "Contract Sent" after dispatch

## Architecture

### Components

- **main.bal**: Salesforce listener service that handles opportunity change events
- **automation.bal**: Core automation logic for contract creation and dispatch
- **functions.bal**: Helper functions for Salesforce data retrieval and business logic
- **connections.bal**: Client and listener initialization
- **types.bal**: Type definitions for Salesforce and DocuSign data structures
- **config.bal**: Configuration variables
- **data_mappings.bal**: Data transformation utilities
- **agents.bal**: Placeholder for future AI agent integrations

## Configuration

Create a `Config.toml` file with the following configuration:

### Salesforce Configuration

```toml
# Listener authentication (username/password)
salesforceUsername = "your-username@example.com"
salesforcePassword = "your-password-with-security-token"

# Client API authentication (OAuth)
salesforceClientId = "your-connected-app-client-id"
salesforceClientSecret = "your-connected-app-client-secret"
salesforceRefreshToken = "your-refresh-token"
salesforceRefreshUrl = "https://login.salesforce.com/services/oauth2/token"
salesforceBaseUrl = "https://login.salesforce.com"

# Change event channel
salesforceChannelName = "/data/ChangeEvents"
```

### DocuSign Configuration

```toml
docusignAccountId = "your-account-id"
docusignAccessToken = "your-access-token"
docusignBaseUrl = "https://demo.docusign.net/restapi"
```

**Note:** The integration uses the official `ballerinax/docusign.dsesign` connector. Ensure you have a valid DocuSign access token with the required scopes for envelope creation.

### Template Configuration

```toml
# Default template
defaultTemplateId = "your-default-template-id"

# Multiple templates by product/deal type
[[templateConfigs]]
templateId = "template-id-for-enterprise"
productType = "Enterprise"
expirationDays = 7

[[templateConfigs]]
templateId = "template-id-for-professional"
productType = "Professional"
expirationDays = 5
```

### Signer Configuration

```toml
# Signer role selection
# Options: "Primary Contact", "Billing Contact", "Decision Maker", "Executive Sponsor"
signerRole = "Primary Contact"

# CC recipients
[[ccRecipients]]
email = "legal@yourcompany.com"
name = "Legal Team"

[[ccRecipients]]
email = "sales-ops@yourcompany.com"
name = "Sales Operations"
```

### Field Mappings

```toml
[[fieldMappings]]
opportunityField = "Name"
docusignField = "OpportunityName"

[[fieldMappings]]
opportunityField = "Amount"
docusignField = "ContractValue"

[[fieldMappings]]
opportunityField = "CloseDate"
docusignField = "CloseDate"
```

### Business Rules

```toml
minimumDealValue = 10000.0
expirationReminderDays = 3
contractSentStage = "Contract Sent"
```

## Setup Instructions

### Prerequisites

1. **Salesforce Setup**:
   - Enable Change Data Capture for Opportunity object
   - Create a Connected App for OAuth authentication
   - Obtain OAuth credentials (client ID, client secret, refresh token)
   - Create a user with API access for the listener

2. **DocuSign Setup**:
   - Create DocuSign account (demo or production)
   - Create contract templates with named fields
   - Generate access token for API authentication
   - Note your account ID

### Installation

1. Clone this repository
2. Create `Config.toml` with your credentials
3. Run the integration:

```bash
bal run
```

## How It Works

1. **Event Reception**: The Salesforce listener receives opportunity update events
2. **Criteria Check**: The system validates:
   - Opportunity stage is "Closed Won"
   - Deal value meets minimum threshold
   - Required data is present
3. **Contact Retrieval**: Fetches the appropriate signer based on configured role
4. **Template Selection**: Chooses the right template based on opportunity type
5. **Envelope Creation**: Creates DocuSign envelope with:
   - Pre-filled fields from opportunity data
   - Configured signer
   - CC recipients
   - Expiration settings
6. **Stage Update**: Updates Salesforce opportunity to "Contract Sent"

## Data Flow

```
Salesforce Opportunity Update (Closed Won)
    ↓
Salesforce Platform Event
    ↓
Ballerina Listener Service
    ↓
Validation & Business Rules Check
    ↓
Retrieve Contact Information
    ↓
Select DocuSign Template
    ↓
Create & Send DocuSign Envelope
    ↓
Update Salesforce Opportunity Stage
```

## Error Handling

The integration includes comprehensive error handling:
- Validates opportunity and contact data
- Checks for missing required fields
- Handles DocuSign API errors
- Logs all operations for troubleshooting

## Extensibility

### Future Enhancements

The `agents.bal` file provides a foundation for AI-powered features:
- Contract review and analysis
- Intelligent template selection
- Risk assessment
- Automated field extraction

## Troubleshooting

### Common Issues

1. **Authentication Errors**:
   - Verify Salesforce credentials and security token
   - Check OAuth token validity
   - Ensure Connected App permissions

2. **Event Not Received**:
   - Verify Change Data Capture is enabled for Opportunity
   - Check channel name configuration
   - Review Salesforce event monitoring

3. **DocuSign Errors**:
   - Verify template ID exists
   - Check field names match template
   - Ensure access token is valid

## License

Copyright © 2024. All rights reserved.
