# Workflows Shared

This repository contains shared workflow implementations and tools.

## Automated Git Commit Workflow (Work in Progress)

### Overview

An n8n workflow that automatically commits and pushes Git changes with AI-generated commit messages, running every 15 minutes.

### Features

- **Automated Scheduling**: Runs every 15 minutes to check for changes
- **Smart Detection**: Only commits when there are actual file changes
- **AI Commit Messages**: Uses AI to generate meaningful commit messages based on file changes
- **Git Operations**: Handles add, commit, and push operations automatically

### Workflow Structure

1. **Schedule Trigger** - Every 15 minutes
2. **Git Status** - Check current repository state
3. **If Changes Exist** - Only proceed if files were modified/added/deleted
4. **Generate Commit Message** - AI creates meaningful commit message
5. **Git Add** - Stage all changes
6. **Git Commit** - Commit with AI-generated message
7. **Git Push** - Push to remote repository

### Files

- `automated-git-commit-workflow.json` - Complete n8n workflow export
- `implementation_plan.md` - Detailed implementation documentation

### Setup Requirements

- n8n instance with Git nodes enabled
- Local Git repository at specified path (`/home/krusty/projects/workflows-shared`)
- AI language model configured (OpenAI, Anthropic, etc.)
- Git credentials configured for push operations

### Configuration Notes

- AI Agent requires a language model connection (currently empty in export - configure in n8n UI)
- Repository path is hardcoded - adjust for your environment
- Schedule interval can be modified in the Schedule Trigger node
- Error handling can be enhanced by connecting error outputs to notification nodes

### Usage

1. Import the JSON file into n8n
2. Configure AI language model connection for the "Generate Commit Message" node
3. Update repository path if different from `/home/krusty/projects/workflows-shared`
4. Configure Git credentials if needed
5. Activate the workflow

### Security Considerations

- Ensure repository credentials are properly secured
- Test in a non-production environment first
- Monitor for conflicts if multiple users push to the same repository
- Consider repository permissions and access controls

### Maintenance

- Monitor workflow executions in n8n
- Check for failed executions and resolve issues
- Update AI prompts if needed for better commit messages
- Adjust schedule timing if conflicts arise

Created as part of automated workflow implementation project.

## Futures Analyst Workflow

### Overview

An n8n workflow that generates cryptocurrency futures trading recommendations using AI-powered technical analysis. Users can request analysis for specific cryptocurrencies and receive complete 4-hour trade setups with entry points, stop-loss levels, and take-profit targets.

### Features

- **Chat-Based Interface**: Receive analysis by sending chat messages like "what's the forecast for Bitcoin?"
- **Real-Time Market Data**: Automatically fetches current price, trading volume, order books, and market data from exchanges
- **AI-Powered Analysis**: Uses AI agent with multiple tools to perform technical analysis using MACD, RSI, EMAs, and Bollinger Bands
- **Complete Trade Setup**: Delivers entry price, stop-loss, and take-profit targets in a formatted recommendation

### Workflow Structure

1. **Chat Message Trigger** - Receives user requests for crypto analysis
2. **Market Data Fetch** - Pulls real-time data from cryptocurrency exchanges
3. **AI Analysis Agent** - Processes data using multiple tools and trading indicators
4. **Trade Recommendation** - Generates detailed 4-hour forecast report
5. **User Response** - Returns formatted trade analysis to the chat

### Files

- `Futures Analyst V3.json` - Latest version of the complete n8n workflow export
- `Futures Analyst.json` - Previous workflow version
- `Futures-Analyst V2.json` - Second version of the workflow
- `Futures-Analyst.json` - Initial workflow version
- `Futures-Analyst.md` - Detailed documentation with images and explanations

### Setup Requirements

- n8n instance with chat triggers and AI agent nodes enabled
- Cryptocurrency API connections for market data
- AI language model configured (OpenAI, Anthropic, etc.)
- Chat platform integration for user communication

### Configuration Notes

- AI Agent requires a language model connection and appropriate prompts for trading analysis
- Cryptocurrency market data sources must be configured (fees may apply)
- Chat integration must be set up for receiving and sending messages
- Analysis can be customized for different cryptocurrencies and timeframes

### Usage

1. Import one of the JSON workflow files into n8n
2. Configure AI language model connection for analysis
3. Set up cryptocurrency exchanges and API connections
4. Configure chat platform integration
5. Send messages like "analyze Bitcoin" or "what's the forecast for ETH?"
6. Receive detailed trade recommendations within seconds

### Security Considerations

- Ensure API keys are properly secured
- Secure chat platform credentials
- Be aware of cryptocurrency market risks and trading disclaimers
- Test with demo accounts before live trading

### Maintenance

- Monitor AI prompt effectiveness and update for better accuracy
- Refresh API connections and update exchange APIs as needed
- Track successful analysis rates and user feedback
- Update trading indicators and analysis methods based on market conditions

## Get Distance To Meeting And Customer Workflow

### Overview

A subworkflow that calculates driving distances from participant locations to both customer addresses and meeting locations using OpenRouteService API. Designed as a subworkflow for larger scheduling and transportation coordination systems.

### Features

- **Address Geocoding**: Converts customer and meeting addresses to geographic coordinates
- **Driving Distance Calculation**: Computes actual driving distances using professional routing service
- **Participant Location Integration**: Includes participant current location for personalized calculations
- **Batch Processing**: Processes multiple addresses efficiently

### Workflow Structure

1. **Address Input** - Receives customer address details (street, number, city, country)
2. **Geocode Customer** - Converts customer address to coordinates
3. **Geocode Meeting** - Converts meeting location to coordinates
4. **Distance to Customer** - Calculates participant distance to customer location
5. **Distance to Meeting** - Calculates participant distance to meeting location
6. **Results Output** - Returns distance data for both locations

