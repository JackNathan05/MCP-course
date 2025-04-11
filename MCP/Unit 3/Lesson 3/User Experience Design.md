
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 3/Lesson 3/_Lesson 3|_Lesson 3]]

---

## Overview

This lesson focuses on designing effective user experiences for applications using MCP and AI models.

## Presenting MCP Capabilities to Users

Users need appropriate awareness of capabilities:

1. **Intuitive UI for available tools**
   - Clear categorization
   - Visual hierarchy
   - Consistent presentation
   - Easily discoverable features

2. **Clear descriptions of functionality**
   - Plain language explanations
   - Purpose statements
   - Capability limitations
   - Access requirements

3. **Appropriate grouping and categorization**
   - Functional grouping
   - Use-case based categories
   - Progressive disclosure
   - Contextual organization

4. **Contextual suggestions**
   - Situational recommendations
   - Just-in-time suggestions
   - Usage pattern recognition
   - Adaptive suggestions

5. **Progressive disclosure**
   - Basic capabilities first
   - Advanced features on demand
   - Gradual complexity
   - Contextual help

Below is an example of how capabilities might be presented in a web interface:

```html
<!-- Example UI for capability presentation -->
<div class="capability-explorer">
  <h2>Available Capabilities</h2>
  
  <!-- Search and filters -->
  <div class="capability-filters">
    <input type="text" placeholder="Search capabilities..." />
    <div class="filter-buttons">
      <button class="active">All</button>
      <button>Tools</button>
      <button>Resources</button>
      <button>Prompts</button>
    </div>
    <div class="category-filters">
      <label><input type="checkbox" checked /> Information</label>
      <label><input type="checkbox" checked /> Communication</label>
      <label><input type="checkbox" checked /> Analysis</label>
      <label><input type="checkbox" checked /> Productivity</label>
    </div>
  </div>
  
  <!-- Capability list -->
  <div class="capability-list">
    <!-- Information category -->
    <div class="capability-category">
      <h3>Information</h3>
      
      <!-- Weather tool -->
      <div class="capability-card tool">
        <div class="capability-header">
          <h4>Weather Forecast</h4>
          <span class="capability-type">Tool</span>
        </div>
        <p class="capability-description">
          Get weather forecasts for any location worldwide
        </p>
        <div class="capability-details">
          <button class="details-toggle">Parameters</button>
          <div class="details-panel">
            <ul>
              <li><strong>location</strong> (required): City name or coordinates</li>
              <li><strong>days</strong> (optional): Number of forecast days (1-10)</li>
            </ul>
          </div>
        </div>
        <div class="capability-actions">
          <button class="try-button">Try it</button>
          <button class="info-button">More info</button>
        </div>
      </div>
      
      <!-- Search documents resource -->
      <div class="capability-card resource">
        <div class="capability-header">
          <h4>Document Search</h4>
          <span class="capability-type">Resource</span>
        </div>
        <p class="capability-description">
          Search within your documents and files
        </p>
        <div class="capability-details">
          <button class="details-toggle">Parameters</button>
          <div class="details-panel">
            <ul>
              <li><strong>query</strong> (required): Search keywords</li>
              <li><strong>file_types</strong> (optional): Filter by file type</li>
              <li><strong>date_range</strong> (optional): Filter by date range</li>
            </ul>
          </div>
        </div>
        <div class="capability-actions">
          <button class="try-button">Try it</button>
          <button class="info-button">More info</button>
        </div>
      </div>
    </div>
    
    <!-- More categories and capabilities -->
  </div>
</div>
```

## UI Patterns for Tool Invocation

Effective tool invocation UIs:

1. **Natural language requests**
   - Conversational interface
   - Command-like queries
   - Intent recognition
   - Contextual understanding

2. **Explicit tool selection interfaces**
   - Tool catalogs
   - Command palettes
   - Contextual menus
   - Quick access buttons

3. **Hybrid approaches**
   - Suggestions during typing
   - Auto-complete for commands
   - Visual aids with text input
   - Modal and non-modal options

4. **Context-aware suggestions**
   - Recently used tools
   - Relevant tools for current context
   - Personalized recommendations
   - Task-specific suggestions

