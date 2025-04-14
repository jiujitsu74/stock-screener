# Stock Screener Project

## Project Structure
- `TASKS.md`: Master task list and system architecture documentation
- `.cursor/rules/`: Directory containing Cursor IDE rules
- `ModelsV.1/`: Directory containing deployment scripts
- Various model implementations and supporting files

## Keeping Rules in Sync
1. Clone this repository on each device:
```bash
git clone https://github.com/jiujitsu74/stock-screener.git
```

2. Set up Cursor IDE on each device and point it to this repository

3. The `.cursor/rules/` directory contains all IDE-specific rules and will be automatically synchronized through Git

4. Always pull latest changes before starting work:
```bash
git pull origin main
```

5. After making changes to rules or tasks, commit and push:
```bash
git add .
git commit -m "Update rules/tasks"
git push origin main
```

## Project Documentation
See `TASKS.md` for:
- Complete task list and status
- System architecture details
- Component responsibilities
- Integration points
- Performance metrics
- Testing instructions
- Troubleshooting guide
