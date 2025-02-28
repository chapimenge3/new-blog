---
title: "Simple JWT Authentication with Cloudflare Workers"
description: "Learn how to implement secure JWT authentication in Cloudflare Workers using a custom TypeScript class."
slug: "simple-jwt-authentication-with-cloudflare-workers"
meta_title: ""
date: 2025-02-28 10:23:39
image: "/images/blogs/programming/cf-jwt.jpg"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["programming", "cloudflare", "jwt", "typescript"]
draft: false
---

If you ever think of auth in Cloudflare workers and you got discouraged by the lack of documentation or the complexity of CF worker runtime, then this blog is for you. I will show you how to implement a simple JWT authentication in Cloudflare Workers using a custom TypeScript class.

{{< toc >}}

# Simple JWT Authentication with Cloudflare Workers

Hey Guys, I am back with another blog. Today, I want to share a solution that saved me a lot of headaches when implementing authentication in Cloudflare Workers. If you've ever tried to handle authentication in Workers, you know it can be quite challenging since the environment lacks many Node.js modules we typically rely on.


**TL;DR**: I've created a custom TypeScript class that handles JWT authentication in Cloudflare Workers. It uses the Web Crypto API and provides a simple interface for signing and verifying tokens. You can find the full source code on [GitHub](https://github.com/chapimenge3/cloudflare-jwt-auth). If you want to use the class in your project just copy the `auth.ts` file from the repository and see how to use it in the example below.

