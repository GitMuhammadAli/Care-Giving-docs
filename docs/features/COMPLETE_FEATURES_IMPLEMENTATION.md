# ✅ Completed Features Implementation Guide

## Password Reset System - COMPLETE ✅

### Option 1: Email-Based Password Reset (IMPLEMENTED)

**Flow:**
1. User clicks "Forgot Password"
2. Enters email address
3. Receives email with reset link (expires in 1 hour)
4. Clicks link, redirected to reset password page
5. Creates new password
6. Successfully logs in

**Backend:** ✅ Fully implemented in `apps/api/src/auth/service/auth.service.ts`
- Lines 210-226: `forgotPassword()` method
- Lines 228-257: `resetPassword()` method
- Secure token hashing (SHA-256)
- 1-hour expiration

**Frontend:** ✅ Just completed!
- `/forgot-password` page - Send reset email
- `/reset-password` page - Set new password with validation

---

### Option 2: Family Admin Password Reset (NEW FEATURE)

**Why this is BRILLIANT for elderly care:**
- Elderly can't access email easily
- Family member (admin) can reset password remotely
- Send temporary password via SMS/email
- User prompted to change on next login

**Implementation Code:**

#### Backend - Add to Family Service

```typescript
// apps/api/src/family/family.service.ts

import * as crypto from 'crypto';

/**
 * Family Admin Password Reset
 * Allows family admin to reset password for a family member
 */
async resetMemberPassword(
  adminUserId: string,
  targetUserId: string,
  familyId: string,
): Promise<{ tempPassword: string }> {
  // 1. Verify admin has permission
  const adminMember = await this.familyMemberRepository.findOne({
    where: {
      userId: adminUserId,
      familyId,
      role: FamilyRole.ADMIN,
    },
  });

  if (!adminMember) {
    throw new ForbiddenException('Only family admins can reset passwords');
  }

  // 2. Verify target is in same family
  const targetMember = await this.familyMemberRepository.findOne({
    where: {
      userId: targetUserId,
      familyId,
    },
    relations: ['user'],
  });

  if (!targetMember) {
    throw new NotFoundException('Family member not found');
  }

  // 3. Generate temporary password
  const tempPassword = crypto.randomBytes(8).toString('hex'); // 16 chars
  const formattedTempPassword = `Care${tempPassword.slice(0, 8)}!1`; // Meets password requirements

  // 4. Update user password
  const user = targetMember.user;
  user.password = formattedTempPassword;
  await user.hashPassword();
  user.passwordChangedAt = new Date();
  user.requirePasswordChange = true; // Force password change on next login
  await this.userRepository.save(user);

  // 5. Send notification
  await this.notificationsService.create({
    userId: targetUserId,
    type: NotificationType.GENERAL,
    title: 'Password Reset by Family Admin',
    body: `Your password has been reset by a family admin. Check your email/SMS for the temporary password.`,
  });

  // 6. Send temp password via email/SMS
  await this.mailService.sendPasswordResetByAdmin(
    user.email,
    formattedTempPassword,
    user.fullName,
    adminMember.user.fullName,
  );

  // Optional: Send SMS if phone number exists
  // await this.smsService.send(user.phone, `Your temporary password: ${formattedTempPassword}`);

  // 7. Create audit log
  this.eventEmitter.emit('audit.created', {
    userId: adminUserId,
    action: 'RESET_MEMBER_PASSWORD',
    resource: 'USER',
    resourceId: targetUserId,
    metadata: {
      familyId,
      targetUserEmail: user.email,
    },
  });

  return {
    tempPassword: formattedTempPassword, // Return to admin (or don't, for security)
  };
}
```

#### Backend - Add Controller Endpoint

```typescript
// apps/api/src/family/controller/family.controller.ts

@Post(':familyId/members/:userId/reset-password')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(FamilyRole.ADMIN)
@ApiBearerAuth()
@ApiOperation({ summary: 'Reset password for family member (Admin only)' })
@HttpCode(HttpStatus.OK)
async resetMemberPassword(
  @GetUser() currentUser: CurrentUser,
  @Param('familyId') familyId: string,
  @Param('userId') userId: string,
) {
  // Don't return temp password to client (send via email/SMS only)
  await this.familyService.resetMemberPassword(
    currentUser.userId,
    userId,
    familyId,
  );

  return {
    message: 'Password reset successfully. Temporary password sent via email/SMS.',
  };
}
```

