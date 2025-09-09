# Running Bitaxe Sentry Locally (Without Docker)

## Prerequisites

- Python 3.11+ (recommended)
- pip package manager

## Setup Instructions

### 1. Create a Virtual Environment (Recommended)
```bash
cd bitaxe-sentry
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 2. Install Dependencies
```bash
pip install -r bitaxe_sentry/requirements.txt
```

### 3. Create Data Directory
```bash
mkdir -p data
```

### 4. Set Environment Variables (Optional)
```bash
export DB_PATH="$(pwd)/data/bitaxe_sentry.db"
export DB_DATA_DIR="$(pwd)/data"
```

Or create a `.env` file in the project root:
```env
DB_PATH=/path/to/bitaxe-sentry/data/bitaxe_sentry.db
DB_DATA_DIR=/path/to/bitaxe-sentry/data
```

## Running the Application

### Option 1: Run Both Services Manually

**Terminal 1 - Background Monitor Service:**
```bash
cd bitaxe-sentry
python -m bitaxe_sentry.sentry
```

**Terminal 2 - Web UI:**
```bash
cd bitaxe-sentry
uvicorn bitaxe_sentry.sentry.webapp:app --host 0.0.0.0 --port 8000
```

### Option 2: Run Web UI Only (Basic Monitoring)
```bash
cd bitaxe-sentry
uvicorn bitaxe_sentry.sentry.webapp:app --host 0.0.0.0 --port 8000 --reload
```

**Note:** The web UI now automatically initializes the database tables on startup, so you don't need to run the background service first.

## Access the Application

- **Web Dashboard:** http://localhost:8000
- **Settings:** http://localhost:8000/settings
- **History:** http://localhost:8000/history

## Configuration

1. Open http://localhost:8000/settings
2. Configure your Bitaxe miner endpoints (e.g., `192.168.1.100`, `192.168.1.101`)
3. Set up Discord webhook URL (optional)
4. Adjust monitoring thresholds
5. Click "Save Settings"

## Directory Structure

After running locally, you'll have:
```
bitaxe-sentry/
├── data/
│   ├── bitaxe_sentry.db    # SQLite database
│   ├── config.json         # Settings file
│   └── sentry.pid          # Process ID file
├── bitaxe_sentry/
│   └── ...                 # Application code
└── LOCAL_SETUP.md          # This file
```

## Service Descriptions

### Background Monitor (`python -m bitaxe_sentry.sentry`)
- Polls configured Bitaxe miners every X minutes
- Stores readings in SQLite database
- Sends Discord alerts for threshold violations
- Cleans up old data automatically
- Handles configuration reloads

### Web UI (`uvicorn bitaxe_sentry.sentry.webapp:app`)
- Dashboard showing current miner status
- Historical charts and data visualization
- Settings configuration interface
- REST API endpoints
- Manual poll triggering

## Development Mode

For development with auto-reload:
```bash
# Web UI with auto-reload
uvicorn bitaxe_sentry.sentry.webapp:app --host 0.0.0.0 --port 8000 --reload

# Background service (restart manually when code changes)
python -m bitaxe_sentry.sentry
```

## Stopping the Application

- **Web UI:** Ctrl+C in the uvicorn terminal
- **Background Service:** Ctrl+C in the sentry terminal
- **Clean Shutdown:** The application will clean up PID files and close connections

## Troubleshooting

### Common Issues

1. **Import Errors**: Make sure you're in the `bitaxe-sentry` directory and dependencies are installed
2. **Permission Errors**: Ensure the `data/` directory is writable
3. **Port Already in Use**: Change the port with `--port 8001`
4. **Database Locked**: Stop all running instances before restarting
5. **"No such table" Errors**: ✅ Fixed! The web UI now auto-creates database tables

### Debug Mode
Add `--log-level debug` to uvicorn for verbose logging:
```bash
uvicorn bitaxe_sentry.sentry.webapp:app --host 0.0.0.0 --port 8000 --log-level debug
```

### Environment Variables
If you need different paths:
```bash
export DB_PATH="/custom/path/bitaxe_sentry.db"
export DB_DATA_DIR="/custom/path"
python -m bitaxe_sentry.sentry
```

## Production Considerations

For production deployment without Docker:
1. Use a process manager like `systemd` or `supervisor`
2. Set up proper logging rotation
3. Configure firewall rules for port 8000
4. Use a reverse proxy (nginx/Apache) in front of uvicorn
5. Enable HTTPS with SSL certificates

### Systemd Service Example
Create `/etc/systemd/system/bitaxe-sentry.service`:
```ini
[Unit]
Description=Bitaxe Sentry Monitor
After=network.target

[Service]
Type=simple
User=bitaxe
WorkingDirectory=/opt/bitaxe-sentry
Environment=DB_DATA_DIR=/var/lib/bitaxe
ExecStart=/opt/bitaxe-sentry/venv/bin/python -m bitaxe_sentry.sentry
Restart=always

[Install]
WantedBy=multi-user.target
```