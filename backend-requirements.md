# Backend Server Requirements for Real Email Integration

## Overview
To implement real email integration, you need a backend server that handles IMAP connections securely. Here's what you need to build:

## Technology Stack
- **Node.js** with Express.js
- **Database**: PostgreSQL or MongoDB
- **Email Libraries**: `node-imap`, `mailparser`
- **Security**: `bcrypt`, `jsonwebtoken`, `helmet`
- **Environment**: `dotenv`

## Required API Endpoints

### Email Configuration Management
```
POST   /api/email-configs           - Add new email configuration
GET    /api/users/:id/email-configs - Get user's email configurations
PATCH  /api/email-configs/:id       - Update email configuration
DELETE /api/email-configs/:id       - Delete email configuration
POST   /api/email-configs/:id/test  - Test IMAP connection
```

### Email Synchronization
```
POST   /api/email-configs/:id/sync  - Sync emails from IMAP server
GET    /api/users/:id/job-emails    - Get processed job emails
PATCH  /api/job-emails/:id/read     - Mark email as read
PATCH  /api/job-emails/:id/star     - Toggle email star
```

### Health Check
```
GET    /api/health                  - Server health check
```

## Database Schema

### email_configs table
```sql
CREATE TABLE email_configs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  email VARCHAR(255) NOT NULL,
  provider VARCHAR(100) NOT NULL,
  imap_server VARCHAR(255) NOT NULL,
  port INTEGER NOT NULL,
  username VARCHAR(255) NOT NULL,
  encrypted_password TEXT NOT NULL,
  ssl BOOLEAN DEFAULT true,
  is_connected BOOLEAN DEFAULT false,
  last_sync TIMESTAMP,
  sync_enabled BOOLEAN DEFAULT true,
  auto_track BOOLEAN DEFAULT true,
  smart_categorization BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### job_emails table
```sql
CREATE TABLE job_emails (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  email_config_id UUID NOT NULL,
  message_id VARCHAR(255) NOT NULL,
  from_address VARCHAR(255) NOT NULL,
  subject TEXT NOT NULL,
  body TEXT NOT NULL,
  email_date TIMESTAMP NOT NULL,
  is_read BOOLEAN DEFAULT false,
  is_starred BOOLEAN DEFAULT false,
  category VARCHAR(50) NOT NULL,
  company_name VARCHAR(255),
  position VARCHAR(255),
  priority VARCHAR(20) DEFAULT 'medium',
  action_required BOOLEAN DEFAULT false,
  confidence INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (email_config_id) REFERENCES email_configs(id)
);
```

## Security Considerations

### Password Encryption
```javascript
const crypto = require('crypto');

// Encrypt password before storing
function encryptPassword(password, secretKey) {
  const cipher = crypto.createCipher('aes-256-cbc', secretKey);
  let encrypted = cipher.update(password, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}

// Decrypt password for IMAP connection
function decryptPassword(encryptedPassword, secretKey) {
  const decipher = crypto.createDecipher('aes-256-cbc', secretKey);
  let decrypted = decipher.update(encryptedPassword, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

### IMAP Connection Example
```javascript
const Imap = require('node-imap');
const { simpleParser } = require('mailparser');

class EmailService {
  constructor(config) {
    this.imap = new Imap({
      user: config.username,
      password: decryptPassword(config.encryptedPassword, process.env.ENCRYPTION_KEY),
      host: config.imapServer,
      port: config.port,
      tls: config.ssl,
      tlsOptions: { rejectUnauthorized: false }
    });
  }

  async connect() {
    return new Promise((resolve, reject) => {
      this.imap.once('ready', resolve);
      this.imap.once('error', reject);
      this.imap.connect();
    });
  }

  async getJobEmails(days = 30) {
    const since = new Date();
    since.setDate(since.getDate() - days);
    
    return new Promise((resolve, reject) => {
      this.imap.openBox('INBOX', true, (err, box) => {
        if (err) return reject(err);
        
        this.imap.search(['UNSEEN', ['SINCE', since]], (err, results) => {
          if (err) return reject(err);
          
          if (results.length === 0) return resolve([]);
          
          const fetch = this.imap.fetch(results, { bodies: '' });
          const emails = [];
          
          fetch.on('message', (msg) => {
            msg.on('body', (stream) => {
              simpleParser(stream, (err, parsed) => {
                if (!err && this.isJobRelated(parsed)) {
                  emails.push(this.processEmail(parsed));
                }
              });
            });
          });
          
          fetch.once('end', () => resolve(emails));
        });
      });
    });
  }

  isJobRelated(email) {
    const content = `${email.subject} ${email.text}`.toLowerCase();
    const jobKeywords = [
      'application', 'interview', 'position', 'job', 'career',
      'hiring', 'recruitment', 'offer', 'candidate'
    ];
    return jobKeywords.some(keyword => content.includes(keyword));
  }

  processEmail(email) {
    // Categorize and extract job details
    return {
      messageId: email.messageId,
      from: email.from.text,
      subject: email.subject,
      body: email.text,
      date: email.date,
      category: this.categorizeEmail(email),
      companyName: this.extractCompany(email),
      position: this.extractPosition(email)
    };
  }
}
```

## Environment Variables
```env
# Server
PORT=3001
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/jobtracker

# Security
JWT_SECRET=your-jwt-secret
ENCRYPTION_KEY=your-encryption-key

# CORS
FRONTEND_URL=http://localhost:5173
```

## Deployment Considerations

1. **SSL/TLS**: Use HTTPS in production
2. **Rate Limiting**: Implement rate limiting for API endpoints
3. **Monitoring**: Add logging and error tracking
4. **Backup**: Regular database backups
5. **Scaling**: Consider using Redis for session management

## Getting Started

1. Create a new Node.js project
2. Install required dependencies
3. Set up database with the provided schema
4. Implement the API endpoints
5. Add proper error handling and validation
6. Test IMAP connections with various email providers
7. Deploy to your preferred hosting platform

This backend will provide secure, real email integration that the frontend can consume via API calls.