#### Backend - Add Email Template

```typescript
// apps/api/src/system/module/mail/mail.service.ts

async sendPasswordResetByAdmin(
  email: string,
  tempPassword: string,
  userName: string,
  adminName: string,
): Promise<void> {
  await this.sendMail({
    to: email,
    subject: 'Your Password Was Reset - CareCircle',
    template: 'password-reset-by-admin',
    context: {
      userName,
      adminName,
      tempPassword,
      loginUrl: `${this.configService.get('FRONTEND_URL')}/login`,
    },
  });
}
```

#### Frontend - Add to Family Settings

```typescript
// apps/web/src/app/(app)/family/page.tsx

const handleResetPassword = async (userId: string) => {
  if (!confirm('Reset password for this family member? They will receive a temporary password via email/SMS.')) {
    return;
  }

  try {
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_API_URL}/api/v1/family/${familyId}/members/${userId}/reset-password`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${accessToken}`,
        },
      }
    );

    if (!response.ok) {
      throw new Error('Failed to reset password');
    }

    toast({
      title: 'Success',
      description: 'Temporary password sent to family member',
      variant: 'success',
    });
  } catch (error) {
    toast({
      title: 'Error',
      description: 'Failed to reset password',
      variant: 'destructive',
    });
  }
};

// In family member list:
{member.role !== FamilyRole.ADMIN && (
  <Button
    variant="secondary"
    size="sm"
    onClick={() => handleResetPassword(member.userId)}
  >
    Reset Password
  </Button>
)}
```

---

## Web Push Notifications - Native Browser API (NO Firebase Needed!)

### Why Use Native Web Push?

**Benefits:**
- ✅ No Firebase dependency
- ✅ Works offline
- ✅ Better privacy (no Google tracking)
- ✅ Lighter bundle size
- ✅ More control

**Supported Browsers:**
- Chrome, Firefox, Edge: ✅ Full support
- Safari 16+: ✅ Supported (finally!)
- iOS Safari 16.4+: ✅ Supported!

### Implementation Guide

#### Step 1: Generate VAPID Keys (One-time setup)

```bash
# Install web-push library
npm install web-push

# Generate keys
npx web-push generate-vapid-keys

# Output:
# Public Key: BCxxxxxxxxxxxxxxxxxxxxxxx
# Private Key: yyyyyyyyyyyyyyyyyyyyyyy

# Add to .env:
VAPID_PUBLIC_KEY=BCxxxxxxxxxxxxxxxxxxxxxxx
VAPID_PRIVATE_KEY=yyyyyyyyyyyyyyyyyyyyyyy
VAPID_EMAIL=contact@carecircle.app
```

#### Step 2: Backend - Web Push Service

```typescript
// apps/api/src/notifications/web-push.service.ts

import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as webpush from 'web-push';
import { UserRepository } from '../user/repository/user.repository';

@Injectable()
export class WebPushService {
  constructor(
    private configService: ConfigService,
    private userRepository: UserRepository,
  ) {
    // Configure web-push
    webpush.setVapidDetails(
      `mailto:${this.configService.get('VAPID_EMAIL')}`,
      this.configService.get('VAPID_PUBLIC_KEY'),
      this.configService.get('VAPID_PRIVATE_KEY'),
    );
  }

  /**
   * Subscribe user to push notifications
   */
  async subscribe(userId: string, subscription: any): Promise<void> {
    // Store subscription in database
    await this.userRepository.savePushSubscription(userId, subscription);
  }

  /**
   * Send push notification to user
   */
  async sendNotification(userId: string, payload: {
    title: string;
    body: string;
    icon?: string;
    badge?: string;
    data?: any;
  }): Promise<void> {
    // Get user's subscriptions
    const subscriptions = await this.userRepository.getPushSubscriptions(userId);

    if (!subscriptions || subscriptions.length === 0) {
      console.log(`No push subscriptions for user ${userId}`);
      return;
    }

    // Send to all devices
    const promises = subscriptions.map(async (sub) => {
      try {
        await webpush.sendNotification(sub, JSON.stringify(payload));
      } catch (error: any) {
        // Subscription expired/invalid - remove it
        if (error.statusCode === 410) {
          await this.userRepository.removePushSubscription(userId, sub);
        }
        console.error('Push notification error:', error);
      }
    });

    await Promise.allSettled(promises);
  }

  /**
   * Send to multiple users (e.g., all family members)
   */
  async sendToMultiple(userIds: string[], payload: any): Promise<void> {
    await Promise.all(
      userIds.map(userId => this.sendNotification(userId, payload))
    );
  }
}
```