5. **History and favorites**
   - Tool usage history
   - Favorite/pinned tools
   - Custom tool groups
   - Personalized shortcuts

Here's an example of a conversational UI with hybrid tool invocation:

```html
<!-- Example of conversational UI with tool suggestions -->
<div class="chat-interface">
  <!-- Chat history -->
  <div class="chat-history">
    <!-- User message -->
    <div class="chat-message user">
      <div class="message-avatar">U</div>
      <div class="message-content">
        <p>What's the weather like in San Francisco today?</p>
      </div>
      <div class="message-time">10:32 AM</div>
    </div>
    
    <!-- Assistant message with tool usage -->
    <div class="chat-message assistant">
      <div class="message-avatar">A</div>
      <div class="message-content">
        <div class="tool-usage">
          <div class="tool-header">
            <span class="tool-icon">🔍</span>
            <span class="tool-name">Weather Forecast</span>
          </div>
          <div class="tool-parameters">
            <span class="param-name">location:</span>
            <span class="param-value">San Francisco</span>
            <span class="param-name">days:</span>
            <span class="param-value">1</span>
          </div>
        </div>
        <p>It's currently 68°F and partly cloudy in San Francisco. There's a 10% chance of rain, and the high will be 72°F. Tonight will be clearer with a low of 56°F.</p>
      </div>
      <div class="message-time">10:32 AM</div>
    </div>
    
    <!-- User message -->
    <div class="chat-message user">
      <div class="message-avatar">U</div>
      <div class="message-content">
        <p>What about tomorrow?</p>
      </div>
      <div class="message-time">10:33 AM</div>
    </div>
    
    <!-- Assistant message -->
    <div class="chat-message assistant">
      <div class="message-avatar">A</div>
      <div class="message-content">
        <div class="tool-usage">
          <div class="tool-header">
            <span class="tool-icon">🔍</span>
            <span class="tool-name">Weather Forecast</span>
          </div>
          <div class="tool-parameters">
            <span class="param-name">location:</span>
            <span class="param-value">San Francisco</span>
            <span class="param-name">days:</span>
            <span class="param-value">2</span>
          </div>
        </div>
        <p>Tomorrow in San Francisco will be sunny with temperatures between 58°F and 74°F. No rain is expected, and winds will be light at 5-10 mph.</p>
      </div>
      <div class="message-time">10:33 AM</div>
    </div>
  </div>
  
  <!-- Input area with tool suggestions -->
  <div class="chat-input-area">
    <div class="tool-suggestions">
      <button class="tool-suggestion">
        <span class="tool-icon">📅</span>
        <span class="tool-name">Calendar Events</span>
      </button>
      <button class="tool-suggestion">
        <span class="tool-icon">📁</span>
        <span class="tool-name">Search Documents</span>
      </button>
      <button class="tool-suggestion">
        <span class="tool-icon">✓</span>
        <span class="tool-name">Task List</span>
      </button>
      <button class="tool-suggestion-more">+</button>
    </div>
    
    <div class="input-container">
      <textarea placeholder="Type a message or select a tool..."></textarea>
      <button class="send-button">Send</button>
    </div>
  </div>
</div>
```

## Displaying Tool Execution Results

Results should be presented effectively:

1. **Clear formatting for different result types**
   - Structured data visualization
   - Text formatting
   - Rich media display
   - Consistent layouts

2. **Visual differentiation from conversation**
   - Distinctive styling
   - Information cards
   - Result containers
   - Visual hierarchy

3. **Progress indicators during execution**
   - Loading states
   - Progress bars
   - Step indicators
   - Cancelation options

4. **Expandable/collapsible detailed results**
   - Summary view by default
   - Expandable details
   - Drill-down capabilities
   - Level-of-detail controls

5. **Integration with conversation flow**
   - Contextual presentation
   - In-line results
   - Reply-based navigation
   - Conversational references

Here's an example of displaying tool execution results:

```javascript
// React component for displaying tool results
import React, { useState } from 'react';

// Tool Result Card component
const ToolResultCard = ({ tool, result, loading, error }) => {
  const [expanded, setExpanded] = useState(false);
  
  // Define result renders based on tool type
  const renderWeatherResult = (result) => (
    <div className="weather-result">
      <div className="weather-header">
        <h3>{result.location.name}, {result.location.country}</h3>
        <div className="weather-date">{result.current.date}</div>
      </div>
      
      <div className="weather-current">
        <div className="weather-temp">
          {result.current.temp_c}°C / {result.current.temp_f}°F
        </div>
        <div className="weather-condition">
          <img src={result.current.condition.icon} alt={result.current.condition.text} />
          <span>{result.current.condition.text}</span>
        </div>
      </div>
      
      {expanded && (
        <div className="weather-details">
          <div className="weather-detail">
            <span className="detail-label">Humidity:</span>
            <span className="detail-value">{result.current.humidity}%</span>
          </div>
          <div className="weather-detail">
            <span className="detail-label">Wind:</span>
            <span className="detail-value">{result.current.wind_kph} km/h {result.current.wind_dir}</span>
          </div>
          <div className="weather-detail">
            <span className="detail-label">Pressure:</span>
            <span className="detail-value">{result.current.pressure_mb} mb</span>
          </div>
          <div className="weather-detail">
            <span className="detail-label">Precipitation:</span>
            <span className="detail-value">{result.current.precip_mm} mm</span>
          </div>
          
          <h4>Forecast</h4>
          <div className="weather-forecast">
            {result.forecast.forecastday.map(day => (
              <div key={day.date} className="forecast-day">
                <div className="forecast-date">{day.date}</div>
                <img src={day.day.condition.icon} alt={day.day.condition.text} />
                <div className="forecast-temp">
                  {day.day.mintemp_c}° - {day.day.maxtemp_c}°
                </div>
              </div>
            ))}
          </div>
        </div>
      )}
      
      <button 
        className="expand-button" 
        onClick={() => setExpanded(!expanded)}
      >
        {expanded ? 'Show Less' : 'Show More'}
      </button>
    </div>
  );
  
  const renderDocumentSearchResult = (result) => (
    <div className="document-search-result">
      <div className="search-summary">
        <span className="result-count">{result.total_results} results</span>
        <span className="query">for "{result.query}"</span>
      </div>
      
      <div className="search-results">
        {result.documents.slice(0, expanded ? result.documents.length : 3).map(doc => (
          <div key={doc.id} className="document-item">
            <div className="document-title">{doc.title}</div>
            <div className="document-snippet">{doc.snippet}</div>
            <div className="document-meta">
              <span className="document-type">{doc.type}</span>
              <span className="document-date">{doc.modified_date}</span>
            </div>
          </div>
        ))}
      </div>
      
      {result.documents.length > 3 && (
        <button 
          className="expand-button" 
          onClick={() => setExpanded(!expanded)}
        >
          {expanded ? 'Show Less' : `Show ${result.documents.length - 3} More Results`}
        </button>
      )}
    </div>
  );
  
  // Select renderer based on tool ID
  const renderResult = () => {
    if (loading) {
      return (
        <div className="loading-state">
          <div className="spinner"></div>
          <div className="loading-message">Running {tool.name}...</div>
        </div>
      );
    }
    
    if (error) {
      return (
        <div className="error-state">
          <div className="error-icon">⚠️</div>
          <div className="error-message">{error.message}</div>
          <button className="retry-button">Retry</button>
        </div>
      );
    }
    
    switch (tool.id) {
      case 'weather.forecast':
        return renderWeatherResult(result);
      case 'document.search':
        return renderDocumentSearchResult(result);
      default:
        // Generic JSON result renderer
        return (
          <div className="generic-result">
            <pre>{JSON.stringify(result, null, 2)}</pre>
          </div>
        );
    }
  };
  
  return (
    <div className={`tool-result-card ${loading ? 'loading' : ''} ${error ? 'error' : ''}`}>
      <div className="tool-result-header">
        <div className="tool-info">
          <span className="tool-icon">{getToolIcon(tool.id)}</span>
          <span className="tool-name">{tool.name}</span>
        </div>
        <div className="tool-actions">
          <button className="action-button">Copy</button>
          <button className="action-button">Save</button>
        </div>
      </div>
      
      <div className="tool-result-content">
        {renderResult()}
      </div>
    </div>
  );
};

// Helper function to get icon for tool
function getToolIcon(toolId) {
  const icons = {
    'weather.forecast': '🌤️',
    'document.search': '🔍',
    'calendar.events': '📅',
    'task.list': '✓',
    'email.send': '📧'
  };
  
  return icons[toolId] || '🔧';
}

export default ToolResultCard;
```

