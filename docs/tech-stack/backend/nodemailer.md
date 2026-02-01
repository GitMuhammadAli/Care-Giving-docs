# Nodemailer

> Email sending for transactional emails.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Node.js email sending library |
| **Why** | Reliable email delivery, templating support |
| **Version** | 6.x |
| **Location** | `apps/api/src/mail/` |

## Setup

```typescript
// mail/mail.module.ts
import { Module } from '@nestjs/common';
import { MailService } from './mail.service';

@Module({
  providers: [MailService],
  exports: [MailService],
})
export class MailModule {}
```

## Mail Service

```typescript
// mail/mail.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as nodemailer from 'nodemailer';
import { Transporter } from 'nodemailer';

@Injectable()
export class MailService {
  private transporter: Transporter;
  private readonly logger = new Logger(MailService.name);

  constructor(private config: ConfigService) {
    this.transporter = nodemailer.createTransport({
      host: config.get('MAILTRAP_HOST'),
      port: config.get('MAILTRAP_PORT'),
      auth: {
        user: config.get('MAILTRAP_USER'),
        pass: config.get('MAILTRAP_PASS'),
      },
    });
  }

  async sendEmail(options: SendEmailOptions): Promise<void> {
    try {
      await this.transporter.sendMail({
        from: `"${this.config.get('MAIL_FROM_NAME')}" <${this.config.get('MAIL_FROM')}>`,
        to: options.to,
        subject: options.subject,
        html: options.html,
        text: options.text,
      });
      this.logger.log(`Email sent to ${options.to}`);
    } catch (error) {
      this.logger.error(`Failed to send email to ${options.to}`, error);
      throw error;
    }
  }
}
```

## Email Templates