### Files

- `Get Distance To Meeting And Customer.json` - Complete n8n subworkflow export

### Setup Requirements

- n8n instance with HTTP Request nodes
- OpenRouteService API key (free tier available)
- Geocoding and distance calculation APIs configured
- GPS/location data from participants (longitude/latitude)

### Configuration Notes

- OpenRouteService API key must be configured for geocoding and routing
- Input format requires specific address components (street, house number, city, country)
- Driving distance calculations use car's driving time and distance
- Retry logic is built-in for API reliability

### Usage

1. Import the JSON file into n8n as a subworkflow
2. Configure OpenRouteService API key in HTTP request nodes
3. Call the subworkflow with address data and participant location
4. Receive distance calculations for planning and scheduling decisions

### Security Considerations

- API keys should be stored securely using n8n credentials
- Location data handling complies with privacy regulations
- Distance data can be used for routing optimization

### Maintenance

- Monitor API usage and costs
- Update API connections if service changes
- Validate distance accuracy against known routes
- Handle API rate limits and error responses

## TimeSheet Delivery CSV Workflow

### Overview

An n8n workflow that processes employee timesheet data from Google Sheets, aggregates it by user and project, generates CSV files, and delivers them via WhatsApp messaging. Includes support for date range filtering, travel time tracking, and batch message delivery.

### Features

- **Google Sheets Integration**: Automatically pulls timesheet data from spreadsheets
- **Date Range Filtering**: Optional date-based filtering for specific reporting periods
- **Travel Time Tracking**: Separately tracks and includes travel time in calculations
- **CSV Generation**: Creates formatted CSV files with user summary and project details
- **WhatsApp Delivery**: Sends CSV files via WhatsApp to assigned users
- **Batch Processing**: Handles multiple users and delivery with error handling

### Workflow Structure

1. **Webhook Trigger** - Receives delivery requests with date range and user mapping
2. **Google Sheets Fetch** - Retrieves complete timesheet data
3. **Data Aggregation** - Groups by user, project, activity with travel time inclusion
4. **CSV Generation** - Creates formatted CSV files for each user
5. **Batch Delivery** - Loops through users sending WhatsApp messages
6. **Results Summary** - Reports success/failure statistics

### Files

- `TimeSheet Delivery CSV.json` - Complete n8n workflow export

### Setup Requirements

- n8n instance with Google Sheets, HTTP Request, and Code nodes
- Google Sheets API credentials for data access
- WhatsApp/Evolution API instance for message delivery
- Google Sheets containing timesheet data in specific format

### Configuration Notes

- Google Sheets integration requires OAuth2 credentials
- Timesheet format must include columns for Name, Projekt, Tätigkeit, Stunden, Reise, etc.
- WhatsApp delivery uses Evolution API with custom endpoint
- Date filtering supports optional startDate/endDate parameters

### Usage

1. Import the JSON file into n8n
2. Configure Google Sheets OAuth2 credentials
3. Set up WhatsApp API connection details
4. Prepare phone number to name mapping in request
5. Trigger workflow with optional date range
6. CSVs are automatically sent to each user's WhatsApp

### Security Considerations

- Google Sheets access should be limited to necessary files
- WhatsApp API credentials must be secured
- User phone numbers should be properly managed
- API access should have appropriate rate limiting

### Maintenance

- Monitor WhatsApp delivery success rates
- Validate Google Sheets data format integrity
- Check API usage limits and costs
- Update user phone mappings as needed
- Archive sent CSV files for audit purposes

## AI Digital Notes Taker Workflow (Work in Progress)

### Overview

An n8n workflow that uses AI to convert handwritten notes from images into digital Markdown files with automatic categorization and saving to Google Drive. Users send images of handwritten notes via chat and receive organized digital versions.

### Features

- **Image Processing**: Uses Google Gemini Vision API to extract text from handwritten notes
- **Interactive AI Agent**: Guides users through classification and confirmation process
- **Automatic File Naming**: Generates appropriate filenames based on content
- **Google Drive Integration**: Saves properly formatted Markdown files to specified folders
- **Tag Generation**: Automatically suggests tags for better organization

### Workflow Structure

1. **Chat Message Trigger** - Receives images via chat platform
2. **AI Agent Interaction** - Guides user through text extraction and classification
3. **Image Analysis** - Extracts handwritten text using Gemini Vision API
4. **Content Classification** - Determines appropriate filename and tags
5. **User Confirmation** - Verifies extracted content before saving
6. **Google Drive Upload** - Saves final Markdown file to cloud storage

### Files

- `AIDigitalNotes.json` - Complete n8n workflow export

### Setup Requirements

- n8n instance with chat triggers and Google Drive integration
- Google Gemini Vision API credentials
- OpenAI API credentials for AI agent
- Google Drive OAuth2 credentials
- Chat platform configured for receiving image messages

### Configuration Notes

- Gemini Vision API handles different handwriting styles and quality
- AI agent uses custom prompts for note classification
- Google Drive folder structure can be configured per user/group
- File naming conventions can be customized

### Usage

1. Import the JSON workflow into n8n
2. Configure Google Gemini and OpenAI API credentials
3. Set up Google Drive OAuth2 connection
4. Configure chat platform integration
5. Send image of handwritten notes via chat
6. Follow AI prompts for content confirmation and classification
7. Receive confirmation when file is saved to Google Drive

### Security Considerations

- API keys for vision and AI services must be secured
- Google Drive permissions should be appropriately scoped
- Chat platform credentials must be protected
- Consider data privacy for sensitive note content

### Maintenance

- Monitor API usage for cost control
- Update AI prompts for better classification accuracy
- Check Google Drive organization regularly
- Validate image processing quality and error rates
