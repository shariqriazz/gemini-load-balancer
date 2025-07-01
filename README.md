DEPRECIATED IN FAVOR OF 
https://github.com/shariqriazz/load-balancer/

# Gemini Load Balancer

A modern NextJS application that serves as a proxy server for the Google Gemini API, with key management, load balancing, and a beautiful UI. This application allows you to efficiently manage multiple Gemini API keys, automatically rotate between them and to monitor your API usage with detailed statistics.

![Gemini Load Balancer](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse2.mm.bing.net%2Fth%3Fid%3DOIP.8qJFfRHJ4w1imQm4ATTdcAHaFl%26pid%3DApi&f=1&ipt=967751466c9cf44b0aa649a4db4edd75c0fba31eeac5b92c314a7c161ab215a1&ipo=images)

Thanks to @SannidhyaSah for his contribution to this application

## Features

- **API Key Management**: Add, remove, and monitor your Gemini API keys
- **Load Balancing**: Automatically rotate between multiple API keys to avoid rate limits
- **Usage Statistics**: Monitor your API usage with detailed charts and metrics
- **Logs Viewer**: View and search through request, error, and key event logs
- **API Playground**: Test the Gemini API directly from the UI
- **Dark/Light Mode**: Toggle between dark and light themes
- **Single Command Execution**: Run both frontend and backend with a single command
- **Security Features**: Key masking, rate limit handling, and failure detection
- **Responsive Design**: Works seamlessly on desktop and mobile devices
- **Real-time Monitoring**: Live updates of key status and usage metrics
- **Customizable Settings**: Configure key rotation, rate limits, and more
- **Import/Export Keys**: Backup and restore your API keys via JSON files.

## Architecture

The Gemini Load Balancer is built using Next.js App Router, which allows for a unified application that handles both the frontend UI and backend API routes. The application follows a modern architecture:

- **Frontend**: React with Chakra UI and Tailwind CSS for a responsive and accessible interface
- **Backend**: Next.js API routes that proxy requests to the Gemini API
- **State Management**: React Context API and SWR for efficient data fetching
- **Data Storage**: SQLite database (`data/database.db`) for API keys, settings, and detailed request logs (`request_logs` table). File-based storage for supplementary debugging logs (incoming requests, errors, key events). Statistics are primarily derived from the database.
- **Styling**: Chakra UI with Tailwind CSS for a consistent design system
- **Error Handling**: Comprehensive error logging and monitoring
- **Type Safety**: Full TypeScript implementation

## Prerequisites

- Node.js 18+ or Bun runtime (Recommended)
- Git (for version control)
- A Gemini API key (for testing)

## Installation

Make sure you have Node.js and Bun installed. Then, clone the repository and install the dependencies:

```bash
# Clone the repository
git clone https://github.com/shariqriazz/gemini-load-balancer.git
cd gemini-load-balancer

# Install dependencies (choose one)
bun install
# OR
npm install --legacy-peer-deps # Use if you encounter peer dependency issues
```

## Configuration

The application requires minimal configuration:

1. Create a `.env` file in the project root (or copy from `.env.example`)
2. Set the necessary environment variables. The `.env.example` file provides comments explaining each variable. Here's an example structure:

```env
# Server Configuration
# The port number the application will run on
PORT=4265

# Admin Login
# Password to access the admin dashboard
ADMIN_PASSWORD=your_secret_admin_password_here # Example: iwfgQ4Qx3YgCzL4KDO0ZXKB5AQwRXk51
# This password is also used to encrypt sensitive data like session information.

# Optional Admin Login Enforcement
# Set to 'false' to disable the admin login requirement for the entire application.
# Set to 'true' or leave blank to enforce admin login.
REQUIRE_ADMIN_LOGIN=true

# Master API Key for Incoming Requests (Optional)
# If set, this single key MUST be provided as a Bearer token in the Authorization header
# for incoming requests to the /api/v1/chat/completions endpoint.
# This adds an authentication layer to YOUR API endpoint.
# Leave blank to skip this specific incoming authentication check.
MASTER_API_KEY=
```

