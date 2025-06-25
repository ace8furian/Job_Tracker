# JobTracker Pro

A comprehensive job application tracking system with real authentication and integrations.

## Features

- **Real Authentication**: Email/password registration and Google OAuth integration
- **Job Application Tracking**: Manage applications with status updates and notes
- **Email Integration**: Connect email accounts to automatically track job-related emails
- **Trello Integration**: Visual job application management with Trello boards
- **Job Search**: Search across multiple job portals with personalized recommendations
- **Profile Management**: Complete user profiles with skills and preferences

## Setup Instructions

### 1. Environment Configuration

Copy `.env.example` to `.env` and configure the following:

```bash
cp .env.example .env
```

### 2. Google OAuth Setup

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google+ API and Gmail API
4. Go to "Credentials" and create an OAuth 2.0 Client ID
5. Add your domain to authorized origins:
   - For development: `http://localhost:5173`
   - For production: your actual domain
6. Copy the Client ID to your `.env` file as `VITE_GOOGLE_CLIENT_ID`

### 3. Installation

```bash
npm install
```

### 4. Development

```bash
npm run dev
```

### 5. Production Build

```bash
npm run build
```

## Real Integrations

### Google OAuth
- Real Google sign-in with proper OAuth flow
- Access to Gmail API for email integration
- User profile information from Google

### Email Integration
- Requires backend server for IMAP connections
- Real email parsing and categorization
- Secure credential storage

### Trello Integration
- Real Trello API integration
- Create and manage boards, lists, and cards
- Sync job applications with Trello

## Security Features

- Secure password hashing
- Session management with expiration
- User data isolation
- OAuth token management
- Input validation and sanitization

## Architecture

- **Frontend**: React + TypeScript + Tailwind CSS
- **Authentication**: Google OAuth + Custom email auth
- **Storage**: LocalStorage (can be replaced with backend)
- **APIs**: Google APIs, Trello API, IMAP integration

## Configuration Notes

- All simulations have been removed
- Real API integrations require proper credentials
- Backend server needed for email integration
- Google OAuth requires domain verification for production

## Support

For issues with setup or configuration, please check:
1. Environment variables are correctly set
2. Google OAuth is properly configured
3. All required APIs are enabled
4. Domain is authorized in Google Console