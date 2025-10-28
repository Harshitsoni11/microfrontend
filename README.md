# Micro Frontends Demo (Host + Chat + Email)

This workspace contains a Module Federation-based micro frontends setup:
- `host` (shell) loads remote components at runtime
- `chat-app` (remote) exposes `./Chat`
- `email-app` (remote) exposes `./Email`

The apps communicate via window CustomEvents and share a consistent dark UI.

## Tools and Frameworks

- Webpack 5 + Module Federation (runtime composition of micro-apps)
- webpack-dev-server (local dev per app)
- React 19 (shared as singleton across all apps)
- Babel (ESNext/JSX transpilation)
- HtmlWebpackPlugin (HTML template)

## Setup and Run

Prerequisites: Node.js 18+ and npm

Install dependencies in each app folder:

```bash
cd "chat-app" && npm install
cd "email-app" && npm install
cd "host" && npm install
```

Run in three terminals:

1) chat-app (port 3001)
```bash
cd "chat-app"
npm start
# Verify: http://localhost:3001/remoteEntry.js returns JS
```

2) email-app (port 3002)
```bash
cd "email-app"
npm start
# Verify: http://localhost:3002/remoteEntry.js returns JS
```

3) host (port 3000)
```bash
cd "host"
npm start
# Open http://localhost:3000
```

If you change webpack configs, stop and restart the affected dev servers.

## Key Architectural Decisions and Trade-offs

- Module Federation
  - Host declares remotes:
    - `chatApp@http://localhost:3001/remoteEntry.js`
    - `emailApp@http://localhost:3002/remoteEntry.js`
  - Remotes expose contracts:
    - `chat-app`: `./Chat`
    - `email-app`: `./Email`
  - Host consumes via dynamic imports (React.lazy): `import("chatApp/Chat")`
  - Trade-off: powerful runtime composition vs. added complexity in build/runtime coordination.

- Shared React as Singleton
  - `react` and `react-dom` shared as singletons with non-strict versions to avoid conflicts.
  - Trade-off: one shared instance simplifies interop but limits using incompatible React versions across apps.

- Async Bootstrap Entries
  - Each app uses `import('./bootstrap')` in `src/index.js` to avoid eager consumption of shared modules.
  - Trade-off: slightly more files/indirection, but better startup reliability.

- Cross-App Communication via Events
  - Lightweight `window` `CustomEvent` bus.
  - `chat-app` emits `mf:chat:newMessage`; `email-app` subscribes and renders notifications.
  - Trade-off: simple and decoupled; for complex state, consider a federated store remote.

- Styling and UX
  - Unified dark theme via inline styles for simplicity.
  - Trade-off: inline styles are quick; a shared design system remote is better for scale.

## Troubleshooting

- Blank page or runtime errors
  - Confirm both remote entries load (200 OK):
    - http://localhost:3001/remoteEntry.js
    - http://localhost:3002/remoteEntry.js
  - Hard refresh the host page (Ctrl+F5) after restarting servers.

- "Shared module is not available for eager consumption"
  - Ensure async bootstrap is used in each app and React is shared as singleton with `strictVersion: false`.

- URIError due to `%PUBLIC_URL%`
  - CRA placeholders removed; static paths like `/favicon.ico` are used in HtmlWebpackPlugin templates.

## Adding Future Micro-Apps

1) Create a new remote with Webpack 5 + Module Federation on a new port (e.g., 3003).
2) Expose a contract, e.g., `"./Notifications": "./src/Notifications"`.
3) Add the remote to `host/webpack.config.js` `remotes` and render with `React.lazy(() => import("notificationsApp/Notifications"))`.
4) Optionally integrate with the event bus or a shared store remote.

## Production Considerations

- Use environment-configured or dynamic remotes (e.g., a `remotes.json`) to add/upgrade remotes without rebuilding the host.
- Publish each remote’s `remoteEntry.js` to a CDN and deploy independently.
- Add error boundaries, retries, and telemetry for remote load failures.
