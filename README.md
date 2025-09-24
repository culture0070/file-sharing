# file-sharing-api

### Steps


### Prerequisites

-   [Docker](https://www.docker.com/get-started)
-   [Node.js](https://nodejs.org/)
-   [npm](https://www.npmjs.com/)

1. **Set up the database**:

    ```sh
    docker-compose -f docker-compose.dev.yml up
    ```

        ```

2. **Run the application**:

    ```sh
    npm install
    npm run migrate
    npm run dev/npm run start
    ```



## Environment Variables

Create a `.env` file in the root of your project with the following variables:

<!-- I added my values already for ease of use -->

FOLDER=uploads 
#  name of the folder where files are being uploaded in app directory

CONFIG="cloudinary" 
#  local or  specific cloud provider  google, aws, azure, cloudinary. But for now only local and cloudinary is supported

SIZE=10485760
# in bytes or 10mb ; size limit for uploading per day in same IP

INACTIVE_DAYS=2 
# days of inactivity before deletion

FILE_CLEANUP_CRON_EXPRESSION=0 0 * * *
#   0  0 * * *   (Minutes) (Hours) (Day of the Month) (Month) (Day of the Week) default is 12 midnight, scheduled time for file cleanup

NODE_ENV="dev"
#  just for dynamic environment dev or prod

PORT=8000
# port

APP_URL=""
#  use for cors to set the fe url in production environment

DATABASE_URL="postgresql://jonix:jonix@13.210.89.55:5433/jonix?schema=public" 
<!-- Please use this database if you are having a hard time setting up docker or local db-->
# db url you can use my deployed db server for ease of use


# I use cloudinary instead,  because I can't use google cloud
#  you can use my cloudinary account creds
CLOUDINARY_FOLDER=uploads
CLOUDINARY_CLOUD_NAME=dnnwkeddt
CLOUDINARY_API_KEY=299618735992661
CLOUDINARY_API_SECRET=oipAXOoEDm627nQC2D0Y5sxT8ak

## 🧪 API Testing with Postman

### Base URL

```text
http://localhost:8000
```

### Available Endpoints

#### 1️⃣ Upload File

- **Method**: `POST`
- **Endpoint**: `/files`
- **Content-Type**: `multipart/form-data`

**Postman Setup:**

1. Create a new `POST` request
2. Set URL: `http://localhost:8000/files`
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
2. Set URL: `http://localhost:8000/files/{publicKey}`
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
2. Set URL: `http://localhost:8000/files/{privateKey}`
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

### ⚙️ Configuration Notes

- **File Size Limit**: 10MB per upload
- **Daily Limit**: Per IP address restrictions
- **Auto Cleanup**: Files deleted after 2 days of inactivity
- **Storage**: Cloudinary (cloud) or Local (based on CONFIG)
- **Supported**: All file types with automatic MIME detection

### 🔧 Postman Collection Variables (Optional)

Set these variables in your Postman collection for easier testing:

- `baseUrl`: `http://localhost:8000`
- `publicKey`: (update after each upload)
- `privateKey`: (update after each upload)

Then use:

- Upload: `{{baseUrl}}/files`
- Download: `{{baseUrl}}/files/{{publicKey}}`
- Delete: `{{baseUrl}}/files/{{privateKey}}`
