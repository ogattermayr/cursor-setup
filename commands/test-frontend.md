# Test Frontend

Manually test the frontend application using browser automation.

## Prerequisites

- Frontend dev server running (`npm run dev` in howthef-frontend/)
- `agent-browser` CLI available

## Workflow

1. **Start the frontend** (if not running):
   ```bash
   cd howthef-frontend && npm run dev
   ```

2. **Open the browser**:
   ```bash
   agent-browser open http://localhost:3000
   ```

3. **Take snapshot to see interactive elements**:
   ```bash
   agent-browser snapshot -i
   ```

4. **Interact with elements using refs from snapshot**:
   ```bash
   agent-browser click @e1
   agent-browser fill @e2 "test input"
   ```

5. **Verify results**:
   ```bash
   agent-browser snapshot -i  # Check new state
   agent-browser screenshot   # Visual verification
   ```

## Common Test Scenarios

### Authentication Flow
```bash
agent-browser open http://localhost:3000/login
agent-browser snapshot -i
# Fill email and password fields
agent-browser fill @e1 "test@example.com"
agent-browser fill @e2 "testpassword"
agent-browser click @e3  # Submit button
agent-browser wait --url "**/dashboard"
agent-browser snapshot -i  # Verify logged in state
```

### Form Submission
```bash
agent-browser open http://localhost:3000/form-page
agent-browser snapshot -i
# Fill form fields and submit
agent-browser wait --text "Success"  # Verify success message
```

### Navigation Test
```bash
agent-browser open http://localhost:3000
agent-browser snapshot -i
agent-browser click @e1  # Nav link
agent-browser wait --load networkidle
agent-browser get url  # Verify correct route
```

## Debugging

```bash
agent-browser open http://localhost:3000 --headed  # Show browser
agent-browser console  # View console messages
agent-browser errors   # View page errors
agent-browser screenshot --full  # Full page screenshot
```

## Cleanup

```bash
agent-browser close
```

## Notes

- Always take a snapshot before interacting to get current element refs
- Re-snapshot after navigation or significant DOM changes
- Use `--headed` flag to see what's happening visually
- Use `agent-browser wait` commands to handle async operations
