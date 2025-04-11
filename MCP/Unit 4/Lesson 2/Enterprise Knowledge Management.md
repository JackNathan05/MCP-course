
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 2/_Lesson 2|_Lesson 2]]

---

## Overview

This lesson explores how MCP can connect AI assistants to enterprise data sources securely and effectively.

## Connecting AI Assistants to Company Data

Enterprise data integration challenges:

1. **Data silos and formats**
   - Multiple data repositories
   - Diverse file formats
   - Structured and unstructured data
   - Legacy systems

2. **Access control and permissions**
   - User-specific access rights
   - Role-based permissions
   - Department-specific restrictions
   - Document-level security

3. **Sensitive information handling**
   - PII (Personally Identifiable Information)
   - Financial data
   - Intellectual property
   - Confidential communications

4. **Integration with existing systems**
   - CRM systems
   - Document management
   - ERP systems
   - Internal wikis and knowledge bases

5. **Compliance requirements**
   - Data protection regulations
   - Industry-specific compliance
   - Audit requirements
   - Cross-border data restrictions

MCP provides a standardized way to address these challenges by creating secure connectors to enterprise data sources while maintaining appropriate access controls and compliance.

## Document Repositories and Search Integration

Effective document access strategies:

1. **Document indexing and retrieval**
   - Full-text indexing
   - Metadata extraction
   - Entity recognition
   - Relevance ranking

2. **Content extraction and parsing**
   - PDF parsing
   - Office document processing
   - Email content extraction
   - Image and diagram OCR

3. **Metadata handling**
   - Author information
   - Creation and modification dates
   - Document categories
   - Custom metadata fields

4. **Version management**
   - Version history tracking
   - Change detection
   - Collaborative editing
   - Draft management

5. **Search optimization**
   - Query understanding
   - Semantic search
   - Faceted search
   - Personalized ranking

These capabilities allow AI assistants to find and use the most relevant documents for user queries.

## Example: Document Repository MCP Server

Here's an example of a document repository MCP server:

