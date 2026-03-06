# Integration Prompt: Salesforce to DocuSign Contract Automation

## Description
Create a Ballerina integration that automatically sends DocuSign contracts when Salesforce opportunities are marked as "Closed Won". This is an event-driven integration that listens to Salesforce change events in real-time and triggers contract creation based on configurable business rules.

## What It Should Do

### Event Trigger
- Listen to Salesforce Platform Events for Opportunity updates using Change Data Capture
- React specifically to opportunities reaching "Closed Won" stage
- Process events in real-time with minimal latency

### Core Functionality
1. **Event Detection**: Monitor Salesforce opportunity changes in real-time
2. **Business Rules Validation**: 
   - Verify opportunity stage is "Closed Won"
   - Check if deal value meets minimum threshold (configurable)
   - Validate required data is present (opportunity name, amount, contact email)
3. **Contact Retrieval**: Fetch the appropriate signer based on role configuration
   - Support multiple role types: Primary Contact, Billing Contact, Decision Maker, Executive Sponsor
   - Fallback to primary contact if configured role not found
4. **Template Selection**: Choose correct DocuSign template based on product/deal type
   - Support multiple templates with different configurations
   - Match by opportunity type or deal type
   - Use default template if no match found
5. **Contract Dispatch**: Create and send DocuSign envelope with:
   - Pre-filled fields from Salesforce opportunity data
   - Configured signer with routing order
   - Multiple CC recipients (Legal, Sales Ops, Finance)
   - Custom email subject
   - Expiration reminders
6. **Status Update**: Update Salesforce opportunity stage to "Contract Sent"
7. **Error Handling**: Comprehensive logging and error handling for all operations

## Technical Requirements

### Connectors & Libraries
- **ballerinax/salesforce** (v8.3.0 or later): For Salesforce event listening and data retrieval
  - Use listener service for Change Data Capture events
  - Use client for SOQL queries and data retrieval
- **ballerinax/docusign.dsesign** (v1.0.0 or later): Official DocuSign eSignature connector
  - Use for envelope creation with templates
  - Support for template roles, tabs, and routing
- **ballerinax/'client.config**: For OAuth2 configuration types
- **ballerina/log**: For comprehensive logging

### Configuration Parameters

#### Salesforce Configuration
- Username and password for listener authentication
- OAuth credentials (client ID, client secret, refresh token) for API calls
- Base URL and channel name for change events
- Support for both sandbox and production environments

#### DocuSign Configuration
- Account ID
- Access token for authentication
- Base URL (demo or production)
- Template ID management

#### Business Rules
- **Minimum Deal Value Threshold**: Only send contracts for opportunities above a certain amount
- **Signer Role Selection**: Configure which contact role to use as the signer
  - Options: Primary Contact, Billing Contact, Decision Maker, Executive Sponsor
- **Template Mapping**: Support multiple templates based on:
  - Product type
  - Deal type
  - Opportunity characteristics
- **Expiration Reminders**: Configure reminder days per template
- **Stage Update**: Define the stage name to update after contract dispatch

#### Field Mappings
- Map Salesforce Opportunity fields to DocuSign template fields
- Support for common fields:
  - Opportunity Name → Contract Name
  - Amount → Contract Value
  - Close Date → Effective Date
  - Custom fields as needed

#### CC Recipients
- Configure multiple CC recipients for contract notifications
- Support for:
  - Legal team
  - Sales operations
  - Finance department
  - Custom recipients

## Data Flow

```
1. Salesforce Opportunity Updated (Stage → "Closed Won")
   ↓
2. Salesforce Platform Event Triggered
   ↓
3. Ballerina Listener Receives Event
   ↓
4. Extract Opportunity ID from Event
   ↓
5. Retrieve Full Opportunity Details
   ↓
6. Validate Opportunity Data
   ↓
7. Check Business Rules (Stage, Amount, etc.)
   ↓
8. Query for Contact Based on Signer Role
   ↓
9. Validate Contact Data (Email, Name)
   ↓
10. Select Appropriate DocuSign Template
    ↓
11. Build Envelope with:
    - Pre-filled fields from opportunity
    - Signer information
    - CC recipients
    - Routing order
    ↓
12. Create and Send DocuSign Envelope
    ↓
13. Update Salesforce Opportunity Stage
    ↓
14. Log Success/Failure
```

