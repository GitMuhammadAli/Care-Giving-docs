# 💬 Stream Chat Setup Guide

Complete guide to setting up real-time family chat in CareCircle using Stream Chat.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Create Stream Account](#create-stream-account)
3. [Get API Keys](#get-api-keys)
4. [Configure Environment](#configure-environment)
5. [Test Chat](#test-chat)
6. [Features](#features)
7. [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

CareCircle integrates Stream Chat for real-time family communication:
- ✅ **Real-time messaging** with family members
- ✅ **File attachments** (photos, documents)
- ✅ **Typing indicators** and read receipts
- ✅ **Message reactions** with emojis
- ✅ **Push notifications** for new messages
- ✅ **Offline support** with message queueing

**Free Tier**: 5 million API calls/month, unlimited channels

---

## 🚀 Create Stream Account

### 1. Sign Up

Visit [https://getstream.io/](https://getstream.io/) and create a free account.

### 2. Create App

1. Click **"Create App"**
2. App Name: `CareCircle` (or your project name)
3. Select Region: Choose closest to your users
4. Click **"Create App"**

---

## 🔑 Get API Keys

### From Dashboard

1. Go to Stream Dashboard
2. Select your app
3. Navigate to **App Settings** → **General**
4. Find your credentials:
   - **API Key** (public - used in frontend)
   - **API Secret** (private - used in backend)

**Security Note**: Never expose the API Secret in frontend code!

---

## ⚙️ Configure Environment

### 1. Add to `.env`

```bash
# Stream Chat Configuration
NEXT_PUBLIC_STREAM_API_KEY=your_api_key_here
STREAM_API_SECRET=your_api_secret_here
```

### 2. Verify Installation

Backend package (already installed):
```bash
cd apps/api
pnpm list stream-chat
```

Frontend packages (already installed):
```bash
cd apps/web
pnpm list stream-chat stream-chat-react
```

### 3. Restart Servers

```bash
# Kill existing processes
pnpm kill

# Restart development servers
pnpm dev
```

---

## 🧪 Test Chat

### 1. Log In

Navigate to `http://localhost:3000/login` and sign in with your account.

### 2. Open Chat

Click **"Family Chat"** in the sidebar navigation.

### 3. Test Features

**Send Messages**:
- Type a message in the input box
- Press Enter or click Send
- Message appears instantly

**Attach Files**:
- Click the paperclip icon
- Select image or document
- File uploads and appears in chat

**React to Messages**:
- Hover over any message
- Click emoji icon
- Select reaction

**View Typing Indicators**:
- Open chat on two different browsers
- Start typing in one
- See "User is typing..." in the other

---

## ✨ Features

### Family Channels

Each family gets a dedicated chat channel:
- All family members auto-joined
- Real-time message sync
- Persistent history

### Direct Messages

One-on-one conversations:
```typescript
// Example: Get DM channel
const { getDirectMessageChannel } = useChat();
const dmChannel = await getDirectMessageChannel('other-user-id');
```

### Care Topic Channels

Dedicated channels for specific topics:
- Medications discussions
- Appointment planning
- Emergency coordination

### Backend Integration

The backend automatically:
- **Creates channels** when families are formed
- **Adds members** when users join families
- **Removes members** when users leave
- **Sends system messages** for activities

Example system message:
```typescript
// When medication is logged
await chatService.sendSystemMessage(
  familyId,
  '💊 John marked Aspirin as taken at 8:00 AM'
);
```

---

## 🎨 UI Customization

### Theme

The chat UI uses Stream's light theme with custom CareCircle styling:

```typescript
<Chat client={client} theme="str-chat__theme-light">
  <Channel channel={channel}>
    {/* Chat components */}
  </Channel>
</Chat>
```

### Custom CSS

Override Stream styles in `apps/web/src/styles/globals.css`:

```css
/* Example: Custom message bubble colors */
.str-chat__message--me .str-chat__message-bubble {
  background: var(--sage);
}
```

---

## 🔐 Security

### Token-Based Auth

Stream uses JWT tokens for authentication:

1. **Backend generates token** with user ID
2. **Frontend requests token** from `/api/v1/chat/token`
3. **Client connects** with token
4. **Token expires** after session ends

### Permissions

Channel permissions are managed automatically:
- Only family members can read/write
- Admins can add/remove members
- System messages from backend only

---

## 🐛 Troubleshooting

### "Chat not initialized" Error

**Cause**: Stream client not connected

**Solution**:
```typescript
// Use autoConnect option
const { client, isConnected } = useChat({ autoConnect: true });

// Wait for connection
if (!isConnected) {
  return <div>Connecting to chat...</div>;
}
```

### "Failed to load channel" Error

**Cause**: Channel doesn't exist or user not a member

**Solution**:
1. Ensure family channel created (happens on family creation)
2. Check user is family member
3. Try refreshing the page

### Messages Not Appearing

**Cause**: WebSocket connection issue

**Solution**:
1. Check browser console for errors
2. Verify API keys are correct
3. Check network tab for WebSocket connection
4. Try disabling VPN/proxy

### Rate Limit Errors

**Cause**: Exceeded free tier limits

**Solution**:
1. Check usage in Stream Dashboard
2. Implement message throttling
3. Upgrade to paid plan if needed

---

## 📊 Usage Monitoring

### Stream Dashboard

Monitor your app usage:
1. Go to Stream Dashboard
2. Select your app
3. View **Analytics**:
   - Total API calls
   - Active users
   - Message volume
   - Channel count

### Free Tier Limits

- **5 million** API calls/month
- **Unlimited** channels
- **Unlimited** messages
- **100 GB** CDN bandwidth

Typical usage for family app:
- **~1,000 API calls** per active user/month
- Supports **~5,000 active users** on free tier

---

## 🚀 Going to Production

### 1. Enable Push Notifications

Configure push notifications in Stream Dashboard:
1. **App Settings** → **Push Notifications**
2. Upload APNs certificate (iOS)
3. Add FCM credentials (Android)

### 2. Configure Webhooks

Receive events from Stream:
1. **App Settings** → **Webhooks**
2. Add your webhook URL
3. Select events to receive

### 3. Set Up Moderation

Enable auto-moderation:
1. **App Settings** → **Moderation**
2. Enable profanity filter
3. Configure block lists
4. Set up custom rules

---

## 📚 Additional Resources

- **Stream Docs**: https://getstream.io/chat/docs/
- **React SDK**: https://getstream.io/chat/docs/sdk/react/
- **API Reference**: https://getstream.io/chat/docs/api/
- **Examples**: https://github.com/GetStream/stream-chat-react

---

## 💡 Tips

**Performance**:
- Use `watch()` instead of `query()` for real-time updates
- Implement pagination for message history
- Cache channel lists locally

**User Experience**:
- Show typing indicators
- Display read receipts
- Enable emoji reactions
- Support message editing/deletion

**Maintenance**:
- Monitor usage in Stream Dashboard
- Set up error tracking (Sentry)
- Log chat events for debugging
- Regular dependency updates

---

_Stream Chat: Bringing families together in real-time. 💬_