```python
# Document repository MCP server
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter, MCPResource

import os
import json
import datetime
import logging
from typing import List, Dict, Any, Optional

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Document database (simplified in-memory version)
class DocumentDB:
    def __init__(self, documents_path):
        self.documents_path = documents_path
        self.documents = []
        self.load_documents()
    
    def load_documents(self):
        """Load documents from the specified directory"""
        try:
            if not os.path.isdir(self.documents_path):
                logger.error(f"Documents path does not exist: {self.documents_path}")
                return
            
            # Walk through directory
            for root, _, files in os.walk(self.documents_path):
                for file in files:
                    if file.endswith(('.txt', '.md', '.json', '.csv', '.html', '.xml', '.pdf')):
                        file_path = os.path.join(root, file)
                        # Get relative path
                        rel_path = os.path.relpath(file_path, self.documents_path)
                        
                        # Extract file information
                        stat = os.stat(file_path)
                        created = datetime.datetime.fromtimestamp(stat.st_ctime)
                        modified = datetime.datetime.fromtimestamp(stat.st_mtime)
                        size = stat.st_size
                        
                        # Extract content (simplified)
                        content = self._extract_content(file_path)
                        
                        # Create document record
                        document = {
                            "id": rel_path,
                            "title": os.path.splitext(file)[0],
                            "path": rel_path,
                            "type": os.path.splitext(file)[1][1:],
                            "size": size,
                            "created": created.isoformat(),
                            "modified": modified.isoformat(),
                            "content": content[:10000],  # Limit content size
                            "indexed": datetime.datetime.now().isoformat()
                        }
                        
                        self.documents.append(document)
            
            logger.info(f"Loaded {len(self.documents)} documents")
            
        except Exception as e:
            logger.error(f"Error loading documents: {str(e)}")
    
    def _extract_content(self, file_path):
        """Extract content from a file (simplified version)"""
        try:
            # For simplicity, just read as text
            # In a real implementation, this would handle different file types appropriately
            with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                return f.read()
        except Exception as e:
            logger.error(f"Error extracting content from {file_path}: {str(e)}")
            return ""
    
    def search(self, query, filters=None, max_results=10):
        """Search for documents matching the query"""
        if not query:
            return []
        
        query = query.lower()
        results = []
        
        for doc in self.documents:
            # Simple search in title and content
            if query in doc["title"].lower() or query in doc["content"].lower():
                # Check filters if provided
                if filters and not self._check_filters(doc, filters):
                    continue
                
                # Add to results with snippet
                snippet = self._create_snippet(doc["content"], query)
                result = {
                    "id": doc["id"],
                    "title": doc["title"],
                    "path": doc["path"],
                    "type": doc["type"],
                    "created": doc["created"],
                    "modified": doc["modified"],
                    "snippet": snippet
                }
                results.append(result)
                
                # Limit results
                if len(results) >= max_results:
                    break
        
        return results
    
    def _check_filters(self, doc, filters):
        """Check if document matches filters"""
        for key, value in filters.items():
            if key not in doc:
                return False
            
            if isinstance(value, list):
                if doc[key] not in value:
                    return False
            else:
                if doc[key] != value:
                    return False
        
        return True
    
    def _create_snippet(self, content, query, context_size=100):
        """Create a snippet around the query"""
        index = content.lower().find(query)
        if index == -1:
            return content[:200] + "..."
        
        start = max(0, index - context_size)
        end = min(len(content), index + len(query) + context_size)
        
        snippet = content[start:end]
        if start > 0:
            snippet = "..." + snippet
        if end < len(content):
            snippet = snippet + "..."
        
        return snippet
    
    def get_document(self, doc_id):
        """Get a document by ID"""
        for doc in self.documents:
            if doc["id"] == doc_id:
                return doc
        return None

# Initialize document database
documents_path = os.environ.get("DOCUMENTS_PATH", "./documents")
doc_db = DocumentDB(documents_path)

# Create MCP server
server = MCPServer()

# Register search tool
@server.tool(
    id="documents.search",
    description="Search for documents in the repository",
    parameters=[
        MCPParameter(
            name="query",
            description="Search query",
            type="string",
            required=True
        ),
        MCPParameter(
            name="document_types",
            description="Filter by document types (e.g., pdf, txt)",
            type="array",
            items={"type": "string"},
            required=False
        ),
        MCPParameter(
            name="max_results",
            description="Maximum number of results",
            type="integer",
            minimum=1,
            maximum=50,
            default=10,
            required=False
        )
    ]
)
async def search_documents(query, document_types=None, max_results=10):
    """Search for documents in the repository"""
    try:
        filters = {}
        if document_types:
            filters["type"] = document_types
        
        results = doc_db.search(query, filters, max_results)
        
        return {
            "query": query,
            "total_results": len(results),
            "results": results
        }
    except Exception as e:
        logger.error(f"Error searching documents: {str(e)}")
        raise Exception(f"Search failed: {str(e)}")

# Register document retrieval tool
@server.tool(
    id="documents.get",
    description="Get a document by ID",
    parameters=[
        MCPParameter(
            name="document_id",
            description="Document ID",
            type="string",
            required=True
        )
    ]
)
async def get_document(document_id):
    """Get a document by ID"""
    try:
        document = doc_db.get_document(document_id)
        
        if not document:
            raise Exception(f"Document not found: {document_id}")
        
        return {
            "id": document["id"],
            "title": document["title"],
            "path": document["path"],
            "type": document["type"],
            "created": document["created"],
            "modified": document["modified"],
            "content": document["content"]
        }
    except Exception as e:
        logger.error(f"Error getting document: {str(e)}")
        raise Exception(f"Failed to get document: {str(e)}")

# Register document resource
@server.resource(
    id="documents.content",
    description="Get the content of a document",
    parameters=[
        MCPParameter(
            name="document_id",
            description="Document ID",
            type="string",
            required=True
        )
    ]
)
async def document_content_resource(document_id):
    """Get the content of a document"""
    try:
        document = doc_db.get_document(document_id)
        
        if not document:
            raise Exception(f"Document not found: {document_id}")
        
        return {
            "title": document["title"],
            "content": document["content"],
            "type": document["type"]
        }
    except Exception as e:
        logger.error(f"Error getting document content: {str(e)}")
        raise Exception(f"Failed to get document content: {str(e)}")

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Document MCP server running on port {port}")
```

## Security and Access Control in Enterprise Settings

Enterprise security requirements:

1. **Role-based access control**
   - User roles and permissions
   - Group-based access
   - Hierarchical permissions
   - Dynamic role assignment

2. **Data classification**
   - Sensitivity levels
   - Classification tags
   - Handling requirements
   - Retention policies

3. **Audit trails**
   - Access logging
   - Query tracking
   - Data usage monitoring
   - Compliance reporting

4. **Data loss prevention**
   - Content filtering
   - Exfiltration protection
   - Watermarking
   - Copy protection

5. **Secure communication**
   - Transport encryption
   - Message signing
   - Session management
   - Key rotation

Example of implementing role-based access control in an MCP server:

```python
# Role-based access control for MCP server
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter

import os
import jwt
import datetime
from functools import wraps

# Simple user database (in production, this would be a real database)
users = {
    "alice": {
        "id": "user-123",
        "name": "Alice Smith",
        "email": "alice@example.com",
        "password": "hashed_password_here",
        "roles": ["admin", "editor"]
    },
    "bob": {
        "id": "user-456",
        "name": "Bob Johnson",
        "email": "bob@example.com",
        "password": "hashed_password_here",
        "roles": ["viewer"]
    }
}

# Role permissions
role_permissions = {
    "admin": ["documents.search", "documents.get", "documents.create", "documents.update", "documents.delete"],
    "editor": ["documents.search", "documents.get", "documents.update"],
    "viewer": ["documents.search", "documents.get"]
}

# JWT secret key
JWT_SECRET = os.environ.get("JWT_SECRET", "your_secret_key_here")

# Create MCP server
server = MCPServer()

# Authentication tool
@server.tool(
    id="auth.login",
    description="Log in to the system",
    parameters=[
        MCPParameter(
            name="username",
            description="Username",
            type="string",
            required=True
        ),
        MCPParameter(
            name="password",
            description="Password",
            type="string",
            required=True
        )
    ]
)
async def login(username, password):
    """Authenticate user and return JWT token"""
    # In production, use proper password hashing and validation
    user = users.get(username)
    
    if not user:
        raise Exception("Invalid username or password")
    
    # Create JWT token
    expiration = datetime.datetime.utcnow() + datetime.timedelta(hours=24)
    payload = {
        "sub": user["id"],
        "name": user["name"],
        "email": user["email"],
        "roles": user["roles"],
        "exp": expiration
    }
    
    token = jwt.encode(payload, JWT_SECRET, algorithm="HS256")
    
    return {
        "token": token,
        "user": {
            "id": user["id"],
            "name": user["name"],
            "email": user["email"],
            "roles": user["roles"]
        }
    }

# Decorator for authorization
def require_auth(*required_roles):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Get context from MCP server
            context = server.get_current_context()
            
            # Check for authorization header
            auth_header = context.request.headers.get("Authorization")
            if not auth_header or not auth_header.startswith("Bearer "):
                raise Exception("Authentication required")
            
            # Extract token
            token = auth_header[7:]
            
            try:
                # Verify token
                payload = jwt.decode(token, JWT_SECRET, algorithms=["HS256"])
                
                # Check roles if required
                if required_roles:
                    user_roles = payload.get("roles", [])
                    if not any(role in user_roles for role in required_roles):
                        raise Exception("Insufficient permissions")
                
                # Add user to context
                context.user = payload
                
            except jwt.ExpiredSignatureError:
                raise Exception("Token expired")
            except jwt.InvalidTokenError:
                raise Exception("Invalid token")
            
            # Call original function
            return await func(*args, **kwargs)
        return wrapper
    return decorator

# Document search tool with authorization
@server.tool(
    id="documents.search",
    description="Search for documents",
    parameters=[
        MCPParameter(
            name="query",
            description="Search query",
            type="string",
            required=True
        ),
        MCPParameter(
            name="max_results",
            description="Maximum number of results",
            type="integer",
            minimum=1,
            maximum=50,
            default=10,
            required=False
        )
    ]
)
@require_auth("admin", "editor", "viewer")
async def search_documents(query, max_results=10):
    """Search for documents with authorization"""
    # Get user from context
    context = server.get_current_context()
    user = context.user
    
    # Log access for audit
    print(f"User {user['name']} ({user['email']}) searched for: {query}")
    
    # Perform search
    # In a real implementation, this would filter results based on user's permissions
    
    return {
        "query": query,
        "results": [
            {"id": "doc-1", "title": "Document 1", "snippet": "This is a sample document..."},
            {"id": "doc-2", "title": "Document 2", "snippet": "Another example document..."}
        ]
    }

# Document retrieval tool with authorization
@server.tool(
    id="documents.get",
    description="Get a document by ID",
    parameters=[
        MCPParameter(
            name="document_id",
            description="Document ID",
            type="string",
            required=True
        )
    ]
)
@require_auth("admin", "editor", "viewer")
async def get_document(document_id):
    """Get a document with authorization"""
    # Get user from context
    context = server.get_current_context()
    user = context.user
    
    # Log access for audit
    print(f"User {user['name']} ({user['email']}) accessed document: {document_id}")
    
    # Check specific document permissions (in a real system)
    # if not has_document_permission(user, document_id):
    #     raise Exception("Access denied for this document")
    
    # Return document
    return {
        "id": document_id,
        "title": "Sample Document",
        "content": "This is the content of the document."
    }

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Secure document MCP server running on port {port}")
```

## Compliance Considerations

Regulatory and compliance aspects:

1. **Data privacy regulations**
   - GDPR (General Data Protection Regulation)
   - CCPA (California Consumer Privacy Act)
   - HIPAA (Health Insurance Portability and Accountability Act)
   - Local data protection laws

2. **Industry-specific regulations**
   - FINRA (Financial Industry Regulatory Authority)
   - FDA (Food and Drug Administration)
   - FERPA (Family Educational Rights and Privacy Act)
   - PCI DSS (Payment Card Industry Data Security Standard)