### Verification Email
```typescript
async sendVerificationEmail(email: string, otp: string, name: string) {
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #0ea5e9; color: white; padding: 20px; text-align: center; }
        .content { padding: 30px; background: #f8fafc; }
        .otp { font-size: 32px; font-weight: bold; color: #0ea5e9; 
               text-align: center; padding: 20px; background: white;
               border-radius: 8px; margin: 20px 0; }
        .footer { text-align: center; padding: 20px; color: #64748b; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>Welcome to CareCircle</h1>
        </div>
        <div class="content">
          <p>Hi ${name},</p>
          <p>Please use the following code to verify your email address:</p>
          <div class="otp">${otp}</div>
          <p>This code expires in 5 minutes.</p>
          <p>If you didn't create an account, please ignore this email.</p>
        </div>
        <div class="footer">
          <p>© ${new Date().getFullYear()} CareCircle. All rights reserved.</p>
        </div>
      </div>
    </body>
    </html>
  `;

  await this.sendEmail({
    to: email,
    subject: 'Verify your CareCircle account',
    html,
    text: `Your verification code is: ${otp}. This code expires in 5 minutes.`,
  });
}
```

### Password Reset Email
```typescript
async sendPasswordResetEmail(email: string, resetToken: string) {
  const resetUrl = `${this.config.get('FRONTEND_URL')}/reset-password?token=${resetToken}`;

  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #0ea5e9; color: white; padding: 20px; text-align: center; }
        .content { padding: 30px; background: #f8fafc; }
        .button { display: inline-block; background: #0ea5e9; color: white;
                  padding: 12px 24px; text-decoration: none; border-radius: 8px; }
        .footer { text-align: center; padding: 20px; color: #64748b; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>Password Reset</h1>
        </div>
        <div class="content">
          <p>You requested to reset your password.</p>
          <p>Click the button below to set a new password:</p>
          <p style="text-align: center;">
            <a href="${resetUrl}" class="button">Reset Password</a>
          </p>
          <p>This link expires in 1 hour.</p>
          <p>If you didn't request this, please ignore this email.</p>
        </div>
        <div class="footer">
          <p>© ${new Date().getFullYear()} CareCircle. All rights reserved.</p>
        </div>
      </div>
    </body>
    </html>
  `;

  await this.sendEmail({
    to: email,
    subject: 'Reset your CareCircle password',
    html,
    text: `Reset your password: ${resetUrl}. This link expires in 1 hour.`,
  });
}
```

### Family Invitation Email
```typescript
async sendFamilyInvitationEmail(
  email: string,
  inviterName: string,
  familyName: string,
  inviteToken: string,
  role: string
) {
  const inviteUrl = `${this.config.get('FRONTEND_URL')}/accept-invite/${inviteToken}`;

  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #0ea5e9; color: white; padding: 20px; text-align: center; }
        .content { padding: 30px; background: #f8fafc; }
        .highlight { background: white; padding: 20px; border-radius: 8px; margin: 20px 0; }
        .button { display: inline-block; background: #10b981; color: white;
                  padding: 14px 28px; text-decoration: none; border-radius: 8px;
                  font-weight: bold; }
        .footer { text-align: center; padding: 20px; color: #64748b; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>You're Invited!</h1>
        </div>
        <div class="content">
          <p><strong>${inviterName}</strong> has invited you to join the <strong>${familyName}</strong> family on CareCircle.</p>
          
          <div class="highlight">
            <p><strong>Your Role:</strong> ${role}</p>
            <p>As a ${role.toLowerCase()}, you'll be able to help coordinate care for your loved ones.</p>
          </div>

          <p style="text-align: center;">
            <a href="${inviteUrl}" class="button">Accept Invitation</a>
          </p>

          <p>This invitation expires in 7 days.</p>
        </div>
        <div class="footer">
          <p>© ${new Date().getFullYear()} CareCircle. All rights reserved.</p>
        </div>
      </div>
    </body>
    </html>
  `;

  await this.sendEmail({
    to: email,
    subject: `${inviterName} invited you to join ${familyName} on CareCircle`,
    html,
    text: `${inviterName} invited you to join ${familyName}. Accept: ${inviteUrl}`,
  });
}
```

### Medication Reminder Email
```typescript
async sendMedicationReminderEmail(
  email: string,
  recipientName: string,
  medicationName: string,
  dosage: string,
  scheduledTime: string
) {
  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #f59e0b; color: white; padding: 20px; text-align: center; }
        .content { padding: 30px; background: #fffbeb; }
        .medication { background: white; padding: 20px; border-radius: 8px; 
                      border-left: 4px solid #f59e0b; margin: 20px 0; }
        .footer { text-align: center; padding: 20px; color: #64748b; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>⏰ Medication Reminder</h1>
        </div>
        <div class="content">
          <p>It's time for ${recipientName}'s medication:</p>
          
          <div class="medication">
            <p><strong>Medication:</strong> ${medicationName}</p>
            <p><strong>Dosage:</strong> ${dosage}</p>
            <p><strong>Scheduled Time:</strong> ${scheduledTime}</p>
          </div>

          <p>Please ensure this medication is administered as scheduled.</p>
        </div>
        <div class="footer">
          <p>© ${new Date().getFullYear()} CareCircle. All rights reserved.</p>
        </div>
      </div>
    </body>
    </html>
  `;

  await this.sendEmail({
    to: email,
    subject: `Medication Reminder: ${medicationName} for ${recipientName}`,
    html,
    text: `Time for ${recipientName}'s medication: ${medicationName} ${dosage} at ${scheduledTime}`,
  });
}
```

## Queue Integration

```typescript
// For high-volume email sending, use BullMQ
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class MailService {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
  ) {}

  async queueEmail(options: SendEmailOptions) {
    await this.emailQueue.add('send', options, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 5000 },
    });
  }
}

// Email processor
@Processor('email')
export class EmailProcessor {
  @Process('send')
  async handleSend(job: Job<SendEmailOptions>) {
    await this.mailService.sendEmail(job.data);
  }
}
```

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MAILTRAP_HOST` | SMTP host | `sandbox.smtp.mailtrap.io` |
| `MAILTRAP_PORT` | SMTP port | `2525` |
| `MAILTRAP_USER` | SMTP username | `username` |
| `MAILTRAP_PASS` | SMTP password | `password` |
| `MAIL_FROM` | Sender email | `noreply@carecircle.com` |
| `MAIL_FROM_NAME` | Sender name | `CareCircle` |

## Providers

### Development (Mailtrap)
- All emails captured, not delivered
- View in Mailtrap inbox
- Free tier: 500 emails/month

### Production Options
- **Resend**: Developer-friendly, good API
- **SendGrid**: High volume, analytics
- **Amazon SES**: Cost-effective at scale
- **Postmark**: Transactional focus

## Troubleshooting

### Emails Not Sending
- Check SMTP credentials
- Verify firewall/network access
- Check Mailtrap inbox (dev)

### Emails Going to Spam
- Configure SPF/DKIM/DMARC
- Use consistent sender address
- Avoid spam trigger words

### Slow Sending
- Use queue for bulk emails
- Implement rate limiting
- Consider dedicated email service

---

*See also: [BullMQ](../workers/bullmq.md), [Authentication](authentication.md)*


