
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[Personal/Courses/MCP/Unit 4/Lesson 3/_Lesson 3|_Lesson 3]]

---

## Practical Exercise 4.3

**Objective**: Build a productivity-focused MCP integration

### Instructions

1. Create an MCP server that connects to:
   - A calendar system
   - An email service
   - A task management system

2. Implement tools for common productivity tasks:
   - Meeting scheduling
   - Email composition and analysis
   - Task creation and management
   - Notes or document creation

3. Build a simple assistant that uses these tools:
   - Create a chat interface
   - Integrate with your MCP server
   - Add natural language processing
   - Implement tool suggestion

4. Create a workflow that orchestrates multiple tools:
   - Design a multi-step workflow (e.g., meeting scheduling)
   - Implement sequential tool calls
   - Handle data passing between tools
   - Add error handling and recovery

### Expected Outcome

A working productivity assistant that:
- Connects to multiple productivity systems
- Responds to natural language requests
- Executes common productivity tasks
- Demonstrates a multi-step workflow

## Code Example (Email Analysis Tool)

Here's a starter implementation for an email analysis tool:

```python
# Email analysis MCP tool
from mcp.server import MCPServer
from mcp.schemas import MCPTool, MCPParameter
import re
import json
import datetime
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Mock email database
class EmailDB:
    def __init__(self):
        self.emails = []
        self.load_sample_emails()
    
    def load_sample_emails(self):
        # Sample emails for demonstration
        self.emails = [
            {
                "id": "email-001",
                "subject": "Quarterly Review Meeting",
                "from": "manager@example.com",
                "to": ["user@example.com"],
                "cc": ["team@example.com"],
                "body": "Let's schedule our quarterly review for next week. Please prepare your project status reports and key metrics. We'll need to discuss the roadmap for Q2 as well.",
                "date": "2025-04-08T14:30:00Z",
                "read": True,
                "folder": "inbox",
                "has_attachments": False
            },
            {
                "id": "email-002",
                "subject": "Project Deadline Extension",
                "from": "client@company.com",
                "to": ["user@example.com"],
                "cc": [],
                "body": "Due to some changes in requirements, we're extending the project deadline by two weeks. The new delivery date is May 15, 2025. Let me know if you have any questions or concerns about the extended timeline.",
                "date": "2025-04-09T10:15:00Z",
                "read": False,
                "folder": "inbox",
                "has_attachments": True
            },
            {
                "id": "email-003",
                "subject": "Team Building Event",
                "from": "hr@example.com",
                "to": ["all-staff@example.com"],
                "cc": [],
                "body": "We're organizing a team building event on April 25, 2025. Please confirm your attendance by April 18. The event will be held at Mountain View Resort and will include various activities and workshops.",
                "date": "2025-04-10T09:00:00Z",
                "read": False,
                "folder": "inbox",
                "has_attachments": False
            }
        ]
    
    def get_emails(self, folder=None, read=None, from_email=None, search_term=None, date_from=None, date_to=None):
        """Get emails with optional filtering"""
        filtered_emails = self.emails.copy()
        
        # Filter by folder
        if folder:
            filtered_emails = [e for e in filtered_emails if e["folder"] == folder]
        
        # Filter by read status
        if read is not None:
            filtered_emails = [e for e in filtered_emails if e["read"] == read]
        
        # Filter by sender
        if from_email:
            filtered_emails = [e for e in filtered_emails if e["from"] == from_email]
        
        # Filter by date range
        if date_from:
            filtered_emails = [e for e in filtered_emails if e["date"] >= date_from]
        
        if date_to:
            filtered_emails = [e for e in filtered_emails if e["date"] <= date_to]
        
        # Filter by search term
        if search_term:
            search_term = search_term.lower()
            filtered_emails = [
                e for e in filtered_emails
                if search_term in e["subject"].lower() or search_term in e["body"].lower()
            ]
        
        return filtered_emails
    
    def get_email(self, email_id):
        """Get a specific email by ID"""
        for email in self.emails:
            if email["id"] == email_id:
                return email
        return None
    
    def analyze_email(self, email_id):
        """Analyze an email for key information"""
        email = self.get_email(email_id)
        
        if not email:
            return None
        
        analysis = {
            "email_id": email_id,
            "subject": email["subject"],
            "from": email["from"],
            "date": email["date"],
            "summary": self._generate_summary(email["body"]),
            "key_points": self._extract_key_points(email["body"]),
            "action_items": self._extract_action_items(email["body"]),
            "dates_mentioned": self._extract_dates(email["body"]),
            "sentiment": self._analyze_sentiment(email["body"]),
            "urgency": self._determine_urgency(email)
        }
        
        return analysis
    
    def _generate_summary(self, text, max_length=150):
        """Generate a summary of the email (simplified version)"""
        # In a real implementation, this would use NLP techniques
        if len(text) <= max_length:
            return text
        
        # Simple approach: first sentence or truncate
        sentences = re.split(r'(?<=[.!?])\s+', text)
        if sentences and len(sentences[0]) <= max_length:
            return sentences[0]
        
        return text[:max_length] + "..."
    
    def _extract_key_points(self, text):
        """Extract key points from email (simplified version)"""
        # In a real implementation, this would use NLP techniques
        sentences = re.split(r'(?<=[.!?])\s+', text)
        
        # Simple approach: look for potential key points
        key_points = []
        
        important_phrases = [
            "important", "key", "critical", "essential", "necessary",
            "required", "need to", "must", "should", "please note"
        ]
        
        for sentence in sentences:
            for phrase in important_phrases:
                if phrase in sentence.lower():
                    key_points.append(sentence)
                    break
        
        return key_points if key_points else ["No key points identified"]
    
    def _extract_action_items(self, text):
        """Extract action items from email (simplified version)"""
        # In a real implementation, this would use NLP techniques
        sentences = re.split(r'(?<=[.!?])\s+', text)
        
        # Simple approach: look for potential action items
        action_items = []
        
        action_phrases = [
            "please", "let me know", "confirm", "review", "send", "prepare",
            "update", "complete", "submit", "respond", "provide"
        ]
        
        for sentence in sentences:
            for phrase in action_phrases:
                if phrase in sentence.lower():
                    action_items.append(sentence)
                    break
        
        return action_items if action_items else ["No action items identified"]
    
    def _extract_dates(self, text):
        """Extract dates mentioned in email (simplified version)"""
        # In a real implementation, this would use NLP techniques
        
        # Simple approach: look for date patterns
        date_patterns = [
            r'\b(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:tember)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?)\s+\d{1,2}(?:st|nd|rd|th)?,?\s+\d{4}\b',
            r'\b\d{1,2}(?:st|nd|rd|th)?\s+(?:Jan(?:uary)?|Feb(?:ruary)?|Mar(?:ch)?|Apr(?:il)?|May|Jun(?:e)?|Jul(?:y)?|Aug(?:ust)?|Sep(?:tember)?|Oct(?:ober)?|Nov(?:ember)?|Dec(?:ember)?),?\s+\d{4}\b',
            r'\b\d{4}-\d{2}-\d{2}\b',
            r'\b\d{1,2}/\d{1,2}/\d{2}(?:\d{2})?\b'
        ]
        
        dates = []
        for pattern in date_patterns:
            matches = re.findall(pattern, text, re.IGNORECASE)
            dates.extend(matches)
        
        return dates if dates else ["No dates identified"]
    
    def _analyze_sentiment(self, text):
        """Analyze sentiment of email (simplified version)"""
        # In a real implementation, this would use NLP techniques
        
        # Simple approach: count positive and negative words
        positive_words = [
            "good", "great", "excellent", "amazing", "wonderful", "fantastic",
            "pleased", "happy", "appreciate", "thank", "thanks", "grateful",
            "excited", "opportunity", "positive", "best", "success"
        ]
        
        negative_words = [
            "bad", "issue", "problem", "concern", "difficult", "unfortunately",
            "sorry", "regret", "disappoint", "fail", "missed", "delay",
            "cancel", "urgent", "complaint", "mistake", "error"
        ]
        
        text_lower = text.lower()
        positive_count = sum(1 for word in positive_words if word in text_lower)
        negative_count = sum(1 for word in negative_words if word in text_lower)
        
        if positive_count > negative_count:
            return "positive"
        elif negative_count > positive_count:
            return "negative"
        else:
            return "neutral"
    
    def _determine_urgency(self, email):
        """Determine urgency of email (simplified version)"""
        # In a real implementation, this would use more complex logic
        
        # Check subject and body for urgent words
        urgent_words = [
            "urgent", "immediate", "asap", "emergency", "critical",
            "deadline", "today", "soon", "quickly", "priority"
        ]
        
        text = email["subject"].lower() + " " + email["body"].lower()
        
        urgent_count = sum(1 for word in urgent_words if word in text)
        
        if urgent_count >= 2:
            return "high"
        elif urgent_count == 1:
            return "medium"
        else:
            return "low"

# Initialize email database
email_db = EmailDB()

# Create MCP server
server = MCPServer()

# Register get emails tool
@server.tool(
    id="email.list",
    description="Get emails with optional filtering",
    parameters=[
        MCPParameter(
            name="folder",
            description="Filter by folder",
            type="string",
            default="inbox",
            required=False
        ),
        MCPParameter(
            name="read",
            description="Filter by read status",
            type="boolean",
            required=False
        ),
        MCPParameter(
            name="from_email",
            description="Filter by sender email",
            type="string",
            required=False
        ),
        MCPParameter(
            name="search_term",
            description="Search in subject and body",
            type="string",
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
async def list_emails(folder="inbox", read=None, from_email=None, search_term=None, max_results=10):
    """Get emails with optional filtering"""
    try:
        emails = email_db.get_emails(folder, read, from_email, search_term)
        
        # Limit results
        emails = emails[:max_results]
        
        # Create filters description
        filters = {}
        if folder:
            filters["folder"] = folder
        if read is not None:
            filters["read"] = read
        if from_email:
            filters["from"] = from_email
        if search_term:
            filters["search"] = search_term
        
        return {
            "filters": filters,
            "count": len(emails),
            "emails": emails
        }
    except Exception as e:
        logger.error(f"Error listing emails: {str(e)}")
        raise Exception(f"Failed to list emails: {str(e)}")

# Register email analysis tool
@server.tool(
    id="email.analyze",
    description="Analyze an email for key information",
    parameters=[
        MCPParameter(
            name="email_id",
            description="Email ID",
            type="string",
            required=True
        )
    ]
)
async def analyze_email(email_id):
    """Analyze an email for key information"""
    try:
        analysis = email_db.analyze_email(email_id)
        
        if not analysis:
            raise Exception(f"Email not found: {email_id}")
        
        return analysis
    except Exception as e:
        logger.error(f"Error analyzing email: {str(e)}")
        raise Exception(f"Failed to analyze email: {str(e)}")

# Start server
if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    server.start(port=port)
    print(f"Email MCP server running on port {port}")
```