Before we dive into the solution if you want to check the hosted version of the project, you can check it out [here](https://https://cf-jwt.builds.et/).

{{< button label="Hosted Sample Site" link="https://cf-jwt.builds.et/" style="solid" >}}


## The Challenge

Working with Cloudflare Workers for authentication is notoriously difficult because:

1. Workers run in a V8 isolate environment, not Node.js
2. No access to standard Node.js crypto modules
3. Limited compatibility with existing JWT libraries
4. Need to use Web Crypto API which has a different interface

I was building a Next.js application that needed secure authentication with Cloudflare Workers as the backend, and after much trial and error, I created a lightweight JWT authentication solution that actually works.

## The Solution: A Custom JWT Authentication Class

I've created a TypeScript class that:

- Uses the Web Crypto API (available in Workers)
- Handles JWT signing and verification
- Supports configurable expiration times
- Provides proper error handling

## Let's Start with the Interface

```typescript
interface JWTOptions {
    expiresIn?: string | number;
}

interface JWTHeader {
    alg: string;
    typ: string;
}

interface JWTPayload {
    [key: string]: unknown;
    iat?: number;
    exp?: number;
}
```

These interfaces define the structure of our JWT tokens and the options we can pass when creating tokens.

## The Core Class

Here's the main class that handles JWT operations:

```typescript
class JWTAuth {
    private secret: string;
    private encoder: TextEncoder;
    private decoder: TextDecoder;

    constructor(secret: string) {
        if (!secret || typeof secret !== 'string') {
            throw new Error('Secret is required and must be a string');
        }
        this.secret = secret;
        this.encoder = new TextEncoder();
        this.decoder = new TextDecoder();
    }
    
    // Methods will be shown below...
}
```

## Creating JWTs (sign method)

The sign method creates a new JWT token with the provided payload:

```typescript
public async sign(payload: JWTPayload, options: JWTOptions = {}): Promise<string> {
    if (!payload || typeof payload !== 'object') {
        throw new Error('Payload must be an object');
    }

    // Set default expiration if not provided
    const expiresIn: string | number = options.expiresIn || '1h';

    // Calculate expiration time
    let exp: number | undefined;
    if (typeof expiresIn === 'number') {
        exp = Math.floor(Date.now() / 1000) + expiresIn;
    } else if (typeof expiresIn === 'string') {
        const match: RegExpMatchArray | null = expiresIn.match(/^(\d+)([smhd])$/);
        if (match) {
            const value: number = parseInt(match[1]);
            const unit: string = match[2];
            const seconds: number = {
                's': value,
                'm': value * 60,
                'h': value * 60 * 60,
                'd': value * 60 * 60 * 24
            }[unit]!;
            exp = Math.floor(Date.now() / 1000) + seconds;
        } else {
            throw new Error('Invalid expiresIn format. Use a number (seconds) or a string like "1h", "30m", etc.');
        }
    }

    // Create full payload with claims
    const fullPayload: JWTPayload = {
        ...payload,
        iat: Math.floor(Date.now() / 1000),
        exp
    };

    // Create header
    const header: JWTHeader = {
        alg: 'HS256',
        typ: 'JWT'
    };

    // Encode header and payload
    const encodedHeader: string = this.base64UrlEncode(this.encoder.encode(JSON.stringify(header)));
    const encodedPayload: string = this.base64UrlEncode(this.encoder.encode(JSON.stringify(fullPayload)));

    // Create signature base
    const signatureBase: string = `${encodedHeader}.${encodedPayload}`;

    // Get key and sign
    const key: CryptoKey = await this.getSecretKey();
    const signature: ArrayBuffer = await crypto.subtle.sign(
        { name: 'HMAC' },
        key,
        this.encoder.encode(signatureBase)
    );

    // Encode signature and create token
    const encodedSignature: string = this.base64UrlEncode(signature);
    return `${signatureBase}.${encodedSignature}`;
}
```

## Verifying JWTs (verify method)

The verify method validates a JWT token and returns the decoded payload:

```typescript
public async verify(token: string): Promise<JWTPayload> {
    if (!token || typeof token !== 'string') {
        throw new Error('Token is required and must be a string');
    }

    // Split token into parts
    const parts: string[] = token.split('.');
    if (parts.length !== 3) {
        throw new Error('Invalid token format');
    }

    const [encodedHeader, encodedPayload, encodedSignature] = parts;

    // Decode header and payload
    try {
        const header: JWTHeader = JSON.parse(this.decoder.decode(this.base64UrlDecode(encodedHeader)));
        const payload: JWTPayload = JSON.parse(this.decoder.decode(this.base64UrlDecode(encodedPayload)));

        // Check algorithm
        if (header.alg !== 'HS256') {
            throw new Error(`Unsupported algorithm: ${header.alg}`);
        }

        // Check expiration
        const now: number = Math.floor(Date.now() / 1000);
        if (payload.exp && payload.exp < now) {
            throw new Error('Token has expired');
        }

        // Verify signature
        const key: CryptoKey = await this.getSecretKey();
        const signatureBase: string = `${encodedHeader}.${encodedPayload}`;
        const signature: ArrayBuffer = this.base64UrlDecode(encodedSignature) as ArrayBuffer;

        const isValid: boolean = await crypto.subtle.verify(
            { name: 'HMAC' },
            key,
            signature,
            this.encoder.encode(signatureBase)
        );

        if (!isValid) {
            throw new Error('Invalid signature');
        }

        return payload;
    } catch (error) {
        if (error instanceof Error) {
            throw new Error(`Token verification failed: ${error.message}`);
        }
        throw new Error('Token verification failed: Unknown error');
    }
}
```

## Helper Methods

The class includes several helper methods:

```typescript
private async getSecretKey(): Promise<CryptoKey> {
    const keyData: Uint8Array = this.encoder.encode(this.secret);
    return await crypto.subtle.importKey(
        'raw',
        keyData,
        {
            name: 'HMAC',
            hash: { name: 'SHA-256' },
        },
        false,
        ['sign', 'verify']
    );
}

private base64UrlEncode(buffer: ArrayBuffer): string {
    const base64: string = btoa(String.fromCharCode(...new Uint8Array(buffer)));
    return base64.replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');
}

private base64UrlDecode(base64Url: string) {
    const padding: string = '='.repeat((4 - (base64Url.length % 4)) % 4);
    const base64: string = base64Url.replace(/-/g, '+').replace(/_/g, '/') + padding;
    const rawData: string = atob(base64);
    const buffer: Uint8Array = new Uint8Array(rawData.length);

    for (let i = 0; i < rawData.length; i++) {
        buffer[i] = rawData.charCodeAt(i);
    }

    return buffer.buffer;
}
```

## Implementing in a Cloudflare Worker

Here's a simple example of how to use this class in a Cloudflare Worker:

```typescript
import JWTAuth from './JWTAuth';

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    
    // Initialize the auth with your secret
    const auth = new JWTAuth(env.JWT_SECRET);
    
    // Login route - create a token
    if (url.pathname === '/api/login' && request.method === 'POST') {
      try {
        const { username, password } = await request.json();
        
        // Here you would validate credentials against your database
        // For this example, we're using a simple check
        if (username === 'admin' && password === 'password') {
          const token = await auth.sign({ 
            sub: '123', 
            username,
            role: 'admin'
          }, { expiresIn: '1d' });
          
          return new Response(JSON.stringify({ token }), {
            headers: { 'Content-Type': 'application/json' }
          });
        } else {
          return new Response(JSON.stringify({ error: 'Invalid credentials' }), {
            status: 401,
            headers: { 'Content-Type': 'application/json' }
          });
        }
      } catch (err) {
        return new Response(JSON.stringify({ error: 'Authentication failed' }), {
          status: 500,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    // Protected route - verify token
    if (url.pathname === '/api/protected') {
      try {
        const authHeader = request.headers.get('Authorization');
        if (!authHeader || !authHeader.startsWith('Bearer ')) {
          return new Response(JSON.stringify({ error: 'Missing token' }), {
            status: 401,
            headers: { 'Content-Type': 'application/json' }
          });
        }
        
        const token = authHeader.split(' ')[1];
        const payload = await auth.verify(token);
        
        return new Response(JSON.stringify({ 
          message: 'Protected data',
          user: payload
        }), {
          headers: { 'Content-Type': 'application/json' }
        });
      } catch (err) {
        return new Response(JSON.stringify({ error: 'Authentication failed' }), {
          status: 401,
          headers: { 'Content-Type': 'application/json' }
        });
      }
    }
    
    return new Response('Not Found', { status: 404 });
  }
};
```

## Creating Middleware for Protected Routes

You can easily create a middleware function to protect routes:

```typescript
async function authMiddleware(request: Request, env: Env) {
  const auth = new JWTAuth(env.JWT_SECRET);
  const authHeader = request.headers.get('Authorization');
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' }
    });
  }
  
  try {
    const token = authHeader.split(' ')[1];
    const payload = await auth.verify(token);
    
    // Add user info to request for downstream handlers
    // Note: We're using a custom property here that wouldn't exist on Request
    // In a real app, you'd pass this data to your handlers another way
    (request as any).user = payload;
    
    return null; // No error, continue to next handler
  } catch (err) {
    return new Response(JSON.stringify({ error: 'Invalid token' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' }
    });
  }
}

// Usage in your router
async function handleProtectedRoute(request: Request, env: Env) {
  // Check auth first
  const authError = await authMiddleware(request, env);
  if (authError) return authError;
  
  // Auth passed, handle request
  const user = (request as any).user;
  return new Response(JSON.stringify({ 
    message: 'Protected data accessed',
    user
  }));
}
```

## Integration with Next.js

If you're using Next.js with Cloudflare Workers (like I did), you can create API routes that utilize this JWT authentication:

```typescript
// pages/api/login.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { JWTAuth } from '@/lib/JWTAuth';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { username, password } = req.body;
    
    // Authenticate user (replace with your auth logic)
    // ...
    
    // Create JWT
    const auth = new JWTAuth(process.env.JWT_SECRET!);
    const token = await auth.sign({ 
      sub: userId,
      username
    });
    
    return res.status(200).json({ token });
  } catch (error) {
    console.error(error);
    return res.status(500).json({ error: 'Authentication failed' });
  }
}
```

## Benefits of This Approach

1. **Native to Cloudflare Workers** - Uses Web Crypto API that's available in the Workers runtime
2. **Lightweight** - No dependencies, just pure TypeScript
3. **Type-safe** - Full TypeScript support
4. **Flexible** - Configurable expiration times and custom payloads
5. **Secure** - Uses HMAC SHA-256 for signatures

## Full Source Code

For the complete implementation and a working example, check out the GitHub repository:

[GitHub: cloudflare-jwt-auth](https://github.com/chapimenge3/cloudflare-jwt-auth)

## Conclusion

Authentication doesn't have to be a nightmare in Cloudflare Workers. With this lightweight JWT implementation, you can add secure authentication to your Workers-based applications without relying on external services or complex workarounds.

I've been using this in production with my Next.js application for several months without issues. If you have questions or suggestions for improvements, feel free to open an issue on the GitHub repo.

Happy coding,
Chapi Menge