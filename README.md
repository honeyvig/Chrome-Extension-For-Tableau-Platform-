# Chrome-Extension-For-Tableau-Platform
 create a secure and seamless solution for generating Tableau tokens within a Chrome extension. This extension will be used internally to streamline access and authentication for the Tableau platform.

- Develop a Chrome extension that securely generates and manages Tableau tokens for internal use.
- Integrate with Tableau’s REST API to automate token creation and management processes.
- Ensure secure handling and storage of authentication credentials and tokens.
- Implement user-friendly UI/UX for token generation and status display within the Chrome extension.
- Enable communication between the extension and Tableau’s internal platform for seamless access.
- Ensure compliance with OAuth 2.0 and Tableau’s authentication protocols.
- Conduct thorough testing and debugging to ensure performance and security.
- Collaborate with internal stakeholders to gather requirements and deliver the solution.

Technical Requirements:
- Proficient in JavaScript, HTML, and CSS for Chrome extension development.
- Experience with Chrome Extension APIs, including messaging and storage capabilities.
- Strong understanding of REST APIs and HTTP request/response handling.
- Knowledge of authentication protocols (e.g., OAuth 2.0, JWT) and token management.
- Familiarity with Tableau’s REST API for token generation.
Experience with security practices for handling sensitive data (encryption, storage).
- Optional: Experience with Node.js or back-end tools to complement front-end token generation.
------------
Creating a Chrome extension for generating Tableau tokens securely requires a few key components, including interacting with Tableau's REST API, handling OAuth 2.0 authentication, and storing the generated tokens securely.

Below is a breakdown of how you can build this Chrome extension, including the technical requirements and code snippets for each step.
Step-by-Step Breakdown
1. Manifest File (manifest.json)

The manifest.json file defines the extension's settings and permissions.

{
  "manifest_version": 3,
  "name": "Tableau Token Generator",
  "description": "Generate Tableau tokens securely.",
  "version": "1.0",
  "permissions": [
    "storage",
    "identity",
    "https://*.tableau.com/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html"
  },
  "oauth2": {
    "client_id": "YOUR_TABLEAU_CLIENT_ID",
    "scopes": ["openid", "view", "write"]
  },
  "host_permissions": [
    "https://*.tableau.com/*"
  ]
}

2. Background Script (background.js)

This background script will handle authentication with Tableau's API using OAuth 2.0 and generate the Tableau token.

// background.js

const TABLEAU_AUTH_URL = 'https://YOUR_TABLEAU_INSTANCE_URL/api/3.9/auth/signin';
const TABLEAU_LOGOUT_URL = 'https://YOUR_TABLEAU_INSTANCE_URL/api/3.9/auth/signout';
const CLIENT_ID = 'YOUR_TABLEAU_CLIENT_ID';
const CLIENT_SECRET = 'YOUR_TABLEAU_CLIENT_SECRET';

chrome.runtime.onInstalled.addListener(() => {
  console.log('Tableau Token Generator Extension Installed');
});

// Handle OAuth2 Authentication Flow
chrome.identity.getAuthToken({ interactive: true }, function (token) {
  if (chrome.runtime.lastError || !token) {
    console.error('Error obtaining OAuth2 token:', chrome.runtime.lastError);
    return;
  }

  // Use the OAuth2 token to authenticate with Tableau
  authenticateWithTableau(token);
});

// Authenticate with Tableau API
async function authenticateWithTableau(oauthToken) {
  try {
    const response = await fetch(TABLEAU_AUTH_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${oauthToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        credentials: {
          name: CLIENT_ID,
          password: CLIENT_SECRET
        }
      })
    });

    if (!response.ok) {
      throw new Error('Tableau authentication failed');
    }

    const data = await response.json();
    const tableauToken = data.credentials.token;
    storeToken(tableauToken);
  } catch (error) {
    console.error('Error authenticating with Tableau:', error);
  }
}

// Store the Tableau token securely using Chrome storage
function storeToken(token) {
  chrome.storage.local.set({ tableauToken: token }, function () {
    console.log('Tableau token stored securely.');
  });
}

// Logout function
async function logoutFromTableau() {
  const tableauToken = await getStoredToken();
  if (!tableauToken) {
    console.error('No token found to log out');
    return;
  }

  try {
    const response = await fetch(TABLEAU_LOGOUT_URL, {
      method: 'POST',
      headers: {
        'X-Tableau-Auth': tableauToken
      }
    });

    if (response.ok) {
      chrome.storage.local.remove('tableauToken');
      console.log('Logged out from Tableau');
    } else {
      console.error('Failed to log out from Tableau');
    }
  } catch (error) {
    console.error('Error logging out from Tableau:', error);
  }
}

// Fetch the stored token
function getStoredToken() {
  return new Promise((resolve, reject) => {
    chrome.storage.local.get('tableauToken', function (data) {
      if (chrome.runtime.lastError) {
        reject(chrome.runtime.lastError);
      } else {
        resolve(data.tableauToken);
      }
    });
  });
}

3. Popup HTML (popup.html)

This is the UI for the extension that users interact with to generate the token and display the status.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tableau Token Generator</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; }
    button { padding: 10px 20px; margin: 10px; }
    #status { margin-top: 20px; }
  </style>
</head>
<body>
  <h1>Tableau Token Generator</h1>
  <button id="generateToken">Generate Token</button>
  <button id="logout">Logout</button>
  <div id="status"></div>

  <script src="popup.js"></script>
</body>
</html>

4. Popup Script (popup.js)

The popup script handles UI interactions and sends messages to the background script.

// popup.js

document.getElementById('generateToken').addEventListener('click', function () {
  chrome.runtime.sendMessage({ action: 'generateToken' }, function (response) {
    document.getElementById('status').textContent = response.status;
  });
});

document.getElementById('logout').addEventListener('click', function () {
  chrome.runtime.sendMessage({ action: 'logout' }, function (response) {
    document.getElementById('status').textContent = response.status;
  });
});

5. OAuth Flow and Token Management

The extension uses Chrome's identity API to trigger the OAuth 2.0 flow. When the user clicks on the extension icon, the app uses chrome.identity.getAuthToken to request an OAuth token. This token is used to authenticate against Tableau's API to generate a Tableau token.

Once the Tableau token is generated, it is stored securely using chrome.storage.local, ensuring the token is persisted across sessions.
6. Security Considerations

    Encryption: For further security, you can implement additional encryption when storing the token using chrome.storage.local. You can use libraries like crypto-js for encryption.

    Token Expiry: Implement token expiry handling, where you check the expiration of the stored Tableau token and refresh it automatically.

7. Testing & Debugging

    Thoroughly test the extension with valid and invalid credentials.
    Ensure proper handling of errors and edge cases, like expired tokens, API downtime, and network failures.
    Use chrome.storage.local to persist tokens and handle scenarios like token refresh or re-authentication.

Final Notes:

    Google OAuth Integration: This extension integrates with Google’s OAuth 2.0, so you'll need to configure your Google API Console to allow OAuth access to the Tableau API.
    Tableau REST API: The integration with Tableau's REST API allows you to generate and manage tokens securely. The token is stored locally for future requests.
    Token Refresh: Consider adding token refresh logic to automatically refresh the token once it expires. This is important for long-lived sessions.
    Secure Handling: Ensure that the tokens are handled securely and avoid any sensitive information from being exposed to unauthorized users.

This extension will enable internal teams to generate and manage Tableau tokens securely, streamlining the authentication and access process for Tableau dashboards.
