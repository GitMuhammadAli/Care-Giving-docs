# Stream Chat React

> Full-featured chat UI components for family messaging.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | React components for Stream Chat |
| **Why** | Pre-built chat UI, real-time messaging |
| **Version** | Latest |
| **Location** | `apps/web/src/app/(app)/chat/` |

## Setup

### Environment Variables
```env
NEXT_PUBLIC_STREAM_API_KEY=your_api_key
STREAM_API_SECRET=your_api_secret  # Backend only
```

### Chat Provider
```tsx
// providers/StreamChatProvider.tsx
'use client';

import { useEffect, useState } from 'react';
import { StreamChat } from 'stream-chat';
import { Chat } from 'stream-chat-react';
import { useAuthStore } from '@/stores/authStore';
import api from '@/lib/api';

const apiKey = process.env.NEXT_PUBLIC_STREAM_API_KEY!;

export function StreamChatProvider({ children }: { children: React.ReactNode }) {
  const [client, setClient] = useState<StreamChat | null>(null);
  const { user, isAuthenticated } = useAuthStore();

  useEffect(() => {
    if (!isAuthenticated || !user) return;

    const initChat = async () => {
      // Get token from backend
      const { token } = await api.getChatToken();

      const chatClient = StreamChat.getInstance(apiKey);
      
      await chatClient.connectUser(
        {
          id: user.id,
          name: user.fullName,
          image: user.avatarUrl,
        },
        token
      );

      setClient(chatClient);
    };

    initChat();

    return () => {
      client?.disconnectUser();
    };
  }, [isAuthenticated, user]);

  if (!client) return <div>Loading chat...</div>;

  return (
    <Chat client={client} theme="str-chat__theme-light">
      {children}
    </Chat>
  );
}
```

## Basic Chat UI

### Full Chat Page
```tsx
// app/(app)/chat/page.tsx
'use client';

import {
  Channel,
  ChannelHeader,
  ChannelList,
  MessageInput,
  MessageList,
  Thread,
  Window,
} from 'stream-chat-react';
import { useAuthStore } from '@/stores/authStore';

export default function ChatPage() {
  const { user } = useAuthStore();

  // Filter for family channels the user is a member of
  const filters = { type: 'family', members: { $in: [user?.id] } };
  const sort = { last_message_at: -1 };

  return (
    <div className="flex h-[calc(100vh-80px)]">
      {/* Channel List Sidebar */}
      <div className="w-80 border-r">
        <ChannelList
          filters={filters}
          sort={sort}
          Preview={CustomChannelPreview}
        />
      </div>

      {/* Chat Window */}
      <div className="flex-1">
        <Channel>
          <Window>
            <ChannelHeader />
            <MessageList />
            <MessageInput />
          </Window>
          <Thread />
        </Channel>
      </div>
    </div>
  );
}
```

### Custom Channel Preview
```tsx
import { ChannelPreviewUIComponentProps } from 'stream-chat-react';

function CustomChannelPreview(props: ChannelPreviewUIComponentProps) {
  const { channel, setActiveChannel, displayTitle, latestMessage, unread } = props;

  return (
    <button
      onClick={() => setActiveChannel?.(channel)}
      className={`w-full p-4 text-left hover:bg-gray-50 ${
        unread ? 'bg-blue-50' : ''
      }`}
    >
      <div className="flex items-center justify-between">
        <span className="font-medium">{displayTitle}</span>
        {unread > 0 && (
          <span className="bg-primary-500 text-white text-xs px-2 py-1 rounded-full">
            {unread}
          </span>
        )}
      </div>
      <p className="text-sm text-gray-500 truncate">{latestMessage}</p>
    </button>
  );
}
```

## Family Channels

### Initialize Family Channel
```typescript
// Called when user opens chat for a family
async function initializeFamilyChat(familyId: string) {
  const response = await api.get(`/chat/family/${familyId}/init`);
  
  if (response.success) {
    // Channel exists or was created
    return {
      channelId: response.channelId,
      channelType: response.channelType,
    };
  }
  
  throw new Error('Failed to initialize family chat');
}
```

