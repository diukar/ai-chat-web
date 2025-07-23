# AI Chat Widget Integration Guide

## Overview
This guide explains how to integrate the AI Chat Widget into any website. The widget sends user messages to a webhook endpoint and displays AI responses in a beautiful chat interface.

## Prerequisites
- React application (React 18+)
- Tailwind CSS configured
- shadcn/ui components
- A webhook endpoint that can process chat messages

## Installation

### 1. Copy Required Files
Copy these files to your project:
```
src/hooks/useChat.ts
src/components/ChatWidget.tsx
src/components/ui/* (shadcn components)
```

### 2. Install Dependencies
```bash
npm install lucide-react class-variance-authority clsx tailwind-merge
```

### 3. Configure Tailwind CSS
Ensure your `tailwind.config.ts` includes the shadcn/ui configuration and your `index.css` has the design tokens.

## Usage

### Basic Integration
```tsx
import { ChatWidget } from '@/components/ChatWidget';

function App() {
  return (
    <div>
      {/* Your app content */}
      <ChatWidget webhookUrl="https://your-webhook-endpoint.com/chat" />
    </div>
  );
}
```

### With Custom Styling
```tsx
<ChatWidget 
  webhookUrl="https://your-webhook-endpoint.com/chat"
  className="bottom-4 right-4" // Custom positioning
/>
```

## Webhook Requirements

### Request Format
The widget sends POST requests with this structure:
```json
{
  "message": "User's message text",
  "sessionId": "user_1234567890_abc123def",
  "userInfo": {
    "page": "/current-page",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "url": "https://example.com/current-page"
  },
  "conversationHistory": [
    {
      "role": "user",
      "content": "Previous message",
      "timestamp": "2024-01-15T10:29:30.000Z"
    },
    {
      "role": "assistant", 
      "content": "Previous response",
      "timestamp": "2024-01-15T10:29:35.000Z"
    }
  ]
}
```

### Required Response Format
Your webhook must return JSON in this format:
```json
{
  "response": "AI assistant's response text",
  "timestamp": "2024-01-15T10:30:05.000Z",
  "sessionId": "user_1234567890_abc123def",
  "status": "success"
}
```

### Response Content Formatting
The AI response supports:
- **Plain URLs**: `https://example.com` (automatically converted to clickable links)
- **Markdown links**: `[Click here](https://example.com)` (rendered as clickable text)
- **Line breaks**: Use `\n` for line breaks in responses
- **Mixed content**: Combine text, links, and line breaks naturally

Example response:
```json
{
  "response": "You can book our service at [this link](https://example.com/booking).\n\nFor more info, visit https://example.com/about\n\nCall us at (555) 123-4567",
  "status": "success"
}
```

## Error Handling

### Webhook Error Response
If your webhook encounters an error, return:
```json
{
  "error": "Error description",
  "status": "error"
}
```

### Network Failures
The widget automatically handles network failures and displays user-friendly error messages.

## Customization

### Styling
The widget uses Tailwind CSS classes and can be customized by:
1. Modifying the `className` prop for positioning
2. Updating the design tokens in your `index.css`
3. Customizing the shadcn/ui theme

### Behavior
Key customization points in `ChatWidget.tsx`:
- Chat title: Line 139 - `"Pure Space Ozone Assistant"`
- Welcome message: Lines 168-170
- Input placeholder: Line 222 - `"Ask about odor removal..."`
- Error messages: Lines 88, 98-101

### Session Management
- Each chat session gets a unique `sessionId`
- Sessions persist until page refresh or manual chat clearing
- Clear chat functionality resets the session

## Example Webhook Implementation

### Node.js/Express Example
```javascript
app.post('/chat', async (req, res) => {
  try {
    const { message, sessionId, userInfo, conversationHistory } = req.body;

    // Process with your AI service
    const aiResponse = await yourAIService.generateResponse({
      message,
      context: conversationHistory,
      userInfo
    });

    res.json({
      response: aiResponse,
      timestamp: new Date().toISOString(),
      sessionId: sessionId,
      status: "success"
    });

  } catch (error) {
    res.status(500).json({
      error: "Failed to process message",
      status: "error"
    });
  }
});
```

### n8n Workflow Example
1. **Webhook Trigger**: Receive POST requests
2. **AI Node**: Process message with your AI service
3. **Response Node**: Return formatted JSON response

Response body template:
```json
{
  "response": "{{ $json.choices[0].message.content }}",
  "timestamp": "{{ new Date().toISOString() }}",
  "sessionId": "{{ $json.sessionId }}",
  "status": "success"
}
```

## Security Considerations

### CORS Configuration
Ensure your webhook endpoint allows requests from your domain:
```javascript
app.use(cors({
  origin: ['https://yourdomain.com', 'http://localhost:3000'],
  methods: ['POST'],
  allowedHeaders: ['Content-Type']
}));
```

### Rate Limiting
Implement rate limiting to prevent abuse:
```javascript
const rateLimit = require('express-rate-limit');

const chatLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 10, // 10 requests per minute
  message: { error: "Too many requests", status: "error" }
});

app.use('/chat', chatLimiter);
```

### Input Validation
Always validate and sanitize incoming messages:
```javascript
const validator = require('validator');

// Validate message content
if (!req.body.message || req.body.message.length > 1000) {
  return res.status(400).json({
    error: "Invalid message",
    status: "error"
  });
}

// Sanitize input
const sanitizedMessage = validator.escape(req.body.message);
```

## Testing

### Manual Testing
1. Deploy the widget to your site
2. Open browser developer tools
3. Send test messages and verify network requests
4. Check webhook logs for proper data reception

### Automated Testing
Test the useChat hook:
```javascript
import { renderHook, act } from '@testing-library/react';
import { useChat } from './useChat';

test('sends message to webhook', async () => {
  const { result } = renderHook(() => useChat('https://test-webhook.com'));

  await act(async () => {
    await result.current.sendMessage('Test message');
  });

  expect(result.current.messages).toHaveLength(2); // User + AI message
});
```

## Troubleshooting

### Common Issues
1. **CORS errors**: Configure webhook CORS settings
2. **Styling issues**: Ensure Tailwind CSS and shadcn/ui are properly configured
3. **Webhook timeout**: Implement proper error handling and timeouts
4. **Missing dependencies**: Install all required packages

### Debug Mode
Add console logging to track requests:
```javascript
// In useChat.ts, add after line 42:
console.log('Sending to webhook:', {
  message: message.trim(),
  sessionId: session.sessionId,
  userInfo,
  conversationHistory: session.messages
});
```

## Support
- Check browser network tab for failed requests
- Verify webhook endpoint accessibility
- Test webhook independently with tools like Postman
- Ensure proper JSON response format
