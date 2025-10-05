# How to create a react project?
- create a CRA project: 

    Create React App is a tool developed by Meta (formerly Facebook) that provides a simple setup for building single-page React applications (SPAs) with minimal configuration.
    + Client-side rendering (CSR) by default
    + Great for small to medium-sized apps or prototypes
    + No built-in routing (uses react-router)
    + No built-in server-side rendering or API routes
    + Simple and lightweight, but less flexible

```shell
npx create-react-app my-app
cd my-app
npm start
```

- create a [NEXTJS project](https://nextjs.org/docs/app/getting-started/installation): 

    Next.js is a React framework developed by Vercel that offers a complete solution for building modern web applications, including SSR (Server-Side Rendering), SSG (Static Site Generation), and more.

    + Built-in routing (file-based)
    + SSR, SSG, ISR, and CSR all supported
    + API routes (serverless functions)
    + Better SEO support via SSR/SSG
    + Middleware, image optimization, and more
    + Highly scalable for enterprise use

```shell
npx create-next-app@latest
```