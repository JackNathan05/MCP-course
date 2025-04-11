
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Overview

This lesson explores how MCP can enhance development environments to provide context-aware coding assistance.

## MCP in Code Editors and IDEs

MCP enables powerful IDE integrations:

1. **Contextual code completion**
   - Semantic understanding of code
   - Project-specific suggestions
   - Pattern recognition
   - Tailored to coding style

2. **Repository analysis and navigation**
   - Codebase understanding
   - Symbol navigation
   - Dependency tracking
   - Version control integration

3. **Documentation generation**
   - Automatic documentation
   - Comment generation
   - API documentation
   - Markdown/doc generation

4. **Test case creation**
   - Automatic test generation
   - Edge case identification
   - Test coverage analysis
   - Test maintenance

5. **Code review assistance**
   - Style checking
   - Bug detection
   - Performance optimization suggestions
   - Security vulnerability identification

MCP provides a standardized way for development tools to access code context and AI capabilities, enabling more intelligent assistance while coding.

## Adding Context-Aware Coding Assistance

MCP servers can provide rich contextual information:

1. **Code structure and navigation**
   - Abstract syntax tree (AST) information
   - Symbol definitions and references
   - Class hierarchies
   - Call graphs

2. **Symbol definitions and references**
   - Variable definitions
   - Function declarations
   - Class definitions
   - Import/module information

3. **Type information and validation**
   - Variable types
   - Return types
   - Parameter types
   - Type checking

4. **Dependency information**
   - Package dependencies
   - Module imports
   - Library usage
   - Version information

5. **Project history and changes**
   - Recent modifications
   - Commit history
   - Author information
   - Issue tracking

By providing this context through MCP, AI models can offer more relevant and useful assistance to developers.

## Repository Integration Patterns

Common patterns for repository integration:

1. **Git history analysis**
   - Commit history processing
   - Change pattern recognition
   - Author contribution analysis
   - Code evolution tracking

2. **Pull request management**
   - PR description analysis
   - Code diff understanding
   - Review suggestion generation
   - Automated review comments

3. **Issue tracking**
   - Issue context retrieval
   - Related issue identification
   - Solution suggestion
   - Progress tracking

4. **Continuous integration status**
   - Build status monitoring
   - Test result analysis
   - Deployment status tracking
   - Pipeline optimization

5. **Code ownership information**
   - Author identification
   - Domain expertise mapping
   - Knowledge distribution
   - Collaborative workflow

MCP enables these integrations to be standardized across different development environments and tools.

## Code Generation Workflows

Effective code generation with MCP:

1. **Understanding project context**
   - Codebase structure analysis
   - Existing patterns recognition
   - Framework identification
   - Style guide adherence

2. **Following established patterns**
   - Style consistency
   - Naming conventions
   - Architecture patterns
   - Project-specific idioms

3. **Integrating with existing code**
   - Import management
   - Function signature matching
   - Type consistency
   - API compatibility

4. **Test-driven generation**
   - Test case analysis
   - Implementation generation to pass tests
   - Test coverage consideration
   - Edge case handling

5. **Documentation generation**
   - Comment generation
   - API documentation
   - Usage examples
   - Parameter documentation

MCP provides the necessary context for AI models to generate code that integrates well with the existing project.

## Example: VS Code MCP Integration

Here's an example of how MCP can be integrated with Visual Studio Code:

```javascript
// MCP integration for VS Code
import * as vscode from 'vscode';
import { MCPClient } from 'mcp-client';

// Extension activation
export async function activate(context: vscode.ExtensionContext) {
  const mcpClient = new MCPClient();
  let connected = false;
  
  // Set up status bar item
  const statusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right, 100);
  statusBarItem.text = "MCP: Disconnected";
  statusBarItem.command = "mcp-extension.connect";
  statusBarItem.show();
  
  // Connect to MCP Server
  const connect = async () => {
    try {
      statusBarItem.text = "MCP: Connecting...";
      
      // Connect to local MCP server (could be configurable)
      await mcpClient.connect('http://localhost:9000');
      connected = true;
      
      // Get capabilities
      const capabilities = await mcpClient.getCapabilities();
      
      // Log successful connection
      console.log(`Connected to MCP server with ${capabilities.length} capabilities`);
      statusBarItem.text = "MCP: Connected";
      
      vscode.window.showInformationMessage('Connected to MCP server successfully');
      return true;
    } catch (error) {
      console.error('Failed to connect to MCP server:', error);
      statusBarItem.text = "MCP: Disconnected";
      vscode.window.showErrorMessage('Failed to connect to MCP server');
      connected = false;
      return false;
    }
  };
  
  // Register commands
  const generateCodeCommand = vscode.commands.registerCommand('mcp-extension.generateCode', async () => {
    if (!connected) {
      const shouldConnect = await vscode.window.showInformationMessage(
        'Not connected to MCP server. Connect now?',
        'Yes', 'No'
      );
      
      if (shouldConnect !== 'Yes' || !(await connect())) {
        return;
      }
    }
    
    try {
      // Get current editor
      const editor = vscode.window.activeTextEditor;
      if (!editor) {
        vscode.window.showInformationMessage('No active editor');
        return;
      }
      
      // Get selected text or cursor position
      const selection = editor.selection;
      const selectedText = editor.document.getText(selection);
      
      // Get file content and context
      const fileContent = editor.document.getText();
      const fileName = editor.document.fileName;
      const fileExtension = fileName.split('.').pop();
      const position = editor.selection.active;
      const lineText = editor.document.lineAt(position.line).text;
      
      // Collect project context
      const projectContext = await collectProjectContext();
      
      // Prepare prompt
      let prompt = '';
      if (selectedText) {
        prompt = `I have selected this code: ${selectedText}\n\nPlease help me improve it or extend it.`;
      } else {
        prompt = `I'm at this point in my code: ${lineText}\n\nPlease suggest what I should add next.`;
      }
      
      // Use MCP tool for code generation
      const result = await mcpClient.invokeTool('code.generate', {
        prompt,
        file_content: fileContent,
        file_name: fileName,
        file_extension: fileExtension,
        selection: {
          start: {
            line: selection.start.line,
            character: selection.start.character
          },
          end: {
            line: selection.end.line,
            character: selection.end.character
          }
        },
        project_context: projectContext
      });
      
      // Handle result
      if (result && result.code) {
        // Show in preview or insert directly
        showCodePreview(result.code, result.explanation);
      } else {
        vscode.window.showInformationMessage('No code generated');
      }
    } catch (error) {
      console.error('Error generating code:', error);
      vscode.window.showErrorMessage(`Error generating code: ${error.message}`);
    }
  });
  
  // Register more commands for documentation, testing, etc.
  
  // Helper function to collect project context
  async function collectProjectContext() {
    try {
      // Get workspace folders
      const workspaceFolders = vscode.workspace.workspaceFolders;
      if (!workspaceFolders) {
        return { type: 'no_workspace' };
      }
      
      const rootPath = workspaceFolders[0].uri.fsPath;
      
      // Use MCP tool to analyze project
      return await mcpClient.invokeTool('project.analyze', {
        root_path: rootPath
      });
    } catch (error) {
      console.error('Error collecting project context:', error);
      return { type: 'error', message: error.message };
    }
  }
  
  // Helper function to show code preview
  function showCodePreview(code, explanation) {
    // Create webview panel
    const panel = vscode.window.createWebviewPanel(
      'codePreview',
      'Generated Code',
      vscode.ViewColumn.Beside,
      {
        enableScripts: true
      }
    );
    
    // Create HTML content
    panel.webview.html = `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Generated Code</title>
        <style>
          body {
            font-family: var(--vscode-font-family);
            padding: 20px;
          }
          pre {
            background-color: var(--vscode-editor-background);
            padding: 16px;
            border-radius: 4px;
            overflow: auto;
          }
          .explanation {
            margin-top: 20px;
            padding: 16px;
            background-color: var(--vscode-input-background);
            border-left: 4px solid var(--vscode-textLink-foreground);
          }
          .buttons {
            margin-top: 20px;
            display: flex;
            gap: 10px;
          }
          button {
            padding: 8px 16px;
            background-color: var(--vscode-button-background);
            color: var(--vscode-button-foreground);
            border: none;
            border-radius: 2px;
            cursor: pointer;
          }
        </style>
      </head>
      <body>
        <h2>Generated Code</h2>
        <pre><code>${escapeHtml(code)}</code></pre>
        
        <div class="explanation">
          <h3>Explanation</h3>
          <p>${escapeHtml(explanation)}</p>
        </div>
        
        <div class="buttons">
          <button id="insertBtn">Insert Code</button>
          <button id="copyBtn">Copy to Clipboard</button>
        </div>
        
        <script>
          const vscode = acquireVsCodeApi();
          
          document.getElementById('insertBtn').addEventListener('click', () => {
            vscode.postMessage({
              command: 'insert',
              code: ${JSON.stringify(code)}
            });
          });
          
          document.getElementById('copyBtn').addEventListener('click', () => {
            vscode.postMessage({
              command: 'copy',
              code: ${JSON.stringify(code)}
            });
          });
        </script>
      </body>
      </html>
    `;
    
    // Handle messages from webview
    panel.webview.onDidReceiveMessage(
      message => {
        switch (message.command) {
          case 'insert':
            insertCode(message.code);
            break;
          case 'copy':
            vscode.env.clipboard.writeText(message.code);
            vscode.window.showInformationMessage('Code copied to clipboard');
            break;
        }
      },
      undefined,
      context.subscriptions
    );
  }
  
  // Helper function to insert code
  function insertCode(code) {
    const editor = vscode.window.activeTextEditor;
    if (editor) {
      editor.edit(editBuilder => {
        editBuilder.replace(editor.selection, code);
      });
    }
  }
  
  // Helper function to escape HTML
  function escapeHtml(unsafe) {
    return unsafe
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#039;");
  }
  
  // Register connection command
  context.subscriptions.push(
    vscode.commands.registerCommand('mcp-extension.connect', connect),
    generateCodeCommand
  );
  
  // Try to connect on startup
  connect();
}

