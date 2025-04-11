
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 1/_Lesson 1|_Lesson 1]]

---

## Practical Exercise 4.1

**Objective**: Implement an MCP server for IDE integration

### Instructions

1. Create an MCP server that provides:
   - Code structure information for a repository
   - Documentation lookup
   - Dependency information

2. Connect the server to an IDE or editor:
   - Use VS Code, JetBrains IDE, or another editor of your choice
   - Create a simple extension or plugin to connect to your MCP server
   - Implement basic UI for interacting with the server

3. Build a simple workflow that uses these capabilities:
   - Code analysis and navigation
   - Documentation generation
   - Dependency visualization

4. Document the integration patterns used:
   - How the IDE communicates with the MCP server
   - How context is shared between them
   - How results are presented to the user

### Expected Outcome

A working IDE integration that:
- Connects to your MCP server
- Provides useful development assistance
- Demonstrates the benefits of MCP for IDE integration
- Is documented with clear integration patterns

## Code Example (Python with VS Code)

Here's a starter implementation for a code structure MCP server:

```python
# MCP server for IDE integration
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPResource
import os
import ast
import json
import glob

# Create server instance
server = MCPServer()

# Tool: Parse Python file structure
@server.tool(
    id="code.structure.python",
    description="Analyzes the structure of a Python file",
    parameters=[
        {
            "name": "file_path", 
            "type": "string", 
            "description": "Path to the Python file"
        }
    ]
)
async def parse_python_structure(file_path):
    try:
        with open(file_path, 'r') as file:
            code = file.read()
        
        # Parse the AST
        tree = ast.parse(code)
        
        # Extract information
        functions = []
        classes = []
        imports = []
        variables = []
        
        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                functions.append({
                    "name": node.name,
                    "line": node.lineno,
                    "args": [arg.arg for arg in node.args.args],
                    "docstring": ast.get_docstring(node)
                })
            elif isinstance(node, ast.ClassDef):
                methods = []
                class_variables = []
                
                for item in node.body:
                    if isinstance(item, ast.FunctionDef):
                        methods.append({
                            "name": item.name,
                            "line": item.lineno,
                            "args": [arg.arg for arg in item.args.args],
                            "docstring": ast.get_docstring(item)
                        })
                    elif isinstance(item, ast.Assign):
                        for target in item.targets:
                            if isinstance(target, ast.Name):
                                class_variables.append({
                                    "name": target.id,
                                    "line": item.lineno
                                })
                
                classes.append({
                    "name": node.name,
                    "line": node.lineno,
                    "methods": methods,
                    "variables": class_variables,
                    "bases": [base.id if isinstance(base, ast.Name) else "Complex" for base in node.bases],
                    "docstring": ast.get_docstring(node)
                })
            elif isinstance(node, ast.Import):
                for name in node.names:
                    imports.append({
                        "name": name.name,
                        "alias": name.asname,
                        "line": node.lineno
                    })
            elif isinstance(node, ast.ImportFrom):
                for name in node.names:
                    imports.append({
                        "name": f"{node.module}.{name.name}",
                        "alias": name.asname,
                        "line": node.lineno
                    })
            elif isinstance(node, ast.Assign) and all(isinstance(target, ast.Name) for target in node.targets):
                for target in node.targets:
                    variables.append({
                        "name": target.id,
                        "line": node.lineno
                    })
        
        return {
            "file": os.path.basename(file_path),
            "functions": functions,
            "classes": classes,
            "imports": imports,
            "variables": variables
        }
    except Exception as e:
        return {"error": str(e)}

# Tool: Analyze project structure
@server.tool(
    id="project.structure",
    description="Analyzes the structure of a project directory",
    parameters=[
        {
            "name": "project_path", 
            "type": "string", 
            "description": "Path to the project directory"
        }
    ]
)
async def analyze_project_structure(project_path):
    try:
        if not os.path.isdir(project_path):
            return {"error": "Path is not a directory"}
        
        # Initialize structure
        structure = {
            "name": os.path.basename(project_path),
            "path": project_path,
            "files": [],
            "directories": [],
            "summary": {
                "total_files": 0,
                "languages": {},
                "file_types": {}
            }
        }
        
        # File extensions to language mapping
        ext_to_language = {
            ".py": "Python",
            ".js": "JavaScript",
            ".ts": "TypeScript",
            ".html": "HTML",
            ".css": "CSS",
            ".java": "Java",
            ".c": "C",
            ".cpp": "C++",
            ".cs": "C#",
            ".go": "Go",
            ".rb": "Ruby",
            ".php": "PHP",
            ".rs": "Rust",
            ".swift": "Swift",
            ".kt": "Kotlin",
            ".md": "Markdown",
            ".json": "JSON",
            ".xml": "XML",
            ".yaml": "YAML",
            ".yml": "YAML",
            ".txt": "Text",
        }
        
        # Walk through directory
        for root, dirs, files in os.walk(project_path):
            # Skip hidden directories
            dirs[:] = [d for d in dirs if not d.startswith('.')]
            
            rel_path = os.path.relpath(root, project_path)
            if rel_path == ".":
                rel_path = ""
            
            # Process files
            for file in files:
                if file.startswith('.'):
                    continue
                
                file_path = os.path.join(root, file)
                file_rel_path = os.path.join(rel_path, file) if rel_path else file
                
                # Get file extension
                _, ext = os.path.splitext(file)
                
                # Update summary
                structure["summary"]["total_files"] += 1
                
                if ext in ext_to_language:
                    language = ext_to_language[ext]
                    structure["summary"]["languages"][language] = structure["summary"]["languages"].get(language, 0) + 1
                
                structure["summary"]["file_types"][ext] = structure["summary"]["file_types"].get(ext, 0) + 1
                
                # Add file to structure
                structure["files"].append({
                    "name": file,
                    "path": file_rel_path,
                    "extension": ext,
                    "size": os.path.getsize(file_path)
                })
            
            # Process directories at current level
            if root == project_path:
                for d in dirs:
                    dir_path = os.path.join(root, d)
                    structure["directories"].append({
                        "name": d,
                        "path": d
                    })
        
        return structure
    except Exception as e:
        return {"error": str(e)}

# Tool: Generate documentation
@server.tool(
    id="code.documentation.generate",
    description="Generates documentation for a Python file",
    parameters=[
        {
            "name": "file_path", 
            "type": "string", 
            "description": "Path to the Python file"
        },
        {
            "name": "format", 
            "type": "string", 
            "description": "Documentation format",
            "enum": ["markdown", "restructuredtext", "numpy", "google"],
            "default": "markdown"
        }
    ]
)
async def generate_documentation(file_path, format="markdown"):
    try:
        # Parse Python file structure
        structure = await parse_python_structure(file_path)
        
        if "error" in structure:
            return {"error": structure["error"]}
        
        # Generate documentation
        if format == "markdown":
            return generate_markdown_documentation(structure)
        elif format == "restructuredtext":
            return generate_rst_documentation(structure)
        elif format == "numpy":
            return generate_numpy_documentation(structure)
        elif format == "google":
            return generate_google_documentation(structure)
        else:
            return {"error": f"Unsupported format: {format}"}
    except Exception as e:
        return {"error": str(e)}

# Helper function for markdown documentation
def generate_markdown_documentation(structure):
    # Generate markdown documentation
    doc = f"# {structure['file']}\n\n"
    
    # Add imports section
    if structure["imports"]:
        doc += "## Imports\n\n"
        for imp in structure["imports"]:
            if imp["alias"]:
                doc += f"- `{imp['name']}` as `{imp['alias']}`\n"
            else:
                doc += f"- `{imp['name']}`\n"
        doc += "\n"
    
    # Add classes section
    if structure["classes"]:
        doc += "## Classes\n\n"
        for cls in structure["classes"]:
            doc += f"### {cls['name']}\n\n"
            
            if cls["docstring"]:
                doc += f"{cls['docstring']}\n\n"
            
            if cls["bases"]:
                doc += f"**Inherits from:** {', '.join(cls['bases'])}\n\n"
            
            if cls["variables"]:
                doc += "#### Class Variables\n\n"
                for var in cls["variables"]:
                    doc += f"- `{var['name']}`\n"
                doc += "\n"
            
            if cls["methods"]:
                doc += "#### Methods\n\n"
                for method in cls["methods"]:
                    args_str = ", ".join(method["args"][1:])  # Skip 'self'
                    doc += f"- `{method['name']}({args_str})`"
                    if method["docstring"]:
                        doc += f": {method['docstring'].split('.')[0]}."
                    doc += "\n"
                doc += "\n"
    
    # Add functions section
    if structure["functions"]:
        doc += "## Functions\n\n"
        for func in structure["functions"]:
            args_str = ", ".join(func["args"])
            doc += f"### `{func['name']}({args_str})`\n\n"
            
            if func["docstring"]:
                doc += f"{func['docstring']}\n\n"
    
    return {"documentation": doc}

# Helper functions for other documentation formats (simplified)
def generate_rst_documentation(structure):
    # Simplified RST documentation
    return {"documentation": "RST documentation not fully implemented yet"}

def generate_numpy_documentation(structure):
    # Simplified NumPy documentation
    return {"documentation": "NumPy documentation not fully implemented yet"}

def generate_google_documentation(structure):
    # Simplified Google documentation
    return {"documentation": "Google documentation not fully implemented yet"}

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"IDE MCP server running on port {port}")
```