## Implementation Structure

### File Organization
- **main.bal**: Salesforce listener service with event handlers
- **automation.bal**: Core automation logic for contract creation
- **functions.bal**: Helper functions for Salesforce operations
- **connections.bal**: Client and listener initialization
- **types.bal**: Type definitions for Salesforce and DocuSign entities
- **config.bal**: Configuration variable declarations
- **data_mappings.bal**: Data transformation and validation utilities
- **agents.bal**: Placeholder for future AI-powered features
- **Config.toml**: Configuration values (credentials, mappings, rules)
- **README.md**: Documentation and setup instructions

### Key Functions

#### Event Handling
- `onCreate()`: Handle new opportunity creation (typically skip)
- `onUpdate()`: Handle opportunity updates (main trigger)
- `onDelete()`: Handle opportunity deletion (no action)
- `onRestore()`: Handle opportunity restoration (no action)

#### Core Processing
- `processOpportunityForContract()`: Main orchestration function
- `getOpportunity()`: Retrieve opportunity details from Salesforce
- `validateOpportunityData()`: Validate opportunity has required fields
- `meetsDispatchCriteria()`: Check if opportunity meets business rules
- `getSignerContact()`: Retrieve contact based on configured role
- `validateContactData()`: Validate contact has required fields
- `selectTemplate()`: Choose appropriate DocuSign template
- `createAndSendEnvelope()`: Create and send DocuSign envelope
- `updateOpportunityStage()`: Update opportunity stage in Salesforce

#### Helper Functions
- `getContactByRole()`: Query Salesforce for contact by role
- `getPrimaryContact()`: Fallback to primary contact
- `buildTemplateFields()`: Map opportunity fields to DocuSign fields
- `buildSignerName()`: Construct full name from contact
- `getOpportunityFieldValue()`: Extract field value by name

## Error Handling

### Validation Errors
- Missing required fields (opportunity name, amount, contact email)
- Invalid data formats (negative amounts, invalid emails)
- Missing contact roles

### API Errors
- Salesforce authentication failures
- DocuSign API errors
- Network timeouts
- Rate limiting

### Business Logic Errors
- Opportunity doesn't meet criteria (wrong stage, below threshold)
- No contact found for specified role
- Template not found
- Envelope creation failures

### Error Response Strategy
- Log all errors with context (opportunity ID, error message)
- Return errors to caller for proper handling
- Don't fail silently - ensure visibility
- Provide fallback mechanisms (e.g., primary contact if role not found)

## Extensibility & Future Enhancements

### AI Agent Integration (agents.bal)
- **Contract Review**: Analyze contract terms before sending
- **Intelligent Template Selection**: Use ML to recommend best template
- **Risk Assessment**: Evaluate deal risk and route for approval
- **Field Extraction**: Automatically extract and populate fields
- **Complexity Analysis**: Determine contract complexity based on deal characteristics

### Additional Features
- **Multi-language Support**: Send contracts in customer's preferred language
- **Approval Workflows**: Route high-value contracts for approval before sending
- **Audit Trail**: Track all contract dispatch activities
- **Retry Mechanism**: Automatically retry failed envelope creation
- **Notification System**: Send alerts on success/failure
- **Analytics**: Track contract dispatch metrics and success rates

## Testing Considerations

### Unit Tests
- Test individual functions with mock data
- Validate data transformation logic
- Test business rule evaluation

### Integration Tests
- Test Salesforce event reception
- Test DocuSign envelope creation
- Test end-to-end flow with test accounts

### Edge Cases
- Opportunity with no contacts
- Multiple contacts with same role
- Missing optional fields
- Template not found scenarios
- Network failures and retries

## Security Considerations

### Credentials Management
- Store all credentials in Config.toml (not in code)
- Use OAuth2 refresh tokens (not static passwords where possible)
- Rotate access tokens regularly
- Use environment-specific configurations

### Data Privacy
- Handle PII (contact emails, names) securely
- Log only necessary information (avoid logging sensitive data)
- Comply with data retention policies

### API Security
- Use HTTPS for all API calls
- Validate SSL certificates
- Implement rate limiting awareness
- Handle authentication errors gracefully

## Performance Considerations

### Optimization
- Use streaming for Salesforce queries
- Minimize API calls (batch where possible)
- Cache template configurations
- Efficient error handling (fail fast)