## Multi-tool Workflow Example

Here's a simplified example of a meeting scheduling workflow that orchestrates multiple tools:

```javascript
// Meeting scheduling workflow
async function scheduleMeetingWorkflow(client, request) {
  console.log("Starting meeting scheduling workflow");
  
  try {
    // Step 1: Parse the meeting request
    console.log("Step 1: Parsing meeting request");
    const meetingInfo = await parseMeetingRequest(request);
    
    // Step 2: Check attendee availability
    console.log("Step 2: Checking attendee availability");
    const availability = await checkAvailability(client, meetingInfo);
    
    // Step 3: Find available time slot
    console.log("Step 3: Finding available time slot");
    const timeSlot = selectBestTimeSlot(availability);
    
    // Step 4: Create calendar event
    console.log("Step 4: Creating calendar event");
    const event = await createCalendarEvent(client, meetingInfo, timeSlot);
    
    // Step 5: Send email notification
    console.log("Step 5: Sending email notification");
    const email = await sendNotificationEmail(client, meetingInfo, event);
    
    // Step 6: Create follow-up task
    console.log("Step 6: Creating follow-up task");
    const task = await createFollowUpTask(client, meetingInfo, event);
    
    // Return successful result
    return {
      status: "success",
      message: "Meeting scheduled successfully",
      event: event,
      notification: email,
      followUp: task
    };
    
  } catch (error) {
    console.error("Error in meeting scheduling workflow:", error);
    
    return {
      status: "error",
      message: `Failed to schedule meeting: ${error.message}`
    };
  }
}

// Parse meeting request using natural language
async function parseMeetingRequest(request) {
  // In a real implementation, this would use an LLM or NLP
  // For this example, we'll use a simplified approach
  
  const meetingRegex = /meeting with (.+?) (?:on|for) (.+?) (?:about|regarding) (.+)/i;
  const match = request.match(meetingRegex);
  
  if (!match) {
    throw new Error("Could not parse meeting request");
  }
  
  const [, attendeesStr, dateTimeStr, topic] = match;
  
  // Parse attendees
  const attendees = attendeesStr.split(/(?:,| and )/);
  
  // Parse date/time (simplified)
  let dateTime = new Date();
  if (dateTimeStr.includes("tomorrow")) {
    dateTime.setDate(dateTime.getDate() + 1);
  }
  
  // Format for ISO date
  const year = dateTime.getFullYear();
  const month = String(dateTime.getMonth() + 1).padStart(2, '0');
  const day = String(dateTime.getDate()).padStart(2, '0');
  const date = `${year}-${month}-${day}`;
  
  return {
    attendees: attendees.map(a => a.trim()),
    date: date,
    topic: topic,
    duration: 30 // Default to 30 minutes
  };
}

// Check availability using calendar tool
async function checkAvailability(client, meetingInfo) {
  const { attendees, date } = meetingInfo;
  
  // Convert attendees to email addresses if needed
  const attendeeEmails = attendees.map(attendee => {
    return attendee.includes('@') ? attendee : `${attendee.toLowerCase()}@example.com`;
  });
  
  try {
    // Call MCP calendar availability tool
    const result = await client.invokeTool("calendar.find_available_times", {
      users: attendeeEmails,
      date: date,
      duration_minutes: meetingInfo.duration,
      working_hours: {
        start: "09:00",
        end: "17:00"
      }
    });
    
    return result;
  } catch (error) {
    throw new Error(`Failed to check availability: ${error.message}`);
  }
}

// Select the best time slot
function selectBestTimeSlot(availability) {
  const { available_slots } = availability;
  
  if (!available_slots || available_slots.length === 0) {
    throw new Error("No available time slots found");
  }
  
  // For this example, just take the first slot
  // In a real implementation, this would use more complex logic
  return available_slots[0];
}

// Create calendar event
async function createCalendarEvent(client, meetingInfo, timeSlot) {
  const { attendees, topic } = meetingInfo;
  
  // Convert attendees to email addresses if needed
  const attendeeEmails = attendees.map(attendee => {
    return attendee.includes('@') ? attendee : `${attendee.toLowerCase()}@example.com`;
  });
  
  try {
    // Call MCP calendar create event tool
    const result = await client.invokeTool("calendar.create_event", {
      title: `Meeting: ${topic}`,
      start: timeSlot.start,
      end: timeSlot.end,
      attendees: attendeeEmails,
      description: `Meeting to discuss: ${topic}`
    });
    
    return result;
  } catch (error) {
    throw new Error(`Failed to create calendar event: ${error.message}`);
  }
}

// Send notification email
async function sendNotificationEmail(client, meetingInfo, event) {
  const { attendees, topic } = meetingInfo;
  
  // Convert attendees to email addresses if needed
  const attendeeEmails = attendees.map(attendee => {
    return attendee.includes('@') ? attendee : `${attendee.toLowerCase()}@example.com`;
  });
  
  try {
    // Call MCP email send tool
    const result = await client.invokeTool("email.send", {
      to: attendeeEmails,
      subject: `Meeting Scheduled: ${topic}`,
      body: `
        A meeting has been scheduled to discuss ${topic}.
        
        Date: ${new Date(event.start).toLocaleDateString()}
        Time: ${new Date(event.start).toLocaleTimeString()} - ${new Date(event.end).toLocaleTimeString()}
        
        Please let me know if you have any questions or need to reschedule.
      `
    });
    
    return result;
  } catch (error) {
    throw new Error(`Failed to send notification email: ${error.message}`);
  }
}

// Create follow-up task
async function createFollowUpTask(client, meetingInfo, event) {
  const { topic } = meetingInfo;
  
  try {
    // Call MCP task create tool
    const result = await client.invokeTool("tasks.create", {
      title: `Prepare for meeting: ${topic}`,
      description: `Prepare materials for the meeting scheduled on ${new Date(event.start).toLocaleDateString()}.`,
      due_date: event.start.split('T')[0], // Due on the day of the meeting
      priority: "medium",
      tags: ["meeting", "preparation"]
    });
    
    return result;
  } catch (error) {
    throw new Error(`Failed to create follow-up task: ${error.message}`);
  }
}
```

