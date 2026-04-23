# MCP Board-O-Mate Server: Advanced Live Visual Diagramming with AI Integration

A comprehensive **TypeScript-based** system that combines **Board-O-Mate's powerful canvas (powered by Excalidraw)** with **Model Context Protocol (MCP)** integration, enabling AI agents to create and manipulate diagrams in real-time on a live canvas.

## 🚀 What This System Does

- **🎨 Live Canvas**: Real-time Board-O-Mate canvas accessible via web browser
- **🤖 AI Integration**: MCP server allows AI agents (like Claude) to create visual diagrams
- **⚡ Real-time Sync**: Elements created via MCP API appear instantly on the canvas
- **🔄 WebSocket Updates**: Live synchronization across multiple connected clients
- **🏗️ Production Ready**: Clean, minimal UI suitable for end users

## 🎥 Demo Video

> **See MCP Board-O-Mate in Action!**

[MCP Board-O-Mate Demo](https://youtu.be/eL6oeUCSU4U)

*Watch how AI agents create and manipulate diagrams in real-time on the live canvas*

## 🏛️ Architecture Overview

This project is intentionally split into **two cooperating servers** plus a **browser frontend**.

- **Canvas Server** (`src/server.ts` → `dist/server.js`)
  - Serves the **frontend** (static files built from `frontend/`) so end-users can open the canvas in a browser.
  - Exposes a **REST API** (documented below) that represents the canvas scene as a set of elements.
  - Hosts a **WebSocket** that pushes live scene updates to all connected browsers (multi-client sync).
  - Owns the **authoritative in-memory element store** for the current scene.

- **MCP Server** (`src/index.ts` → `dist/index.js`)
  - Implements the **Model Context Protocol** so AI clients can call named tools like `create_element`, `query_elements`, etc.
  - Translates MCP tool invocations into **HTTP requests** to the Canvas Server.
  - Acts as the boundary layer between “AI intent” (tool calls) and “canvas state” (elements on the scene).

- **Frontend** (`frontend/src/`)
  - A React + TypeScript app embedding the Excalidraw-based canvas experience.
  - Maintains a live view by subscribing to the Canvas Server’s WebSocket and rehydrating the scene.
  - Provides a minimal UI surface area (connection status, clear canvas) so the canvas stays end-user friendly.

### Data flow (what happens on an AI tool call)

1. An AI agent (Claude Desktop, Cursor, VS Code MCP, etc.) invokes an MCP tool (for example, `batch_create_elements`).
2. The **MCP Server** validates/normalizes the request shape and forwards it to the Canvas Server’s REST API.
3. The **Canvas Server** applies the change to the in-memory scene (create/update/delete elements).
4. The **Canvas Server** broadcasts the updated scene (or delta, depending on implementation) over WebSocket.
5. All connected browsers update immediately and render the new diagram.

### Why two servers?

- **Separation of responsibilities**: the Canvas Server focuses on being a reliable, real-time “diagram state service”, while the MCP Server focuses on “AI tool surface area” and MCP protocol compatibility.
- **Safer evolution**: MCP tools can evolve (new capabilities, stricter validation) without changing the browser app, and the canvas UX can evolve without breaking AI integrations.

### Build outputs and runtime entrypoints

- `npm run build` produces:
  - **Compiled servers** in `dist/` (`dist/index.js`, `dist/server.js`)
  - A **built frontend** that the Canvas Server serves from `dist/frontend/` (or an equivalent build output)

## 🌟 Key Features

### **Modern TypeScript Architecture**
- **Full TypeScript Migration**: Complete type safety for backend and frontend
- **Comprehensive Type Definitions**: Excalidraw elements, API responses, WebSocket messages
- **Strict Type Checking**: Enhanced development experience and compile-time error detection
- **Type-Safe React Components**: TSX components with proper props typing

### **Real-time Canvas Integration**
- Elements created via MCP appear instantly on the live canvas
- WebSocket-based real-time synchronization
- Multi-client support with live updates

### **Production-Ready Interface**
- Clean, minimal UI with connection status
- Simple "Clear Canvas" functionality
- No development clutter or debug information

### **Comprehensive MCP API**
- **Element Creation**: rectangles, ellipses, diamonds, arrows, text, lines
- **Element Management**: update, delete, query with filters
- **Batch Operations**: create multiple elements in one call
- **Advanced Features**: grouping, alignment, distribution, locking

### **Robust Architecture**
- TypeScript-based Express.js backend with REST API + WebSocket
- React frontend with official Excalidraw package and TypeScript
- Dual-path element loading for reliability
- Auto-reconnection and error handling

## 📦 Installation & Setup

### **✅ Recommended: Local Development Setup**

> **Most stable and feature-complete option**

#### **1. Clone the Repository**
```bash
git clone https://github.com/yctimlin/mcp_board_o_mate.git
cd mcp_board_o_mate
npm install
```

#### **2. Build the Frontend**
```bash
npm run build
```

#### **3. Start the System**

##### **Option A: Production Mode (Recommended)**
```bash
# Start canvas server (serves frontend + API)
npm run canvas
```

##### **Option B: Development Mode**
```bash
# Start both canvas server and Vite dev server
npm run dev
```

#### **4. Access the Canvas**
Open your browser and navigate to:
```
http://localhost:3000
```

## 🔧 Available Scripts

| Script | Description |
|--------|-------------|
| `npm start` | Build and start MCP server (`dist/index.js`) |
| `npm run canvas` | Build and start canvas server (`dist/server.js`) |
| `npm run build` | Build both frontend and TypeScript backend |
| `npm run build:frontend` | Build React frontend only |
| `npm run build:server` | Compile TypeScript backend to JavaScript |
| `npm run dev` | Start TypeScript watch mode + Vite dev server |
| `npm run type-check` | Run TypeScript type checking without compilation |
| `npm run production` | Build + start in production mode |

## 🎯 Usage Guide

### **For End Users**
1. Open the canvas at `http://localhost:3000`
2. Check connection status (should show "Connected")
3. AI agents can now create diagrams that appear in real-time
4. Use "Clear Canvas" to remove all elements

### **For AI Agents (via MCP)**
The MCP server provides these tools for creating visual diagrams:

#### **Basic Element Creation**
```javascript
// Create a rectangle
{
  "type": "rectangle",
  "x": 100,
  "y": 100, 
  "width": 200,
  "height": 100,
  "backgroundColor": "#e3f2fd",
  "strokeColor": "#1976d2",
  "strokeWidth": 2
}
```

#### **Create Text Elements**
```javascript
{
  "type": "text",
  "x": 150,
  "y": 125,
  "text": "Process Step",
  "fontSize": 16,
  "strokeColor": "#333333"
}
```

#### **Create Arrows & Lines**
```javascript
{
  "type": "arrow",
  "x": 300,
  "y": 130,
  "width": 100,
  "height": 0,
  "strokeColor": "#666666",
  "strokeWidth": 2
}
```

#### **Batch Creation for Complex Diagrams**
```javascript
{
  "elements": [
    {
      "type": "rectangle",
      "x": 100,
      "y": 100,
      "width": 120,
      "height": 60,
      "backgroundColor": "#fff3e0",
      "strokeColor": "#ff9800"
    },
    {
      "type": "text", 
      "x": 130,
      "y": 125,
      "text": "Start",
      "fontSize": 16
    }
  ]
}
```

## 🔌 Integration with Claude Desktop

### **✅ Recommended: Using Local Installation**

For the **local development version** (most stable), add this configuration to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp_board_o_mate": {
      "command": "node",
      "args": ["/absolute/path/to/mcp_board_o_mate/dist/index.js"]
    }
  }
}
```

**Important**: Replace `/absolute/path/to/mcp_board_o_mate` with the actual absolute path to your cloned repository. Note that the path now points to `dist/index.js` (the compiled TypeScript output).

## 🔧 Integration with Other Tools

### **Cursor IDE**

Add to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "mcp_board_o_mate": {
      "command": "node",
      "args": ["/absolute/path/to/mcp_board_o_mate/dist/index.js"]
    }
  }
}
```

