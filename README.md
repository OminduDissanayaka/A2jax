# A2jax

A secure, lightweight JavaScript library for making HTTP requests with built-in security features and flexible configuration options.

## Features

- 🔒 **Multi-level Security**: Three security levels (low, medium, high) with automatic input sanitization
- 🛡️ **CSRF Protection**: Built-in CSRF token handling for secure requests
- ⚡ **Rate Limiting**: Configurable request rate limiting to prevent abuse
- 🔑 **API Key Support**: Secure API key authentication for high-security applications
- ⏱️ **Timeout Management**: Configurable request timeouts with automatic handling
- 🌐 **Base URL Support**: Set a base URL for all requests
- 📝 **Input Sanitization**: Automatic sanitization of request data based on security level
- 🚫 **Request Size Limits**: Configurable limits to prevent oversized requests

## Installation

```bash
npm install a2jax
```

Or include directly in your HTML:

```html
<script src="https://cdn.jsdelivr.net/npm/a2jax@latest/dist/a2jax.min.js"></script>
```

## Quick Start

```javascript
// Basic usage with default medium security
const xhr = new A2jax();

// Make a GET request
xhr.get('/api/users')
    .then(response => {
        console.log('Users:', response.data);
    })
    .catch(error => {
        console.error('Error:', error);
    });

// Make a POST request
xhr.post('/api/users', {
    name: 'John Doe',
    email: 'john@example.com'
})
    .then(response => {
        console.log('User created:', response.data);
    });
```

## Configuration

### Basic Configuration

```javascript
const xhr = new A2jax({
    security: true,
    securityLevel: 'high',
    timeout: 15000,
    baseURL: 'https://api.example.com',
    apiKey: 'your-api-key-here'
});
```

### Security Levels

| Level | Features |
|-------|----------|
| **Low** | Basic input sanitization, no CSRF protection, no rate limiting |
| **Medium** (Default) | Input sanitization, CSRF protection, rate limiting (10 req/sec), SSL validation, 1MB request limit |
| **High** | All medium features + enhanced rate limiting (5 req/sec), 512KB limit, API key requirement, session validation |

### Configuration Methods

```javascript
// Method 1: During initialization
const xhr = new A2jax({
    security: true,
    securityLevel: 'high',
    apiKey: 'your-api-key'
});

// Method 2: Using configuration methods
const xhr2 = new A2jax();
xhr2.setSecurity(true, 'medium')
    .setApiKey('your-api-key')
    .setBaseURL('https://api.example.com');

// Method 3: Static factory method
const xhr3 = A2jax.create({
    security: true,
    securityLevel: 'high',
    timeout: 20000
});
```

## API Reference

### GET Requests

```javascript
// Basic GET request
xhr.get('/api/users')
    .then(response => console.log(response.data));

// GET with custom headers
xhr.get('/api/users/123', {
    headers: {
        'Authorization': 'Bearer your-token',
        'Accept': 'application/json'
    }
})
    .then(response => console.log(response.data));

// GET with query parameters
const params = new URLSearchParams({ include: 'profile,settings' });
xhr.get(`/api/users/123?${params}`)
    .then(response => console.log(response.data));
```

### POST Requests

```javascript
// Basic POST request
xhr.post('/api/users', {
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
})
    .then(response => console.log(response.data));

// POST with custom headers
xhr.post('/api/login', {
    username: 'johndoe',
    password: 'securepassword'
}, {
    headers: {
        'Content-Type': 'application/json',
        'X-Client-Version': '1.0.0'
    }
})
    .then(response => {
        localStorage.setItem('authToken', response.data.token);
    });
```

## Advanced Usage

### API Client Pattern

```javascript
class APIClient {
    constructor(baseURL, apiKey) {
        this.client = new A2jax({
            security: true,
            securityLevel: 'high',
            baseURL: baseURL,
            apiKey: apiKey,
            timeout: 30000,
            defaultHeaders: {
                'Accept': 'application/json',
                'User-Agent': 'MyApp/1.0'
            }
        });
    }

    async getUsers() {
        try {
            const response = await this.client.get('/users');
            return response.data;
        } catch (error) {
            throw new Error(`Failed to fetch users: ${error.error}`);
        }
    }

    async createUser(userData) {
        try {
            const response = await this.client.post('/users', userData);
            return response.data;
        } catch (error) {
            if (error.status === 400) {
                throw new Error('Invalid user data provided');
            }
            throw new Error(`Failed to create user: ${error.error}`);
        }
    }
}

// Usage
const api = new APIClient('https://api.myapp.com', 'your-api-key');
const users = await api.getUsers();
```

