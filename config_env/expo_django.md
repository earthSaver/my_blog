# Expo + Django
ensure your physical phone and computer are under the same wifi
## Django
### 1. set up launch.json
in .vscode/launch.json, for the runsever args,change the ip address to `0.0.0.0`
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Django",
            "type": "debugpy",
            "request": "launch",
            "program": "${workspaceFolder}\\manage.py",
            "args": [
                "runserver",
                "0.0.0.0:8001",
            ],
            "django": true,
            "justMyCode": true
        }
    ]
}
```

### 2. set up setting.py
in setting.py file, you need to set up CORS
```python
# for dev env
import os
import dotenv
dotenv.load_dotenv()

if os.environ.get('ENV') == "dev":
    CORS_ALLOW_ALL_ORIGINS = True

# for prod env, replace with your real domain
CORS_ALLOWED_ORIGINS = [
    'https://www.example.com',
]
```
### 3. get local ip address
run the project and get the local address `http://192.168.1.224:8001`, for example:
```text
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

October 17, 2024 - 15:50:53
Django version 5.1, using settings 'mysite.settings'
Starting development server at http://192.168.1.224:8001
Quit the server with CONTROL-C.
```
## Expo 
### 1. Set up url in env
in .env file, set up **EXPO_PUBLIC_API_URL (must use this name)**
```text
EXPO_PUBLIC_API_URL=http://192.168.1.224:8001
```
in eas.json, set up env variables in different environments: development, preview, production
```json
{
  "cli": {
    "version": ">= 12.5.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      },
      "env": {
        "EXPO_PUBLIC_API_URL": "http://192.168.1.224:8001"
      }
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      },
      "env": {
        "EXPO_PUBLIC_API_URL": "https://www.example.com"
      }
    },
    "production": {
      "autoIncrement": true,
      "android": {
        "buildType": "app-bundle"
      },
      "env": {
        "EXPO_PUBLIC_API_URL": "https://www.example.com"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```
### 2. Load url from env
for example, in apis.js file
```javascript
export const API_URL = process.env.EXPO_PUBLIC_API_URL;

const api = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
  withCredentials: true, // Important for CSRF cookies to be sent
  timeout: 10000,
});

export const getCsrfToken = async () => {
  try {
    // Fetch the CSRF token by making a request to the set-csrf endpoint
    const response = await api.get('/set-csrf/');
    
    // Extract CSRF token from response headers
    const setCookieHeader = response.headers['set-cookie'];

    // Find the CSRF token from Set-Cookie header
    const csrfToken = setCookieHeader.find(cookie => cookie.startsWith('csrftoken='))?.split(';')[0].split('=')[1] || null;
    
    if (csrfToken) {
      // Store the CSRF token in AsyncStorage
      await AsyncStorage.setItem('csrfToken', csrfToken);
      console.log('CSRF token set successfully');
    } else {
      console.error('CSRF token not found in response');
    }
  } catch (error) {
    console.error('Error setting CSRF token:', error);
  }
};
```