### Create Topic Channel
```typescript
// Create a channel for a specific care topic
async function createTopicChannel(
  familyId: string,
  topic: string,
  topicName: string
) {
  const response = await api.post(`/chat/family/${familyId}/topic`, {
    topic,
    topicName,
    memberIds: [], // Optional: specific members
  });

  return response.channelId;
}

// Example: Create "Medication Discussion" channel
createTopicChannel(familyId, 'medications', 'Medication Discussion');
```

## Custom Components

### Custom Message
```tsx
import { MessageSimple, MessageUIComponentProps } from 'stream-chat-react';

function CustomMessage(props: MessageUIComponentProps) {
  const { message } = props;

  // Add custom styling for urgent messages
  const isUrgent = message.type === 'emergency';

  return (
    <div className={isUrgent ? 'bg-red-50 border-l-4 border-red-500' : ''}>
      <MessageSimple {...props} />
    </div>
  );
}

// Usage
<Channel Message={CustomMessage}>
  ...
</Channel>
```

### Custom Message Input
```tsx
import { MessageInputFlat, MessageInputProps } from 'stream-chat-react';

function CustomMessageInput(props: MessageInputProps) {
  return (
    <div className="p-4 border-t">
      <MessageInputFlat {...props} />
    </div>
  );
}
```

## Notifications

### Unread Count
```tsx
import { useChatContext } from 'stream-chat-react';

function ChatNotificationBadge() {
  const { client } = useChatContext();
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    const handleEvent = () => {
      const count = client?.user?.total_unread_count || 0;
      setUnreadCount(count);
    };

    client?.on('notification.mark_read', handleEvent);
    client?.on('notification.message_new', handleEvent);

    // Initial count
    handleEvent();

    return () => {
      client?.off('notification.mark_read', handleEvent);
      client?.off('notification.message_new', handleEvent);
    };
  }, [client]);

  if (unreadCount === 0) return null;

  return (
    <span className="bg-red-500 text-white text-xs px-2 py-1 rounded-full">
      {unreadCount}
    </span>
  );
}
```

## Styling

### CSS Imports
```tsx
// Import Stream Chat styles
import 'stream-chat-react/dist/css/v2/index.css';

// Or customize with your own theme
import './custom-chat-theme.css';
```

### Custom Theme Variables
```css
/* custom-chat-theme.css */
.str-chat {
  --str-chat__primary-color: #0ea5e9;
  --str-chat__active-primary-color: #0284c7;
  --str-chat__surface-color: #ffffff;
  --str-chat__secondary-surface-color: #f8fafc;
  --str-chat__primary-surface-color: #f0f9ff;
  --str-chat__primary-surface-color-low-emphasis: #e0f2fe;
  --str-chat__border-radius-circle: 9999px;
}
```

## File Uploads

### Image & Document Sharing
```tsx
<Channel
  acceptedFiles={['image/*', 'application/pdf', '.doc', '.docx']}
  maxNumberOfFiles={5}
  multipleUploads={true}
>
  <Window>
    <MessageList />
    <MessageInput />
  </Window>
</Channel>
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/chat/token` | GET | Get user's chat token |
| `/api/v1/chat/family/:familyId/init` | GET | Initialize family channel |
| `/api/v1/chat/family/:familyId/channel` | POST | Create family channel |
| `/api/v1/chat/family/:familyId/topic` | POST | Create topic channel |
| `/api/v1/chat/channels` | GET | Get user's channels |
| `/api/v1/chat/status` | GET | Check if Stream Chat is configured |

## Troubleshooting

### "Chat not configured"
- Verify `NEXT_PUBLIC_STREAM_API_KEY` is set
- Verify `STREAM_API_SECRET` is set on backend

### User Not Connected
- Check if token is valid
- Verify user ID matches backend

### Messages Not Sending
- Check channel permissions
- Verify user is a channel member

---

*See also: [Socket.io Client](socket-io-client.md), [Chat Backend](../backend/nestjs.md)*