### Error Handling

```javascript
async function handleAPICall() {
    try {
        const response = await xhr.post('/api/sensitive-data', {
            data: 'sensitive information'
        });
        
        console.log('Success:', response.data);
        
    } catch (error) {
        if (error.error === 'Rate limit exceeded. Please slow down your requests.') {
            console.warn('Too many requests, waiting before retry...');
            setTimeout(() => handleAPICall(), 2000);
            
        } else if (error.error === 'Request timeout') {
            console.error('Request timed out, check your connection');
            
        } else if (error.status === 413) {
            console.error('Request too large, reduce data size');
            
        } else if (error.status >= 500) {
            console.error('Server error, try again later');
            
        } else {
            console.error('Request failed:', error);
        }
    }
}
```

### Retry with Exponential Backoff

```javascript
async function retryRequest(url, data, maxRetries = 3) {
    const xhr = new A2jax({
        security: true,
        securityLevel: 'medium'
    });

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await xhr.post(url, data);
            return response;
            
        } catch (error) {
            if (attempt === maxRetries) {
                throw error;
            }
            
            // Wait with exponential backoff
            const delay = Math.pow(2, attempt) * 1000;
            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}
```

## Environment-Specific Configuration

### Development Environment

```javascript
const devXhr = new A2jax({
    security: true,
    securityLevel: 'low',
    baseURL: 'http://localhost:3000/api',
    timeout: 5000
});
```

### Production Environment

```javascript
const prodXhr = new A2jax({
    security: true,
    securityLevel: 'high',
    baseURL: 'https://secure-api.production.com',
    apiKey: process.env.API_KEY,
    timeout: 15000,
    defaultHeaders: {
        'X-App-Version': '2.1.0',
        'X-Client-ID': 'web-client'
    }
});
```

## Common Patterns

### Form Submission

```javascript
async function submitContactForm(formData) {
    const xhr = new A2jax({
        security: true,
        securityLevel: 'medium'
    });

    try {
        const response = await xhr.post('/api/contact', {
            name: formData.name,
            email: formData.email,
            message: formData.message,
            timestamp: Date.now()
        });

        showNotification('Message sent successfully!', 'success');
        return response.data;

    } catch (error) {
        showNotification('Failed to send message. Please try again.', 'error');
        throw error;
    }
}
```

### Loading Multiple Resources

```javascript
async function loadUserDashboard(userId) {
    const xhr = new A2jax({
        security: true,
        securityLevel: 'high',
        apiKey: getAuthToken()
    });

    showLoadingSpinner(true);

    try {
        const [userResponse, statsResponse] = await Promise.all([
            xhr.get(`/api/users/${userId}`),
            xhr.get(`/api/users/${userId}/stats`)
        ]);

        updateUserProfile(userResponse.data);
        updateUserStats(statsResponse.data);

    } catch (error) {
        if (error.status === 401) {
            redirectToLogin();
        } else {
            showErrorMessage('Failed to load dashboard data');
        }
    } finally {
        showLoadingSpinner(false);
    }
}
```

## Security Features

### Input Sanitization
A2jax automatically sanitizes input data based on the configured security level, preventing XSS attacks and other malicious input.

### CSRF Protection
Medium and high security levels include automatic CSRF token handling to prevent cross-site request forgery attacks.

### Rate Limiting
Built-in rate limiting prevents abuse:
- Medium security: 10 requests per second
- High security: 5 requests per second

### Request Size Limits
Configurable limits prevent oversized requests:
- Medium security: 1MB limit
- High security: 512KB limit

## Browser Support

A2jax supports all modern browsers that support:
- Promises
- Fetch API or XMLHttpRequest
- ES6 features

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, please open an issue on GitHub or contact the maintainers.

---

**A2jax** - Secure HTTP requests made simple.