Note: Google Gemini API keys and rotation settings are managed through the UI (stored in the `data/database.db` SQLite database), not directly in the `.env` file.

## Recommended Settings

For optimal performance and reliability, we recommend the following configuration:

### API Key Management

- Add at least 5 API keys for proper load balancing
- Keep at least 3 backup keys for failover
- Remove keys that consistently fail or get rate limited

### Performance Settings

- Set key rotation request count to 3-5 requests per key
- Set rate limit cooldown to 60 seconds
- Configure max failure count to 3 before key deactivation
- Enable automatic key rotation

### Monitoring

- Check the dashboard daily for key health
- Review error logs every few days
- Monitor key usage distribution for balance
- Keep track of error rates for each key

### Best Practices

- Regularly rotate your API keys (every 30-90 days)
- Keep your total request count below 80% of your quota
- Use the playground to test new prompts before production
- Save frequently used prompts for consistency
- Tag or label your keys based on their usage (e.g., "production", "testing")

### Resource Management

- Set log retention to 14-30 days to manage storage
- Archive important logs before deletion
- Regularly backup your `data/database.db` file.
- Clean up unused saved prompts periodically

## Running the Application

Development mode with hot reloading:

```bash
# Using Bun
bun dev
# OR (using explicit run command)
# bun run dev
```

Production deployment:

```bash
# Build the application
bun build
# OR (using explicit run command)
# bun run build

# Start the production server
bun start
# OR (using explicit run command)
# bun run start
```

The application will be available at http://localhost:4269 (or your configured PORT)

Using PM2 for process management:

```bash
# Ensure pm2 is installed globally (e.g., npm install -g pm2 or bun install -g pm2)

# Start the application using pm2 with bun
pm2 start bun --name gemini-load-balancer -- start

# OR Start the application using pm2 with npm
# pm2 start npm --name gemini-load-balancer -- run start

# Monitor the application
# pm2 list
# pm2 logs gemini-load-balancer
```

## Security Considerations

1. **API Key Protection**:

   - Keys are stored in the SQLite database (`data/database.db`). Ensure appropriate file system permissions for this file.
   - Keys are masked in the UI and logs
   - Access to the admin panel is protected by the `ADMIN_PASSWORD` set in the `.env` file. This password also encrypts sensitive data.

2. **Rate Limiting**:

   - Built-in rate limit detection
   - Automatic key rotation on rate limits
   - Configurable cooldown periods

3. **Error Handling**:
   - Failed keys are automatically disabled
   - Comprehensive error logging
   - Automatic retry mechanisms

## Using as an API Service

To use this load balancer as an API service for your applications:

1. Start the application and access the UI at http://localhost:4269
2. Go to the "API Keys" section and add your Gemini API keys through the UI
3. In your client application, configure the following:
   - Base URL: `http://localhost:4269/api/v1` (or your deployed URL)
   - Authorization Header:
     - If `MASTER_API_KEY` is set in the server's `.env` file, incoming requests to `/api/v1/chat/completions` **must** include the header `Authorization: Bearer <MASTER_API_KEY>`.
     - If `MASTER_API_KEY` is **not** set (left blank) in the `.env` file, this specific authorization check is skipped. The load balancer will still use its managed Google Gemini keys for outgoing requests.
   - Model: Will be automatically populated from the available Gemini models

Example configuration in your client:

```javascript
const configuration = {
  baseURL: "http://localhost:4269/api/v1",
  // apiKey: "any-string-works", // If MASTER_API_KEY is not set on server
  // OR
  // headers: { Authorization: "Bearer your_secret_master_key_here" } // If MASTER_API_KEY is set on server
  model: "gemini-2.5-pro-exp", // Available models are shown in the dropdown
};
```

## Upgrading from Previous Versions

If you're upgrading from a version that used JSON files for storage (keys.json and settings.json) to this version which uses SQLite database, follow these steps to ensure a smooth upgrade:

### Step 1: Update Your Code

First, update your local repository to get the latest code:

```bash
# Navigate to your project directory
cd path/to/gemini-load-balancer

# Pull the latest changes from the repository
git pull origin main

# If you have local changes, you might need to stash them first:
# git stash
# git pull origin main
# git stash pop
```

### Step 2: Install New Dependencies

This version requires additional dependencies for SQLite database support. Install them using:

```bash
# Using Bun (recommended)
bun install

# OR using npm
npm install --legacy-peer-deps
```

### Step 3: Run the Migration Script

Now you need to migrate your existing data from JSON files to the SQLite database:

```bash
# Using Bun
bun scripts/migrate-json-to-db.js

# OR using Node.js
node scripts/migrate-json-to-db.js
```

This script will:
1. Read your existing data from `data/keys.json` and `data/settings.json`
2. Migrate all data to the SQLite database (`data/database.db`)
3. Preserve all your API keys, their statistics, and application settings
4. Log the migration progress

It's recommended to back up your `data` folder before migration. The script is safe to run multiple times as it will skip existing entries.

### Step 4: Start the Updated Application

After successful migration, start the application as usual:

```bash
# Development mode
bun dev

# OR production mode
bun build
bun start
```

The application will automatically use the database for all operations. The original JSON files will not be modified or deleted, but they will no longer be used.

### Troubleshooting

If you encounter any issues during migration:

1. Check that the `data` directory has correct permissions
2. Ensure your JSON files contain valid data
3. Check the console output for specific error messages
4. If migration fails, you can try again after fixing any issues

For database issues after migration, you can check the database integrity:

```bash
# Using SQLite command line (if installed)
sqlite3 data/database.db "PRAGMA integrity_check;"
```

## Development

### Project Structure

```
gemini-load-balancer/
├── data/                        # Data storage (ensure this directory is writable by the application)
│   └── database.db            # SQLite database for keys and settings
├── logs/                        # Log files (ensure this directory is writable)
├── scripts/                     # Utility scripts
│   └── migrate-json-to-db.js    # Script to migrate old JSON data to SQLite
├── public/                      # Static assets
├── src/                         # Source code
│   ├── app/                     # Next.js App Router
│   │   ├── api/                 # API routes
│   │   │   ├── admin/           # Admin API endpoints
│   │   │   │   ├── keys/        # Key management (CRUD, Import, Export)
│   │   │   │   └── cleanup-logs/ # Log cleanup endpoint
│   │   │   ├── logs/            # Logs API endpoint (for viewing file logs)
│   │   │   ├── settings/        # Settings API endpoint
│   │   │   ├── stats/           # Statistics API endpoint (DB-driven)
│   │   │   └── v1/              # Gemini API proxy endpoints
│   │   ├── dashboard/           # Dashboard page
│   │   ├── keys/                # Key management page
│   │   ├── logs/                # Logs viewer page
│   │   ├── playground/          # API playground page
│   │   ├── settings/            # Settings page
│   │   └── stats/               # Statistics page
│   ├── components/              # React components
│   ├── contexts/                # React contexts
│   ├── hooks/                   # Custom React hooks
│   └── lib/                     # Library code
│       ├── models/              # Data models (ApiKey, RequestLog, etc.)
│       ├── services/            # Services
│       └── utils/               # Utility functions
```

### Adding Features

To add new features to the Gemini Load Balancer:

1. **Frontend Components**: Add new components in the `src/components` directory
2. **API Routes**: Add new API routes in the `src/app/api` directory
3. **Pages**: Add new pages in the `src/app` directory

### Technology Stack

- **Framework**: Next.js 14+
- **UI Library**: Chakra UI with Tailwind CSS
- **State Management**: React Context API + SWR for data fetching
- **API Communication**: Built-in Next.js API routes + Axios for external calls
- **Charts**: Recharts for usage statistics
- **Database**: SQLite (via `sqlite` and `sqlite3` packages)
- **Concurrency**: `async-mutex`
- **Package Manager**: Bun (Recommended)
- **Styling**: Chakra UI + Tailwind CSS
- **Icons**: React Icons

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

ISC