// Extension deactivation
export function deactivate() {
  // Clean up resources
}
```

## MCP Servers for Development Environments

To support IDE integration, you can create MCP servers that provide development-specific capabilities:

1. **Code Analysis Server**
   - Parse and analyze code
   - Extract symbol information
   - Provide type information
   - Generate AST

2. **Repository Server**
   - Access Git history
   - Provide commit information
   - Track file changes
   - Process pull requests

3. **Package Manager Server**
   - Query package information
   - Check dependencies
   - Resolve versions
   - Suggest updates

4. **Build System Server**
   - Run builds
   - Process build logs
   - Report errors
   - Optimize build configurations

5. **Test Runner Server**
   - Execute tests
   - Report test results
   - Generate test coverage
   - Suggest test improvements

These servers can work together to provide comprehensive development assistance.

## Real-World Example: GitHub MCP Server

Here's a Python implementation of a GitHub MCP server:

```python
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter, MCPResource
import os
import requests
import json

# GitHub API client
class GitHubClient:
    def __init__(self, token):
        self.token = token
        self.base_url = "https://api.github.com"
        self.headers = {
            "Authorization": f"token {token}",
            "Accept": "application/vnd.github.v3+json"
        }
    
    def get_repository(self, owner, repo):
        url = f"{self.base_url}/repos/{owner}/{repo}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_pull_requests(self, owner, repo, state="open"):
        url = f"{self.base_url}/repos/{owner}/{repo}/pulls"
        params = {"state": state}
        response = requests.get(url, headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()
    
    def get_pull_request(self, owner, repo, pr_number):
        url = f"{self.base_url}/repos/{owner}/{repo}/pulls/{pr_number}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_pull_request_files(self, owner, repo, pr_number):
        url = f"{self.base_url}/repos/{owner}/{repo}/pulls/{pr_number}/files"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_issues(self, owner, repo, state="open"):
        url = f"{self.base_url}/repos/{owner}/{repo}/issues"
        params = {"state": state}
        response = requests.get(url, headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()
    
    def get_issue(self, owner, repo, issue_number):
        url = f"{self.base_url}/repos/{owner}/{repo}/issues/{issue_number}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()
    
    def get_commits(self, owner, repo, path=None):
        url = f"{self.base_url}/repos/{owner}/{repo}/commits"
        params = {}
        if path:
            params["path"] = path
        response = requests.get(url, headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()
    
    def get_commit(self, owner, repo, sha):
        url = f"{self.base_url}/repos/{owner}/{repo}/commits/{sha}"
        response = requests.get(url, headers=self.headers)
        response.raise_for_status()
        return response.json()

# Create MCP server
server = MCPServer()

# Initialize GitHub client from environment variable
github_token = os.environ.get("GITHUB_TOKEN")
if not github_token:
    raise ValueError("GITHUB_TOKEN environment variable is required")

github_client = GitHubClient(github_token)

# Register tool: Get repository info
@server.tool(
    id="github.repository.info",
    description="Get information about a GitHub repository",
    parameters=[
        MCPParameter(
            name="owner",
            description="Repository owner (user or organization)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="repo",
            description="Repository name",
            type="string",
            required=True
        )
    ]
)
async def get_repository_info(owner, repo):
    """Get information about a GitHub repository"""
    try:
        repo_info = github_client.get_repository(owner, repo)
        
        # Extract relevant information
        return {
            "id": repo_info["id"],
            "name": repo_info["name"],
            "full_name": repo_info["full_name"],
            "description": repo_info["description"],
            "url": repo_info["html_url"],
            "default_branch": repo_info["default_branch"],
            "stars": repo_info["stargazers_count"],
            "forks": repo_info["forks_count"],
            "open_issues": repo_info["open_issues_count"],
            "language": repo_info["language"],
            "created_at": repo_info["created_at"],
            "updated_at": repo_info["updated_at"],
            "is_private": repo_info["private"],
            "owner": {
                "login": repo_info["owner"]["login"],
                "id": repo_info["owner"]["id"],
                "url": repo_info["owner"]["html_url"],
                "type": repo_info["owner"]["type"]
            }
        }
    except Exception as e:
        raise Exception(f"Error getting repository info: {str(e)}")

# Register tool: Get pull requests
@server.tool(
    id="github.pulls.list",
    description="Get pull requests for a GitHub repository",
    parameters=[
        MCPParameter(
            name="owner",
            description="Repository owner (user or organization)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="repo",
            description="Repository name",
            type="string",
            required=True
        ),
        MCPParameter(
            name="state",
            description="Pull request state (open, closed, all)",
            type="string",
            enum=["open", "closed", "all"],
            default="open",
            required=False
        )
    ]
)
async def get_pull_requests(owner, repo, state="open"):
    """Get pull requests for a GitHub repository"""
    try:
        pulls = github_client.get_pull_requests(owner, repo, state)
        
        # Extract relevant information
        return [
            {
                "number": pr["number"],
                "title": pr["title"],
                "url": pr["html_url"],
                "state": pr["state"],
                "created_at": pr["created_at"],
                "updated_at": pr["updated_at"],
                "user": {
                    "login": pr["user"]["login"],
                    "url": pr["user"]["html_url"]
                },
                "head": pr["head"]["ref"],
                "base": pr["base"]["ref"]
            }
            for pr in pulls
        ]
    except Exception as e:
        raise Exception(f"Error getting pull requests: {str(e)}")

# Register tool: Get pull request details
@server.tool(
    id="github.pulls.get",
    description="Get details of a specific pull request",
    parameters=[
        MCPParameter(
            name="owner",
            description="Repository owner (user or organization)",
            type="string",
            required=True
        ),
        MCPParameter(
            name="repo",
            description="Repository name",
            type="string",
            required=True
        ),
        MCPParameter(
            name="pr_number",
            description="Pull request number",
            type="integer",
            required=True
        )
    ]
)
async def get_pull_request_details(owner, repo, pr_number):
    """Get details of a specific pull request"""
    try:
        pr = github_client.get_pull_request(owner, repo, pr_number)
        files = github_client.get_pull_request_files(owner, repo, pr_number)
        
        # Extract relevant information
        return {
            "number": pr["number"],
            "title": pr["title"],
            "body": pr["body"],
            "state": pr["state"],
            "url": pr["html_url"],
            "created_at": pr["created_at"],
            "updated_at": pr["updated_at"],
            "merged_at": pr["merged_at"],
            "mergeable": pr["mergeable"],
            "additions": pr["additions"],
            "deletions": pr["deletions"],
            "changed_files": pr["changed_files"],
            "user": {
                "login": pr["user"]["login"],
                "url": pr["user"]["html_url"]
            },
            "head": {
                "ref": pr["head"]["ref"],
                "sha": pr["head"]["sha"],
                "repo": pr["head"]["repo"]["full_name"] if pr["head"]["repo"] else None
            },
            "base": {
                "ref": pr["base"]["ref"],
                "sha": pr["base"]["sha"],
                "repo": pr["base"]["repo"]["full_name"]
            },
            "files": [
                {
                    "filename": file["filename"],
                    "status": file["status"],
                    "additions": file["additions"],
                    "deletions": file["deletions"],
                    "changes": file["changes"],
                    "patch": file.get("patch", "")
                }
                for file in files
            ]
        }
    except Exception as e:
        raise Exception(f"Error getting pull request details: {str(e)}")

# Register more tools for issues, commits, etc.

# Run the server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"GitHub MCP server running on port {port}")
```

## Benefits of MCP for Development Environments

Using MCP for development environments provides several key benefits:

1. **Standardized Integration**
   - Common protocol across tools
   - Simplified integration
   - Consistent experience
   - Tool interoperability

2. **Enhanced Context Awareness**
   - Access to project-wide context
   - Cross-file understanding
   - Repository-level insights
   - Ecosystem awareness

3. **AI Model Flexibility**
   - Easy switching between models
   - Consistent interface for tools
   - Capability discovery
   - Future compatibility

4. **Privacy and Security**
   - Local processing options
   - Controlled data sharing
   - Selective context provision
   - Authentication and authorization

5. **Extensibility**
   - Easy addition of new capabilities
   - Custom tool integration
   - Tailored to specific needs
   - Community-driven expansion

These benefits make MCP an attractive option for enhancing development environments with AI capabilities.

## Summary

MCP enables powerful IDE integrations by providing contextual coding assistance, repository integration, and intelligent code generation capabilities. By standardizing these integrations through the Model Context Protocol, development tools can offer more consistent, powerful, and flexible AI-assisted features to developers.
