### **What is this project?**

It’s a web application called **Pulse 360°** built with modern web technologies (Next.js, React, TypeScript) to help manage Microsoft cloud services like Azure, Microsoft 365, Power Platform, and Intune. Think of it as a dashboard that talks to Microsoft APIs securely.

***

### **How does it work?**


*   **Framework & Language:** Uses Next.js (a popular web framework) and React for the user interface. Code is mostly JavaScript with some TypeScript for type safety.
*   **Authentication:** It connects to Microsoft Entra ID (formerly Azure AD) for sign-in and permissions. It uses NextAuth and MSAL libraries to handle secure token exchange.
*   **APIs:** Pulls data from Microsoft Graph (for M365), Azure Resource Graph (for Azure), and Power Platform APIs.
*   **Environment:** Runs on Node.js (version 18 or higher).

***

### **What’s inside the project?**

*   **Pages & Routes:** The app’s screens live under `src/app/...`. API endpoints (server-side logic) are also defined there.
*   **Shared Libraries:** Helpers for authentication and API calls are in `src/lib/`.
*   **Tests:** Automated browser tests using Playwright are in `tests/`.
*   **Config Files:** Settings for Next.js, TypeScript, linting, and Playwright are in the root.

***

### **What do you need to run it?**

*   Install Node.js 18+.
*   Create a `.env.local` file with:
    *   App URL and secret for NextAuth.
    *   Azure credentials (Tenant ID, Client ID, Client Secret).
*   Optional: Azure Subscription ID for extra features.

***

### **How do you start it?**

1.  **Install dependencies:** `npm install`
2.  **Run in development:** `npm run dev` (starts a local server on port 3000)
3.  **Build for production:** `npm run build`
4.  **Start production server:** `npm start`
5.  **Lint and test:** `npm run lint` and `npx playwright test`

***

### **Important rules for editing code**

*   **Server vs Client Components:** Next.js splits code into server-rendered and client-rendered parts. Follow the guide’s rules to avoid bugs like infinite loops.
*   **Auth tokens:** Don’t mix login scopes incorrectly; resource tokens are fetched server-side.
*   **Retry logic:** When calling Microsoft APIs, handle errors gracefully (e.g., retry on temporary failures).
*   **Structural edits:** If you change code, keep brackets and tags balanced to avoid breaking the app.

***

### **Why does this matter?**

These practices ensure:

*   The app runs securely and efficiently.
*   You avoid common pitfalls like broken layouts or authentication errors.
*   Tests and linting help catch mistakes before deploying.

***