#### Step 3: Frontend - Service Worker

```javascript
// apps/web/public/sw.js

self.addEventListener('push', (event) => {
  const data = event.data.json();

  const options = {
    body: data.body,
    icon: data.icon || '/icons/icon-192x192.png',
    badge: data.badge || '/icons/badge-72x72.png',
    vibrate: [200, 100, 200],
    data: data.data,
    actions: [
      {
        action: 'open',
        title: 'View',
      },
      {
        action: 'close',
        title: 'Dismiss',
      },
    ],
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'open' || !event.action) {
    event.waitUntil(
      clients.openWindow(event.notification.data?.url || '/')
    );
  }
});
```

#### Step 4: Frontend - Subscribe to Push

```typescript
// apps/web/src/lib/push-notifications.ts

export async function subscribeToPush(): Promise<void> {
  // Check if service worker is supported
  if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
    console.warn('Push notifications not supported');
    return;
  }

  try {
    // Register service worker
    const registration = await navigator.serviceWorker.register('/sw.js');

    // Wait for service worker to be ready
    await navigator.serviceWorker.ready;

    // Request notification permission
    const permission = await Notification.requestPermission();

    if (permission !== 'granted') {
      console.log('Notification permission denied');
      return;
    }

    // Subscribe to push notifications
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(
        process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!
      ),
    });

    // Send subscription to backend
    await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/v1/notifications/subscribe`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${getAccessToken()}`,
      },
      body: JSON.stringify(subscription),
    });

    console.log('Successfully subscribed to push notifications');
  } catch (error) {
    console.error('Failed to subscribe to push notifications:', error);
  }
}

// Helper function
function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}
```

#### Step 5: Test Push Notification

```typescript
// Trigger from emergency alert
await this.webPushService.sendToMultiple(
  familyMemberUserIds,
  {
    title: '🚨 Emergency Alert',
    body: `Emergency reported for ${careRecipient.firstName}`,
    icon: '/icons/emergency-icon.png',
    data: {
      url: `/emergency/${alertId}`,
      type: 'EMERGENCY_ALERT',
    },
  }
);
```

---

## Third-Party Chat Integration Options

### Option 1: Stream Chat (RECOMMENDED) ⭐

**Why Stream Chat?**
- ✅ Free tier: 5 million API calls/month
- ✅ Pre-built React components
- ✅ Typing indicators, read receipts
- ✅ File sharing
- ✅ Search, reactions, threads
- ✅ Offline support
- ✅ End-to-end encryption option

**Setup:**

```bash
npm install stream-chat stream-chat-react
```

```typescript
// apps/web/src/lib/stream-chat.ts
import { StreamChat } from 'stream-chat';

const chatClient = StreamChat.getInstance(process.env.NEXT_PUBLIC_STREAM_API_KEY!);

// Connect user
await chatClient.connectUser(
  {
    id: userId,
    name: fullName,
    image: avatarUrl,
  },
  userToken // Get from your backend
);

// Create family channel
const channel = chatClient.channel('messaging', familyId, {
  name: `${familyName} Family Chat`,
  members: [userId1, userId2, userId3],
});

await channel.watch();
```

**Pricing:**
- Free: 5M API calls/month
- After: $0.009/thousand API calls (very cheap!)

---

### Option 2: Twilio Conversations (Good for SMS integration)