### Scalability
- Handle high volume of opportunity updates
- Implement queuing if needed
- Monitor resource usage
- Consider async processing for non-critical operations

## Monitoring & Observability

### Logging
- Log all major operations (event received, envelope created, etc.)
- Include correlation IDs (opportunity ID, envelope ID)
- Use appropriate log levels (INFO, WARN, ERROR)
- Structured logging for easy parsing

### Metrics to Track
- Number of events received
- Number of contracts sent
- Success/failure rates
- Processing time per opportunity
- API call latency

### Alerts
- Failed envelope creation
- Authentication failures
- Repeated validation errors
- Unusual activity patterns

## Deployment

### Prerequisites
- Ballerina Swan Lake (2201.9.0 or later) installed
- Salesforce org with:
  - Change Data Capture enabled for Opportunity object
  - Connected App created for OAuth
  - User with API access
- DocuSign account with:
  - Contract templates configured with named fields
  - OAuth application created
  - Access token generated
- Network access to both Salesforce and DocuSign APIs

### Configuration Steps

#### Salesforce Setup
1. Enable Change Data Capture:
   - Go to Setup → Integrations → Change Data Capture
   - Select "Opportunity" object
   - Save changes
2. Create Connected App:
   - Go to Setup → Apps → App Manager → New Connected App
   - Enable OAuth Settings
   - Add required scopes: `api`, `refresh_token`, `offline_access`
   - Note Client ID and Client Secret
3. Generate Refresh Token:
   - Use OAuth 2.0 flow to obtain refresh token
   - Store securely for configuration

#### DocuSign Setup
1. Create Templates:
   - Log in to DocuSign
   - Go to Templates → New Template
   - Add fields with labels matching your field mappings
   - Note Template ID
2. Generate Access Token:
   - Go to Settings → Apps and Keys
   - Create new application or use existing
   - Generate access token with required scopes
   - Note Account ID and Access Token

#### Configuration File
1. Create `Config.toml` in project root
2. Add all required credentials (see Configuration section)
3. Configure business rules, field mappings, and templates
4. Test with sample data first

### Running Locally
```bash
# Install dependencies
bal build

# Run the integration
bal run

# Run with specific config file
bal run --config-file=Config.toml

# View logs
# Logs will show event reception, processing, and envelope creation
```

### Deploying on Choreo
1. Sign in to Choreo account
2. Create new Integration project
3. Import this repository
4. Select Technology: Ballerina
5. Choose Integration Type: Service
6. Configure environment variables for all credentials
7. Deploy to development environment
8. Test with Salesforce opportunity updates
9. Monitor logs for successful processing
10. Promote to production with production credentials

## Success Criteria

### Functional
- ✅ Automatically detects Closed Won opportunities
- ✅ Validates business rules before sending
- ✅ Selects correct signer based on role
- ✅ Chooses appropriate template based on deal type
- ✅ Pre-fills DocuSign fields from Salesforce data
- ✅ Includes configured CC recipients
- ✅ Updates opportunity stage after sending
- ✅ Handles errors gracefully with proper logging

### Non-Functional
- ✅ Processes events within 30 seconds
- ✅ 99%+ success rate for valid opportunities
- ✅ Comprehensive error logging
- ✅ Secure credential management
- ✅ Easy configuration and deployment
- ✅ Well-documented code and setup process

## Example Use Case

**Scenario**: Enterprise SaaS company closes a $50,000 deal

1. Sales rep updates opportunity stage to "Closed Won" in Salesforce
2. Change Data Capture triggers event
3. Integration receives event and retrieves opportunity details
4. Validates: Stage = "Closed Won", Amount = $50,000 (above $10,000 threshold)
5. Queries for "Primary Contact" on the opportunity
6. Finds contact: John Smith (john.smith@customer.com)
7. Selects "Enterprise" template based on opportunity type
8. Creates DocuSign envelope with:
   - Signer: John Smith
   - Pre-filled fields: Opportunity Name, Amount, Close Date
   - CC: legal@company.com, sales-ops@company.com
9. Sends envelope via DocuSign API
10. Updates opportunity stage to "Contract Sent"
11. Logs success with envelope ID

**Result**: Contract automatically sent to customer within seconds of closing the deal, with legal and sales ops notified.