3. **Data retention policies**
   - Retention schedules
   - Legal holds
   - Archiving requirements
   - Disposal procedures

4. **Information governance**
   - Data stewardship
   - Classification systems
   - Usage policies
   - Cross-border transfers

5. **Audit requirements**
   - Access logging
   - Change tracking
   - Compliance reporting
   - Evidence collection

MCP can help address these requirements by providing:

- Standardized access controls and authentication
- Consistent logging and auditing
- Centralized policy enforcement
- Transparent data access patterns

## Implementing Compliance in MCP Servers

Here's an example of implementing compliance logging in an MCP server:

```python
# Compliance logging for MCP server
import json
import time
import uuid
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("compliance.log"),
        logging.StreamHandler()
    ]
)

# Create compliance logger
class ComplianceLogger:
    def __init__(self):
        self.logger = logging.getLogger("compliance")
    
    def log_access(self, user_id, action, resource, success, details=None):
        """Log access to resources"""
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_id": str(uuid.uuid4()),
            "event_type": "resource_access",
            "user_id": user_id,
            "action": action,
            "resource": resource,
            "success": success,
            "details": details or {}
        }
        
        self.logger.info(json.dumps(log_entry))
        return log_entry
    
    def log_authentication(self, user_id, success, method, ip_address, details=None):
        """Log authentication events"""
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_id": str(uuid.uuid4()),
            "event_type": "authentication",
            "user_id": user_id,
            "success": success,
            "method": method,
            "ip_address": ip_address,
            "details": details or {}
        }
        
        self.logger.info(json.dumps(log_entry))
        return log_entry
    
    def log_data_export(self, user_id, resource, format, records_count, details=None):
        """Log data export events"""
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_id": str(uuid.uuid4()),
            "event_type": "data_export",
            "user_id": user_id,
            "resource": resource,
            "format": format,
            "records_count": records_count,
            "details": details or {}
        }
        
        self.logger.info(json.dumps(log_entry))
        return log_entry
    
    def log_administration(self, user_id, action, target, details=None):
        """Log administrative actions"""
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_id": str(uuid.uuid4()),
            "event_type": "administration",
            "user_id": user_id,
            "action": action,
            "target": target,
            "details": details or {}
        }
        
        self.logger.info(json.dumps(log_entry))
        return log_entry

# Create compliance logger instance
compliance_logger = ComplianceLogger()

# Example usage in an MCP server tool
@server.tool(
    id="documents.search",
    description="Search for documents",
    parameters=[
        MCPParameter(
            name="query",
            description="Search query",
            type="string",
            required=True
        )
    ]
)
@require_auth("viewer", "editor", "admin")
async def search_documents(query):
    # Get user from context
    context = server.get_current_context()
    user = context.user
    
    try:
        # Perform search
        results = document_db.search(query)
        
        # Log successful access
        compliance_logger.log_access(
            user_id=user["id"],
            action="search",
            resource="documents",
            success=True,
            details={
                "query": query,
                "results_count": len(results)
            }
        )
        
        return {
            "query": query,
            "results": results
        }
    except Exception as e:
        # Log failed access
        compliance_logger.log_access(
            user_id=user["id"],
            action="search",
            resource="documents",
            success=False,
            details={
                "query": query,
                "error": str(e)
            }
        )
        
        raise e
```

## Enterprise Integration Best Practices

When implementing MCP for enterprise knowledge management, follow these best practices:

1. **Layered Security Approach**
   - Network security
   - Application security
   - Data security
   - User authentication and authorization

2. **Privacy by Design**
   - Data minimization
   - Purpose limitation
   - Access restrictions
   - Anonymization when possible

3. **Compliance Documentation**
   - Security controls mapping
   - Compliance matrix
   - Policy enforcement
   - Regular auditing

4. **Integration with Identity Providers**
   - Single Sign-On (SSO)
   - Multi-factor authentication
   - Directory services
   - Federated identity

5. **Performance at Scale**
   - Caching strategies
   - Query optimization
   - Load balancing
   - Resource management

6. **Resilience and Disaster Recovery**
   - High availability
   - Data backup
   - Failover mechanisms
   - Business continuity planning

7. **Monitoring and Alerting**
   - Performance monitoring
   - Security monitoring
   - Compliance monitoring
   - Anomaly detection

## Summary

MCP enables secure and compliant enterprise knowledge management by providing standardized ways to connect AI assistants to company data sources. By implementing proper access controls, compliance logging, and security measures, organizations can leverage AI capabilities while maintaining governance and security requirements.

In the next lesson, we'll explore how MCP can be used to integrate with productivity applications like email, calendar, and task management systems.