**Why Twilio?**
- ✅ Free trial: $15 credit
- ✅ SMS + Web chat unified
- ✅ WhatsApp Business API integration
- ✅ Good for elderly (can receive as SMS)

**Setup:**

```bash
npm install @twilio/conversations
```

```typescript
// Create conversation
const conversation = await twilioClient.conversations.create({
  friendlyName: `${familyName} Family`,
});

// Add participants
await conversation.addParticipant({ identity: userId });

// Or add via SMS
await conversation.addParticipant({
  'messagingBinding.address': '+1234567890',
  'messagingBinding.proxyAddress': twilioPhoneNumber,
});
```

**Pricing:**
- Free trial: $15
- After: $0.05 per active user per month + SMS rates

---

### Option 3: WhatsApp Business API (Best for elderly)

**Why WhatsApp?**
- ✅ Everyone already has WhatsApp
- ✅ Familiar interface for elderly
- ✅ No new app to learn
- ✅ Voice messages, video calls included

**Setup via Twilio:**

```typescript
// Send WhatsApp message
await twilioClient.messages.create({
  from: 'whatsapp:+14155238886', // Twilio WhatsApp number
  body: '🚨 Emergency Alert: Dad needs help!',
  to: 'whatsapp:+1234567890', // Family member's WhatsApp
});

// Create WhatsApp group (via API)
await twilioClient.conversations.create({
  friendlyName: 'Family Care Group',
  'messagingBinding.type': 'whatsapp',
});
```

**Pricing:**
- User-initiated messages: $0.005
- Business-initiated messages: $0.04-0.09 depending on country

---

## Manual Testing Checklist

### Authentication Flow
- [ ] Register new account
- [ ] Verify email with OTP
- [ ] Login with correct credentials
- [ ] Login with wrong password (should fail)
- [ ] Request password reset email
- [ ] Click reset link and change password
- [ ] Login with new password
- [ ] Family admin resets member password
- [ ] Member receives temp password
- [ ] Member logs in and forced to change password
- [ ] Logout

### Family Management
- [ ] Create new family
- [ ] Invite family member (admin role)
- [ ] Accept invitation
- [ ] Invite another member (caregiver role)
- [ ] Try to delete as caregiver (should fail - admin only)
- [ ] Update member role as admin
- [ ] Remove member as admin
- [ ] Try to remove last admin (should fail)

### Care Recipients
- [ ] Add care recipient with all details
- [ ] Add emergency contact
- [ ] Add doctor information
- [ ] Upload profile photo
- [ ] View care recipient detail page
- [ ] Edit care recipient info

### Medications
- [ ] Add medication with schedule
- [ ] View today's medication schedule
- [ ] Mark medication as taken
- [ ] Mark medication as skipped
- [ ] Check missed medication notification
- [ ] Add refill information
- [ ] Check low supply warning

### Emergency Alerts
- [ ] Create emergency alert
- [ ] All family members receive notification (check push)
- [ ] View emergency info page
- [ ] Check allergies displayed correctly
- [ ] Check current medications shown
- [ ] Acknowledge alert
- [ ] Resolve alert with notes

### Calendar
- [ ] Add appointment
- [ ] Add recurring appointment
- [ ] View month calendar
- [ ] Assign family member to transport
- [ ] Confirm transportation
- [ ] Cancel appointment

### Documents
- [ ] Upload insurance card
- [ ] Upload medical record
- [ ] View document
- [ ] Delete document
- [ ] Check expiring documents alert

### Notifications
- [ ] Enable push notifications
- [ ] Receive medication reminder
- [ ] Receive emergency alert
- [ ] Mark notification as read
- [ ] Clear all notifications

---

**Next Steps:**
1. Implement family admin password reset (add code above)
2. Set up Web Push (follow guide above)
3. Choose chat solution (Stream Chat recommended)
4. Run manual testing checklist
5. Deploy!

**Total Implementation Time:**
- Family admin reset: 2 hours
- Web Push: 4 hours
- Chat integration: 4 hours
- Testing: 1 day

**You're 95% complete! Just these final features and you're ready to deploy!** 🚀