### **VS Code MCP Extension**

For VS Code MCP extension, add to your settings:

```json
{
  "mcp": {
    "servers": {
      "mcp_board_o_mate": {
        "command": "node",
        "args": ["/absolute/path/to/mcp_board_o_mate/dist/index.js"]
      }
    }
  }
}
```

## 🛠️ Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `EXPRESS_SERVER_URL` | `http://localhost:3000` | Canvas server URL for MCP sync |
| `ENABLE_CANVAS_SYNC` | `true` | Enable/disable canvas synchronization |
| `DEBUG` | `false` | Enable debug logging |
| `PORT` | `3000` | Canvas server port |
| `HOST` | `localhost` | Canvas server host |

## 📊 API Endpoints

The canvas server provides these REST endpoints:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/elements` | Get all elements |
| `POST` | `/api/elements` | Create new element |
| `PUT` | `/api/elements/:id` | Update element |
| `DELETE` | `/api/elements/:id` | Delete element |
| `POST` | `/api/elements/batch` | Create multiple elements |
| `GET` | `/health` | Server health check |

## 🎨 MCP Tools Available

### **Element Management**
- `create_element`
  - Use when you want to create a single element (shape, arrow, text, etc.) with explicit geometry and styling.
  - The MCP server forwards this to the canvas REST API, and the element becomes visible to all connected clients via WebSocket.
- `update_element`
  - Use when you already know an element id and want to change geometry, styling, text, lock state, or other supported fields.
  - The element is updated in the authoritative scene store and broadcast live.
- `delete_element`
  - Use to remove a specific element by id.
- `query_elements`
  - Use to find elements that match filters (type, text content, bounds, grouping, etc.), then feed those ids into update/delete operations.

### **Batch Operations**
- `batch_create_elements`
  - Use to create multiple elements in one tool call (useful for flows, org charts, multi-step diagrams).
  - Batch creation reduces round-trips and ensures the canvas updates as a coherent “diagram chunk”.

### **Element Organization**  
- `group_elements`
  - Use after creating a cluster of related shapes/text so they can be moved as a unit.
- `ungroup_elements`
  - Use when you need to re-layout or edit individual members of a group.
- `align_elements`
  - Use to make diagrams look intentional (left/center/right or top/middle/bottom alignment across a selected set).
- `distribute_elements`
  - Use to evenly space elements for readability (especially in step-by-step flows).
- `lock_elements` / `unlock_elements`
  - Use to prevent accidental edits to a finished section of the diagram while iterating elsewhere.

### **Resource Access**
- `get_resource`
  - Use to fetch non-mutating resources like the current scene, element library, or theme-related data, depending on what the server exposes.

## 🏗️ Development Architecture

### **Frontend** (`frontend/src/`)
- **React + TypeScript**: Modern TSX components with full type safety
- **Vite Build System**: Fast development and optimized production builds
- **Official Excalidraw**: `@excalidraw/excalidraw` package with TypeScript types
- **WebSocket Client**: Type-safe real-time element synchronization
- **Clean UI**: Production-ready interface with proper TypeScript typing

### **Canvas Server** (`src/server.ts` → `dist/server.js`)
- **TypeScript + Express.js**: Fully typed REST API + static file serving
- **WebSocket**: Type-safe real-time client communication  
- **Element Storage**: In-memory with comprehensive type definitions
- **CORS**: Cross-origin support with proper typing

### **MCP Server** (`src/index.ts` → `dist/index.js`)
- **TypeScript MCP Protocol**: Type-safe Model Context Protocol implementation
- **Canvas Sync**: Strongly typed HTTP requests to canvas server
- **Element Management**: Full CRUD operations with comprehensive type checking
- **Batch Support**: Type-safe complex diagram creation

### **Type System** (`src/types.ts`)
- **Excalidraw Element Types**: Complete type definitions for all element types
- **API Response Types**: Strongly typed REST API interfaces
- **WebSocket Message Types**: Type-safe real-time communication
- **Server Element Types**: Enhanced element types with metadata

## 🐛 Troubleshooting

### **Canvas Not Loading**
- Ensure `npm run build` completed successfully
- Check that `dist/index.html` exists
- Verify canvas server is running on port 3000

### **Elements Not Syncing**
- Confirm MCP server is running (`npm start`)
- Check `ENABLE_CANVAS_SYNC=true` in environment
- Verify canvas server is accessible at `EXPRESS_SERVER_URL`

### **WebSocket Connection Issues**  
- Check browser console for WebSocket errors
- Ensure no firewall blocking WebSocket connections
- Try refreshing the browser page

### **Build Errors**
- Delete `node_modules` and run `npm install`
- Check Node.js version (requires 16+)
- Ensure all dependencies are installed
- Run `npm run type-check` to identify TypeScript issues
- Verify `dist/` directory is created after `npm run build:server`

## 📋 Project Structure

```
mcp_board_o_mate/
├── frontend/
│   ├── src/
│   │   ├── App.tsx          # Main React component (TypeScript)
│   │   └── main.tsx         # React entry point (TypeScript)
│   └── index.html           # HTML template
├── src/ (TypeScript Source)
│   ├── index.ts            # MCP server (TypeScript)
│   ├── server.ts           # Canvas server (Express + WebSocket, TypeScript)
│   ├── types.ts            # Comprehensive type definitions
│   └── utils/
│       └── logger.ts       # Logging utility (TypeScript)
├── dist/ (Compiled Output)
│   ├── index.js            # Compiled MCP server
│   ├── server.js           # Compiled Canvas server
│   ├── types.js            # Compiled type definitions
│   ├── utils/
│   │   └── logger.js       # Compiled logging utility
│   └── frontend/           # Built React frontend
├── tsconfig.json          # TypeScript configuration
├── vite.config.js         # Vite build configuration
├── package.json           # Dependencies and scripts
└── README.md              # This file
```

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
