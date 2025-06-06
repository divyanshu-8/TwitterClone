# Twitter Clone - Technical Documentation
## Project Overview
This project is a Twitter clone built using Next.js, MongoDB, and NextAuth.js. It aims to replicate the core functionality of Twitter, allowing users to post tweets, follow other users, like posts, and engage in conversations.
**Key Features:**
*   **User Authentication:** Secure user authentication using NextAuth.js with Google Provider.
*   **Tweet Posting:** Users can create and post tweets with text and images.
*   **Following/Followers:** Users can follow other users to see their tweets in their home timeline.
*   **Likes:** Users can like tweets.
*   **Replies:** Users can reply to tweets, creating threaded conversations.
*   **Profile Management:** Users can update their profile information, including avatar, cover photo, bio, and username.
*   **Image Uploads:** Users can upload images to accompany their tweets and profile information.
**Supported Platforms/Requirements:**
*   Node.js (v16 or higher)
*   npm or yarn
*   MongoDB database
*   Google Cloud Console project with OAuth 2.0 Client ID and Secret
*   AWS S3 bucket for image storage
## Getting Started
### Installation
1.  **Clone the repository:**
    ```bash
    git clone <repository_url>
    ```
    
2.  **Navigate to the project directory:**
    ```bash
    cd twitter-clone
    ```
    
3.  **Install dependencies:**
    ```bash
    npm install
    # or
    yarn install
    ```
    
### Configuration
1.  **Environment Variables:** Create a `.env.local` file in the project root and add the following environment variables:
   
    *   `MONGODB_URI`:  The connection string for your MongoDB database.
    *   `GOOGLE_CLIENT_ID`: Your Google OAuth 2.0 Client ID.  Obtain this from the Google Cloud Console.
    *   `GOOGLE_CLIENT_SECRET`: Your Google OAuth 2.0 Client Secret. Obtain this from the Google Cloud Console.
    *   `NEXTAUTH_SECRET`: A random string used to encrypt the NextAuth.js JWT.  Generate a secure random string.
    *   `NEXTAUTH_URL`: The URL of your application.  This is important for production deployments.
    *   `S3_ACCESS_KEY`: Your AWS S3 access key.
    *   `S3_SECRET_ACCESS_KEY`: Your AWS S3 secret access key.