## VS Code Extension Example

Here's a simplified example of a VS Code extension that connects to your MCP server:

```typescript
// VS Code extension for MCP integration
import * as vscode from 'vscode';
import * as path from 'path';
import fetch from 'node-fetch';

// Activate extension
export function activate(context: vscode.ExtensionContext) {
  console.log('MCP IDE extension is now active');

  // Register commands
  let analyzeFileCommand = vscode.commands.registerCommand('mcp-ide.analyzeFile', analyzeCurrentFile);
  let generateDocCommand = vscode.commands.registerCommand('mcp-ide.generateDocumentation', generateDocumentation);
  let analyzeProjectCommand = vscode.commands.registerCommand('mcp-ide.analyzeProject', analyzeProject);

  // Add commands to context
  context.subscriptions.push(analyzeFileCommand);
  context.subscriptions.push(generateDocCommand);
  context.subscriptions.push(analyzeProjectCommand);

  // Create status bar item
  const statusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right, 100);
  statusBarItem.text = "$(rocket) MCP";
  statusBarItem.tooltip = "MCP IDE Tools";
  statusBarItem.command = "mcp-ide.showCommands";
  statusBarItem.show();

  // Register show commands menu
  let showCommandsMenu = vscode.commands.registerCommand('mcp-ide.showCommands', () => {
    vscode.window.showQuickPick([
      { label: "Analyze Current File", description: "Analyze structure of current file" },
      { label: "Generate Documentation", description: "Generate documentation for current file" },
      { label: "Analyze Project", description: "Analyze structure of current project" }
    ]).then(selection => {
      if (!selection) return;

      if (selection.label === "Analyze Current File") {
        vscode.commands.executeCommand('mcp-ide.analyzeFile');
      } else if (selection.label === "Generate Documentation") {
        vscode.commands.executeCommand('mcp-ide.generateDocumentation');
      } else if (selection.label === "Analyze Project") {
        vscode.commands.executeCommand('mcp-ide.analyzeProject');
      }
    });
  });

  context.subscriptions.push(showCommandsMenu);
  context.subscriptions.push(statusBarItem);
}

// Deactivate extension
export function deactivate() {}

// Analyze current file
async function analyzeCurrentFile() {
  try {
    const editor = vscode.window.activeTextEditor;
    if (!editor) {
      vscode.window.showInformationMessage('No active editor');
      return;
    }

    const filePath = editor.document.fileName;
    const fileExt = path.extname(filePath);

    if (fileExt !== '.py') {
      vscode.window.showInformationMessage('Only Python files are supported for now');
      return;
    }

    vscode.window.withProgress({
      location: vscode.ProgressLocation.Notification,
      title: "Analyzing file...",
      cancellable: false
    }, async (progress) => {
      progress.report({ increment: 0 });

      try {
        const result = await callMcpTool('code.structure.python', { file_path: filePath });
        progress.report({ increment: 100 });

        if (result.error) {
          vscode.window.showErrorMessage(`Analysis failed: ${result.error}`);
          return;
        }

        // Show results in webview
        const panel = vscode.window.createWebviewPanel(
          'fileStructure',
          `Structure: ${path.basename(filePath)}`,
          vscode.ViewColumn.Beside,
          { enableScripts: true }
        );

        panel.webview.html = generateStructureHtml(result);
      } catch (error) {
        vscode.window.showErrorMessage(`Failed to analyze file: ${error.message}`);
      }
    });
  } catch (error) {
    vscode.window.showErrorMessage(`Error: ${error.message}`);
  }
}

// Generate documentation
async function generateDocumentation() {
  try {
    const editor = vscode.window.activeTextEditor;
    if (!editor) {
      vscode.window.showInformationMessage('No active editor');
      return;
    }

    const filePath = editor.document.fileName;
    const fileExt = path.extname(filePath);

    if (fileExt !== '.py') {
      vscode.window.showInformationMessage('Only Python files are supported for now');
      return;
    }

    // Ask for documentation format
    const format = await vscode.window.showQuickPick(
      ['markdown', 'restructuredtext', 'numpy', 'google'],
      { placeHolder: 'Select documentation format' }
    );

    if (!format) return;

    vscode.window.withProgress({
      location: vscode.ProgressLocation.Notification,
      title: "Generating documentation...",
      cancellable: false
    }, async (progress) => {
      progress.report({ increment: 0 });

      try {
        const result = await callMcpTool('code.documentation.generate', {
          file_path: filePath,
          format
        });
        progress.report({ increment: 100 });

        if (result.error) {
          vscode.window.showErrorMessage(`Documentation generation failed: ${result.error}`);
          return;
        }

        // Show documentation
        const doc = await vscode.workspace.openTextDocument({
          content: result.documentation,
          language: format === 'markdown' ? 'markdown' : 'text'
        });
        await vscode.window.showTextDocument(doc, vscode.ViewColumn.Beside);
      } catch (error) {
        vscode.window.showErrorMessage(`Failed to generate documentation: ${error.message}`);
      }
    });
  } catch (error) {
    vscode.window.showErrorMessage(`Error: ${error.message}`);
  }
}

// Analyze project structure
async function analyzeProject() {
  try {
    const workspaceFolders = vscode.workspace.workspaceFolders;
    if (!workspaceFolders || workspaceFolders.length === 0) {
      vscode.window.showInformationMessage('No workspace folder open');
      return;
    }

    const projectPath = workspaceFolders[0].uri.fsPath;

    vscode.window.withProgress({
      location: vscode.ProgressLocation.Notification,
      title: "Analyzing project...",
      cancellable: false
    }, async (progress) => {
      progress.report({ increment: 0 });

      try {
        const result = await callMcpTool('project.structure', { project_path: projectPath });
        progress.report({ increment: 100 });

        if (result.error) {
          vscode.window.showErrorMessage(`Project analysis failed: ${result.error}`);
          return;
        }

        // Show results in webview
        const panel = vscode.window.createWebviewPanel(
          'projectStructure',
          `Project Structure: ${result.name}`,
          vscode.ViewColumn.Active,
          { enableScripts: true }
        );

        panel.webview.html = generateProjectStructureHtml(result);
      } catch (error) {
        vscode.window.showErrorMessage(`Failed to analyze project: ${error.message}`);
      }
    });
  } catch (error) {
    vscode.window.showErrorMessage(`Error: ${error.message}`);
  }
}

// Call MCP tool
async function callMcpTool(toolId: string, params: any) {
  try {
    const response = await fetch('http://localhost:8000/mcp/invoke', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        tool_id: toolId,
        parameters: params
      })
    });

    if (!response.ok) {
      throw new Error(`HTTP error ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error('Error calling MCP tool:', error);
    throw error;
  }
}