## Error Handling and User Feedback

User-friendly error handling:

1. **Clear error messages**
   - Plain language explanations
   - Problem identification
   - Error context
   - Severity indication

2. **Suggested remediation steps**
   - Actionable advice
   - Corrective options
   - Alternative approaches
   - Self-service solutions

3. **Retry mechanisms**
   - One-click retry
   - Parameter adjustment
   - Alternative tool suggestions
   - Fallback options

4. **Fallback options**
   - Graceful degradation
   - Alternative functionality
   - Manual process options
   - Help resources

Here's an example of error handling in a React component:

```javascript
// Error handling in React component
import React, { useState } from 'react';

const ToolExecutor = ({ toolId, initialParameters, onResult }) => {
  const [parameters, setParameters] = useState(initialParameters || {});
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [result, setResult] = useState(null);
  
  // Execute the tool
  const executeTool = async () => {
    setLoading(true);
    setError(null);
    
    try {
      // Validate parameters
      const validationError = validateParameters(toolId, parameters);
      if (validationError) {
        throw new Error(validationError);
      }
      
      // Execute the tool
      const result = await mcpClient.invokeTool(toolId, parameters);
      
      // Handle successful result
      setResult(result);
      onResult(result);
      
    } catch (err) {
      console.error('Tool execution error:', err);
      
      // Categorize error
      let errorMessage = 'An unexpected error occurred';
      let errorType = 'unknown';
      let remediation = null;
      
      if (err.message.includes('validation')) {
        errorType = 'validation';
        errorMessage = err.message;
        remediation = 'Please check your input parameters and try again.';
      } else if (err.message.includes('permission')) {
        errorType = 'permission';
        errorMessage = 'You don\'t have permission to use this tool';
        remediation = 'Please contact your administrator for access.';
      } else if (err.message.includes('timeout')) {
        errorType = 'timeout';
        errorMessage = 'The operation timed out';
        remediation = 'Please try again or use simpler parameters.';
      } else if (err.message.includes('not found')) {
        errorType = 'notFound';
        errorMessage = 'The requested resource was not found';
        remediation = 'Please check that the resource exists and try again.';
      }
      
      setError({
        message: errorMessage,
        type: errorType,
        remediation,
        rawError: err
      });
      
    } finally {
      setLoading(false);
    }
  };
  
  // Validate parameters
  const validateParameters = (toolId, params) => {
    // Get tool definition
    const tool = getToolDefinition(toolId);
    
    if (!tool) {
      return 'Unknown tool';
    }
    
    // Check required parameters
    for (const param of tool.parameters) {
      if (param.required && (params[param.name] === undefined || params[param.name] === null || params[param.name] === '')) {
        return `${param.name} is required`;
      }
    }
    
    // Type validation
    for (const param of tool.parameters) {
      const value = params[param.name];
      
      if (value === undefined || value === null) {
        continue;
      }
      
      // Type checking
      switch (param.type) {
        case 'string':
          if (typeof value !== 'string') {
            return `${param.name} must be a string`;
          }
          break;
        case 'integer':
        case 'number':
          if (typeof value !== 'number') {
            return `${param.name} must be a number`;
          }
          if (param.type === 'integer' && !Number.isInteger(value)) {
            return `${param.name} must be an integer`;
          }
          if (param.minimum !== undefined && value < param.minimum) {
            return `${param.name} must be at least ${param.minimum}`;
          }
          if (param.maximum !== undefined && value > param.maximum) {
            return `${param.name} must be at most ${param.maximum}`;
          }
          break;
        case 'boolean':