2.  **Google OAuth Setup:**
    *   Go to the Google Cloud Console ([https://console.cloud.google.com/](https://console.cloud.google.com/)).
    *   Create a new project or select an existing one.
    *   Enable the Google People API.
    *   Create OAuth 2.0 credentials.
    *   Configure the Authorized JavaScript origins and Authorized redirect URIs.  For development, use `http://localhost:3000`.  For production, use your deployed URL.
3.  **AWS S3 Setup:**
    *   Create an AWS S3 bucket.
    *   Configure the bucket for public read access (for images).
    *   Create an IAM user with permissions to upload objects to the bucket.
    *   Obtain the access key and secret access key for the IAM user.
### Running the Application
1.  **Start the development server:**
    ```bash
    npm run dev
    # or
    yarn dev
    ```
    
2.  **Open your browser and navigate to `http://localhost:3000`.**
**Key Components:**
*   **`components`:** Contains reusable React components for building the user interface.
*   **`hooks`:** Contains custom React hooks for managing state and logic.  `useUserInfo` is a key hook for fetching and providing user data.
*   **`lib`:** Contains utility functions for database connections (MongoDB and Mongoose).
*   **`models`:** Defines the Mongoose schemas for the database models (User, Post, Like, Follower).
*   **`pages`:** Contains the Next.js pages and API routes.  Dynamic routes (e.g., `[username].js`, `[id].js`) are used for user profiles and individual posts.
*   **`api`:** Contains the API routes that handle backend logic, such as authentication, data fetching, and data manipulation.
## API Documentation
### `api/auth/[...nextauth].js`
*   **Purpose:** NextAuth.js configuration for handling authentication.
*   **Providers:** Google Provider is configured for authentication.
*   **Adapter:** MongoDBAdapter is used to store user sessions in MongoDB.
*   **Callbacks:** The `session` callback is used to add the user ID to the session object.
### `api/users.js`
*   **Purpose:** Manages user data (fetching and updating).
*   **Methods:**
    *   **`PUT`:** Updates the username for a user.
        *   **Input:** `username` in the request body (JSON).
        *   **Output:** `"ok"` (JSON).
    *   **`GET`:** Retrieves user information.
        *   **Input:** `id` or `username` in the query parameters.
        *   **Output:**
            *   `user`: User object (JSON).
            *   `follow`: Follower object if the current user follows the requested user (JSON), otherwise `null`.
    **Example GET Request:**
        /api/users?username=testuser
    
    **Example GET Response:**
    ```json
    {
      "user": {
        "_id": "64d1234567890abcdef12345",
        "name": "Test User",
        "email": "test@example.com",
        "image": "https://example.com/image.jpg",
        "cover": null,
        "bio": null,
        "username": "testuser",
        "__v": 0
      },
      "follow": null
    }
    
### `api/posts.js`
*   **Purpose:** Manages posts (fetching and creating).
*   **Methods:**
    *   **`GET`:** Retrieves posts.
        *   **Input:**
            *   `id`:  Retrieves a specific post by ID.
            *   `parent`: Retrieves replies to a specific post (used for comments).
            *   `author`: Retrieves posts by a specific author.
            *   If none of the above are provided, retrieves posts for the home timeline (posts from users the current user follows).
        *   **Output:**
            *   `post`: Post object (JSON) if `id` is provided.
            *   `posts`: Array of post objects (JSON) if `parent` or `author` is provided, or for the home timeline.
            *   `idsLikedByMe`: Array of post IDs that the current user has liked (JSON).
    *   **`POST`:** Creates a new post.
        *   **Input:** `text`, `parent` (optional), and `images` (optional) in the request body (JSON).
        *   **Output:** The created post object (JSON).
    **Example POST Request:**
    ```json
    {
      "text": "This is a new tweet!",
      "images": ["https://example.com/image1.jpg", "https://example.com/image2.jpg"]
    }
    
    **Example GET Request (Home Timeline):**
        /api/posts
    
### `api/like.js`
*   **Purpose:** Handles liking and unliking posts.
*   **Method:**
    *   **`POST`:** Likes or unlikes a post.
        *   **Input:** `id` (post ID) in the request body (JSON).
        *   **Output:**
            *   `null` (JSON) if the post was unliked.
            *   `{like: <like_object>}` (JSON) if the post was liked.
### `api/followers.js`
*   **Purpose:** Handles following and unfollowing users.
*   **Method:**
    *   **`POST`:** Follows or unfollows a user.
        *   **Input:** `destination` (user ID to follow/unfollow) in the request body (JSON).
        *   **Output:**
            *   `null` (JSON) if the user was unfollowed.
            *   `Follower` object (JSON) if the user was followed.
### `api/profile.js`
*   **Purpose:** Updates user profile information.
*   **Method:**
    *   **`PUT`:** Updates the user's bio, name, and username.
        *   **Input:** `bio`, `name`, and `username` in the request body (JSON).
        *   **Output:** `"ok"` (JSON).
### `api/upload.js`
*   **Purpose:** Handles file uploads (images).
*   **Method:**
    *   **`POST`:** Uploads an image to AWS S3.
        *   **Input:**  `cover` or `image` file in the request body (multipart/form-data).
        *   **Output:**
            *   `files`: Information about the uploaded file.
            *   `err`: Error object if an error occurred.
            *   `data`: AWS S3 upload response data.
            *   `fileInfo`: Information about the uploaded file.
            *   `src`: URL of the uploaded image in S3 (JSON).