// Generate HTML for structure display
function generateStructureHtml(structure: any): string {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>File Structure</title>
      <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        h1 { color: #333; }
        .section { margin: 20px 0; }
        .item { margin: 10px 0; }
        .name { font-weight: bold; }
        .line { color: #666; }
        .docstring { color: #008800; margin-left: 20px; }
      </style>
    </head>
    <body>
      <h1>Structure: ${structure.file}</h1>
      
      <div class="section">
        <h2>Imports (${structure.imports.length})</h2>
        ${structure.imports.map(imp => `
          <div class="item">
            <span class="name">${imp.name}</span>
            ${imp.alias ? `as <span class="alias">${imp.alias}</span>` : ''}
            <span class="line">- Line ${imp.line}</span>
          </div>
        `).join('')}
      </div>
      
      <div class="section">
        <h2>Classes (${structure.classes.length})</h2>
        ${structure.classes.map(cls => `
          <div class="item">
            <div class="name">${cls.name}</div>
            <div class="line">Line ${cls.line}</div>
            ${cls.docstring ? `<div class="docstring">${cls.docstring}</div>` : ''}
            
            <div style="margin-left: 20px;">
              <h4>Methods (${cls.methods.length})</h4>
              ${cls.methods.map(method => `
                <div class="item">
                  <span class="name">${method.name}</span>
                  <span class="params">(${method.args.join(', ')})</span>
                  <span class="line">- Line ${method.line}</span>
                  ${method.docstring ? `<div class="docstring">${method.docstring}</div>` : ''}
                </div>
              `).join('')}
            </div>
          </div>
        `).join('')}
      </div>
      
      <div class="section">
        <h2>Functions (${structure.functions.length})</h2>
        ${structure.functions.map(func => `
          <div class="item">
            <span class="name">${func.name}</span>
            <span class="params">(${func.args.join(', ')})</span>
            <span class="line">- Line ${func.line}</span>
            ${func.docstring ? `<div class="docstring">${func.docstring}</div>` : ''}
          </div>
        `).join('')}
      </div>
    </body>
    </html>
  `;
}

// Generate HTML for project structure display
function generateProjectStructureHtml(structure: any): string {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Project Structure</title>
      <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        h1 { color: #333; }
        .section { margin: 20px 0; }
        .chart { height: 300px; margin: 20px 0; }
        .file-list { max-height: 400px; overflow-y: auto; border: 1px solid #ddd; padding: 10px; }
        .file-item { margin: 5px 0; }
      </style>
      <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    </head>
    <body>
      <h1>Project Structure: ${structure.name}</h1>
      
      <div class="section">
        <h2>Summary</h2>
        <p>Total Files: ${structure.summary.total_files}</p>
        
        <h3>Languages</h3>
        <div class="chart">
          <canvas id="languagesChart"></canvas>
        </div>
        
        <h3>File Types</h3>
        <div class="chart">
          <canvas id="fileTypesChart"></canvas>
        </div>
      </div>
      
      <div class="section">
        <h2>Files (${structure.files.length})</h2>
        <div class="file-list">
          ${structure.files.map(file => `
            <div class="file-item">
              <span class="file-path">${file.path}</span>
              <span class="file-size">(${formatFileSize(file.size)})</span>
            </div>
          `).join('')}
        </div>
      </div>
      
      <script>
        // Create language chart
        const languagesCtx = document.getElementById('languagesChart').getContext('2d');
        new Chart(languagesCtx, {
          type: 'pie',
          data: {
            labels: ${JSON.stringify(Object.keys(structure.summary.languages))},
            datasets: [{
              data: ${JSON.stringify(Object.values(structure.summary.languages))},
              backgroundColor: [
                '#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0', '#9966FF',
                '#FF9F40', '#C9CBCF', '#7BC043', '#F37736', '#EEC900'
              ]
            }]
          },
          options: {
            responsive: true,
            plugins: {
              legend: {
                position: 'right',
              },
              title: {
                display: true,
                text: 'Languages'
              }
            }
          }
        });
        
        // Create file types chart
        const fileTypesCtx = document.getElementById('fileTypesChart').getContext('2d');
        new Chart(fileTypesCtx, {
          type: 'bar',
          data: {
            labels: ${JSON.stringify(Object.keys(structure.summary.file_types))},
            datasets: [{
              label: 'Number of Files',
              data: ${JSON.stringify(Object.values(structure.summary.file_types))},
              backgroundColor: '#36A2EB'
            }]
          },
          options: {
            responsive: true,
            plugins: {
              legend: {
                display: false
              },
              title: {
                display: true,
                text: 'File Types'
              }
            }
          }
        });
        
        // Format file size
        function formatFileSize(bytes) {
          if (bytes < 1024) return bytes + ' B';
          if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
          return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
        }
      </script>
    </body>
    </html>
  `;
}
```

## Extension Activities

1. **Advanced IDE Integration**:
   - Add code completion suggestions based on repository analysis
   - Implement automatic documentation generation on file save
   - Create visualization tools for code dependencies
   - Add code quality assessment features

2. **Multi-Language Support**:
   - Extend the code structure analysis to other languages (JavaScript, Java, etc.)
   - Implement language-specific documentation generators
   - Add cross-language reference tracking
   - Create language-agnostic code navigation features

3. **Repository-Level Analysis**:
   - Implement code similarity detection
   - Add architectural pattern recognition
   - Create contributor activity visualization
   - Add code health metrics and trend analysis

## Recommended Reading

1. [MCP for IDE Integration Guide](https://modelcontextprotocol.io/docs/guides/ide-integration/)
   - Architecture patterns for IDE integration
   - Best practices for code analysis
   - Performance optimization techniques
   - User experience considerations

2. [Code Analysis Patterns](https://modelcontextprotocol.io/docs/patterns/code-analysis/)
   - Abstract syntax tree (AST) analysis techniques
   - Symbol resolution strategies
   - Type inference approaches
   - Cross-file reference tracking

3. [Documentation Generation](https://modelcontextprotocol.io/docs/guides/documentation-generation/)
   - Documentation standards and formats
   - Automatic comment generation
   - API documentation strategies
   - Documentation quality assessment

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What are the main components needed for MCP integration with an IDE?
2. How can MCP enhance code completion capabilities?
3. What types of code analysis can be performed using MCP?
4. How can repository-level context improve coding assistance?
5. What security considerations are important for IDE integration?
6. What types of documentation can be automatically generated?
7. How can MCP improve code navigation in large projects?
8. What are the challenges in providing cross-language support?

## IDE Integration Analysis

Analyze existing IDE extensions that use AI for coding assistance:

1. **Feature Comparison**:
   - Make a list of features provided by popular AI coding assistants
   - Identify strengths and limitations of each
   - Determine which features could be enhanced with MCP
   - Design new features that would be possible with MCP

2. **Architecture Analysis**:
   - Examine how existing extensions integrate with AI models
   - Identify pain points in current architectures
   - Design an improved architecture using MCP
   - Create a migration path for existing extensions

3. **User Experience Evaluation**:
   - Evaluate the user experience of current AI coding assistants
   - Identify friction points and limitations
   - Design an improved user experience with MCP
   - Create mockups of the new user interface

## Additional Resources

### Development Tools
- [MCP IDE Extension Template](https://modelcontextprotocol.io/tools/ide-extension-template/)
- [Code Analysis Libraries](https://modelcontextprotocol.io/docs/resources/code-analysis-libraries/)
- [Documentation Generation Tools](https://modelcontextprotocol.io/docs/resources/documentation-tools/)

### Integration Examples
- [VS Code MCP Extension](https://modelcontextprotocol.io/examples/vscode-extension/)
- [JetBrains MCP Plugin](https://modelcontextprotocol.io/examples/jetbrains-plugin/)
- [Eclipse MCP Integration](https://modelcontextprotocol.io/examples/eclipse-integration/)

### Advanced Topics
- [AST Analysis Techniques](https://modelcontextprotocol.io/docs/topics/ast-analysis/)
- [Code Generation Best Practices](https://modelcontextprotocol.io/docs/topics/code-generation/)
- [Repository Mining with MCP](https://modelcontextprotocol.io/docs/topics/repository-mining/)