## Extension Activities

1. **Email Workflow Automation**:
   - Create an email processing workflow
   - Implement automatic categorization
   - Add follow-up task generation
   - Create email summary generation

2. **Advanced Calendar Integration**:
   - Implement recurring meeting scheduling
   - Add travel time consideration
   - Create calendar conflict resolution
   - Implement meeting preparation workflows

3. **Cross-Tool Intelligence**:
   - Create a dashboard that combines data from multiple tools
   - Implement priority-based task suggestions
   - Add predictive scheduling
   - Create automated daily planning

## Recommended Reading

1. [MCP for Productivity Applications](https://modelcontextprotocol.io/docs/guides/productivity-integration/)
   - Integration patterns for productivity tools
   - User experience considerations
   - Performance optimization
   - Implementation strategies

2. [Workflow Orchestration](https://modelcontextprotocol.io/docs/guides/workflow-orchestration/)
   - Multi-tool workflow design
   - Data passing between tools
   - Error handling strategies
   - User interaction patterns

3. [Chat Interface Design](https://modelcontextprotocol.io/docs/guides/chat-interfaces/)
   - Conversational UI best practices
   - Tool integration in chat
   - Context management
   - Error recovery in conversations

## Self-Assessment Questions

Test your understanding by answering these questions:

1. How can MCP enhance chat interfaces and assistants?
2. What patterns work well for calendar and email integration?
3. How can MCP improve task management workflows?
4. What considerations are important for multi-tool orchestration?
5. How should user consent be managed in productivity applications?
6. What data passing patterns are useful for workflow orchestration?
7. How can error recovery be implemented in multi-step workflows?
8. What security considerations are important for productivity integrations?

## Productivity Workflow Design Exercise

Design a productivity workflow for one of the following scenarios:

1. **Meeting Preparation**:
   - Design a workflow that prepares for an upcoming meeting
   - Include steps for gathering relevant documents, analyzing previous meetings, creating an agenda, and setting up follow-up tasks
   - Identify which MCP tools would be required
   - Create a sequence diagram showing the flow

2. **Email Processing**:
   - Design a workflow that helps process a cluttered inbox
   - Include steps for categorizing emails, identifying important messages, generating responses, and creating tasks
   - Identify which MCP tools would be required
   - Create a sequence diagram showing the flow

3. **Project Planning**:
   - Design a workflow that helps plan a new project
   - Include steps for gathering requirements, creating tasks, scheduling meetings, and setting up documentation
   - Identify which MCP tools would be required
   - Create a sequence diagram showing the flow

For your chosen scenario, document:
- The goal of the workflow
- Input data required
- Steps in the workflow
- MCP tools used at each step
- Data passed between steps
- Error handling considerations
- User interaction points

## Chat Interface Mockup

Create a mockup of a chat interface that integrates with your MCP productivity tools:

1. Design the main chat interface
2. Show how tools are suggested or invoked
3. Illustrate how tool execution is displayed
4. Demonstrate error handling in the conversation
5. Show multi-step workflow execution

Include annotations explaining:
- How context is maintained between interactions
- How tool suggestions are generated
- How execution results are formatted
- How errors are communicated
- How user confirmations are requested

## Additional Resources

### Integration Patterns
- [Calendar Integration Best Practices](https://modelcontextprotocol.io/docs/patterns/calendar-integration/)
- [Email Analysis Techniques](https://modelcontextprotocol.io/docs/patterns/email-analysis/)
- [Task Management Workflows](https://modelcontextprotocol.io/docs/patterns/task-management/)

### Implementation Examples
- [Chat Interface with MCP](https://modelcontextprotocol.io/examples/chat-interface/)
- [Productivity Assistant](https://modelcontextprotocol.io/examples/productivity-assistant/)
- [Email Processing System](https://modelcontextprotocol.io/examples/email-processing/)

### Advanced Topics
- [Context Management in Conversations](https://modelcontextprotocol.io/docs/topics/context-management/)
- [Natural Language Command Parsing](https://modelcontextprotocol.io/docs/topics/command-parsing/)
- [Multi-Step Workflow Orchestration](https://modelcontextprotocol.io/docs/topics/workflow-orchestration/)
