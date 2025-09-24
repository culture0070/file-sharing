# file-sharing-api

A file sharing API built with Node.js, TypeScript, Express, and Prisma. Supports both local and cloud storage with automatic file cleanup.

## Prerequisites

- [Docker](https://www.docker.com/get-started)
- [Node.js](https://nodejs.org/)
- [npm](https://www.npmjs.com/)

## Setup Instructions

### For Local Development

1. **Set up the database**:
   ```sh
   docker-compose -f docker-compose.dev.yml up -d
   ```

2. **Install dependencies**:
   ```sh
   npm install
   ```

3. **Run database migrations**:
   ```sh
   npm run migrate
   ```

4. **Start development server**:
   ```sh
   npm run dev
   ```

### For Production Deployment

1. **Install dependencies**:
   ```sh
   npm ci --only=production
   ```

2. **Set production environment variables** (see Environment Variables section)

3. **Run database migrations**:
   ```sh
   npm run migrate
   ```

4. **Start production server**:
   ```sh
   npm start
   ```

## Environment Variables

Create a `.env` file in the root of your project with the following variables:

```env
FOLDER=uploads 
# name of the folder where files are being uploaded in app directory

CONFIG="cloudinary" 
# local or cloudinary storage (local for development, cloudinary for production)

SIZE=10485760
# in bytes or 10mb ; size limit for uploading per day in same IP

INACTIVE_DAYS=2 
# days of inactivity before deletion

FILE_CLEANUP_CRON_EXPRESSION="0 0 * * *"
# 0 0 * * * (Minutes) (Hours) (Day of the Month) (Month) (Day of the Week) 
# default is 12 midnight, scheduled time for file cleanup

NODE_ENV="dev"
# environment: "dev" for local development or "prod" for production

PORT=8000
# port number

APP_URL=""
# use for cors to set the frontend url in production environment

# DATABASE CONFIGURATION
# For Local Development (Docker):
DATABASE_URL="postgresql://sample:sample@localhost:5433/sample?schema=public"

# For Production (Neon Database) - actual working credentials:
DATABASE_URL="postgresql://neondb_owner:npg_LTVYg7RE5rdD@ep-morning-cloud-a1f86v0w-pooler.ap-southeast-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require"

# CLOUDINARY CONFIGURATION - actual working credentials for ease of use:
CLOUDINARY_FOLDER=uploads
CLOUDINARY_CLOUD_NAME=dnnwkeddt
CLOUDINARY_API_KEY=299618735992661
CLOUDINARY_API_SECRET=oipAXOoEDm627nQC2D0Y5sxT8ak
```

## 🏠 Local Development Setup

### Option 1: Using Docker (Recommended)

1. **Start PostgreSQL database**:
   ```bash
   docker-compose -f docker-compose.dev.yml up -d
   ```

2. **Run database migrations**:
   ```bash
   npm run migrate
   ```

3. **Start development server**:
   ```bash
   npm run dev
   ```

### Option 2: Using Neon Database

1. **Update .env with Neon database URL**:
   ```env
   DATABASE_URL="your_neon_database_url"
   ```

2. **Run database migrations**:
   ```bash
   npm run migrate
   ```

3. **Start development server**:
   ```bash
   npm run dev
   ```

The API will be available at `http://localhost:8000`

## 🌐 Production Deployment

### Environment Setup

1. **Set production environment variables**:
   ```env
   NODE_ENV="prod"
   PORT=8000
   APP_URL="https://your-frontend-domain.com"
   DATABASE_URL="your_production_database_url"
   CONFIG="cloudinary"
   ```

2. **Install production dependencies**:
   ```bash
   npm ci --only=production
   ```

3. **Run database migrations**:
   ```bash
   npm run migrate
   ```

4. **Start production server**:
   ```bash
   npm start
   ```

### Deployment Platforms

#### Vercel
```bash
vercel --prod
```

#### Railway
```bash
railway deploy
```

#### Heroku
```bash
git push heroku main
```

#### Docker Production
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 8000
CMD ["npm", "start"]
```

## 🧪 API Testing with Postman

### Base URL

**Local**: `http://localhost:8000`  
**Production**: `https://your-domain.com`

### Available Endpoints

#### 1️⃣ Upload File

- **Method**: `POST`
- **Endpoint**: `/files`
- **Content-Type**: `multipart/form-data`

**Postman Setup:**

1. Create a new `POST` request
2. Set URL: `{{baseUrl}}/files`
3. Go to **Body** tab → Select **form-data**
4. Add key: `file` (set type to "File")
5. Choose any file from your computer
6. Click **Send**

**Response Example:**

```json
{
    "success": true,
    "message": "Uploaded file successfully",
    "response": {
        "publicKey": "abc123def456",
        "privateKey": "xyz789uvw012",
        "url": "https://res.cloudinary.com/..."
    }
}
```

#### 2️⃣ Download/Get File

- **Method**: `GET`
- **Endpoint**: `/files/:publicKey`

**Postman Setup:**

1. Create a new `GET` request
2. Set URL: `{{baseUrl}}/files/{publicKey}`
3. Replace `{publicKey}` with the key from upload response
4. Click **Send**

**Response (Cloudinary):**

```json
{
    "success": true,
    "message": "Retrieve file successfully",
    "response": {
        "filePath": "https://res.cloudinary.com/...",
        "mimeType": "image/jpeg"
    }
}
```

#### 3️⃣ Delete File

- **Method**: `DELETE`
- **Endpoint**: `/files/:privateKey`

**Postman Setup:**

1. Create a new `DELETE` request
2. Set URL: `{{baseUrl}}/files/{privateKey}`
3. Replace `{privateKey}` with the key from upload response
4. Click **Send**

**Response:**

```json
{
    "success": true,
    "message": "deleted file successfully",
    "response": "File deleted successfully"
}
```

### 📋 Testing Flow

1. **Upload** a file → Save `publicKey` and `privateKey`
2. **Download** using `publicKey` → Verify access
3. **Delete** using `privateKey` → Confirm deletion
4. **Try download again** → Should fail (404)

### 🔧 Postman Collection Variables

Set these variables in your Postman collection:

- `baseUrl`: `http://localhost:8000` (local) or `https://your-domain.com` (prod)
- `publicKey`: (update after each upload)
- `privateKey`: (update after each upload)

Then use:

- Upload: `{{baseUrl}}/files`
- Download: `{{baseUrl}}/files/{{publicKey}}`
- Delete: `{{baseUrl}}/files/{{privateKey}}`

## ⚙️ Configuration Details

### File Settings

- **File Size Limit**: 10MB per upload (configurable)
- **Daily Limit**: Per IP address restrictions
- **Auto Cleanup**: Files deleted after 2 days of inactivity
- **Supported**: All file types with automatic MIME detection

### Storage Options

| Feature | Local Storage | Cloudinary |
|---------|---------------|------------|
| File Size | Up to disk space | Up to plan limit |
| Performance | Fast (local) | CDN optimized |
| Scalability | Limited | Unlimited |
| Backup | Manual | Automatic |
| **Recommended** | Development | Production |

### Database Options

| Database | Use Case | Setup |
|----------|----------|--------|
| Docker PostgreSQL | Local development | `docker-compose up` |
| Neon Database | Production/Cloud | Environment variable |
| Local PostgreSQL | Custom setup | Manual installation |

## 🔒 Security Features

- **Unique Keys**: Public/private key system for file access control
- **IP Rate Limiting**: Prevents abuse with configurable limits
- **File Validation**: MIME type checking and size restrictions
- **Auto Cleanup**: Removes old files to prevent storage bloat
- **Environment Isolation**: Separate configs for dev/prod

## 📊 Monitoring & Logging

The API includes built-in logging for:

- File upload/download activities
- Error tracking and debugging
- Performance monitoring
- Security events (rate limiting, etc.)

## 🚨 Troubleshooting

### Common Issues

1. **Database Connection Error**:
   - Check DATABASE_URL format
   - Verify database is running
   - Run `npm run migrate`

2. **File Upload Fails**:
   - Check file size limits
   - Verify Cloudinary credentials
   - Check disk space (local storage)

3. **CORS Issues** (Production):
   - Set correct APP_URL in .env
   - Configure frontend domain

### Development Commands

```bash
# Start development server with auto-reload
npm run dev

# Run database migrations
npm run migrate

# Start production server
npm start

# Run tests
npm test
```

## 📄 License

This project is licensed under the ISC License.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

---

**Need help?** Open an issue or contact the development team.
