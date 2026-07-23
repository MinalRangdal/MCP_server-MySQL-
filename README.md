# MySQL MCP Server Integration

Connects Antigravity IDE to a MySQL database using MCP (Model Context Protocol), via the [`mysql-mcp-server`](https://github.com/designcomputer/mysql_mcp_server) project.

## How it works

```
Antigravity (AI agent) → MySQL MCP Server (uvx) → MySQL (Docker container)
```

The agent calls tools exposed by the MCP server instead of connecting to MySQL directly. Config lives in `.agents/mcp_config.json` — see `mcp.json` in this repo for the format (replace the password placeholder with your own locally).

## Setup

**1. Start MySQL in Docker**
```bash
docker run --name mysql-mcp -e MYSQL_ROOT_PASSWORD=<root_password> -p 3307:3306 -d mysql:8.0
docker update --restart unless-stopped mysql-mcp
```

**2. Create database, table, and a dedicated user**
```bash
docker exec -it mysql-mcp mysql -u root -p
```
```sql
CREATE DATABASE company_db;
USE company_db;
CREATE TABLE employees (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100),
  role VARCHAR(100)
);
INSERT INTO employees (name, role) VALUES ('Alice', 'Engineer'), ('Bob', 'Manager');

CREATE USER 'mcp_user'@'%' IDENTIFIED BY '<mcp_user_password>';
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.* TO 'mcp_user'@'%';
FLUSH PRIVILEGES;
```

**3. Install `uv`**
```bash
pip install uv
```

**4. Add the config**

Copy `mcp.json` into `.agents/mcp_config.json` in this project, and fill in your real password (keep this file local — don't commit it).

**5. Open this folder in Antigravity**

It reads `.agents/mcp_config.json` automatically and starts the MySQL tool for this project.

**6. Test**

Ask the agent: `list the tables in company_db`

Or test standalone with [MCP Inspector](https://github.com/modelcontextprotocol/inspector):
```bash
set MYSQL_HOST=127.0.0.1
set MYSQL_PORT=3307
set MYSQL_USER=mcp_user
set MYSQL_PASSWORD=<mcp_user_password>
set MYSQL_DATABASE=company_db
npx @modelcontextprotocol/inspector uvx --from mysql-mcp-server mysql_mcp_server
```

## Why these choices

- **Docker** instead of a native MySQL install — avoids OS-level password/config issues, fully reproducible
- **Dedicated `mcp_user`** instead of root — least-privilege access, no admin rights
- **Workspace-scoped config** (`.agents/mcp_config.json`) instead of global — MySQL access only available when this project is open

## Structure

```
mysql-mcp-assignment/
├── .agents/
│   └── mcp_config.json   # local only, real password, gitignored
├── mcp.json               # placeholder password, safe to push
└── README.md
```

