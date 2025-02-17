# ChefGPT: Your AI-Powered Culinary Assistant

> AI-powered culinary assistant. Chat with a virtual Michelin-star chef, get personalized recipes, and elevate your cooking skills.

**Live Demo:** [https://deep-chef.netlify.app/](https://deep-chef.netlify.app/) 

## Overview

ChefGPT brings the expertise of a Michelin-starred chef, Auguste, directly to your kitchen. This web application leverages the power of AI to provide a personalized and interactive culinary experience. Create custom profiles, chat with Auguste, and receive recipe suggestions tailored to your specific needs and preferences, even using image recognition of your ingredients!

## Features & How It Works

ChefGPT is designed to be your all-in-one culinary companion. Here's how it works:

1.  **Create Personalized Profiles:**

    *   Define your cooking skill level (from Beginner to Professional).
    *   Specify dietary restrictions (Vegetarian, Vegan, Gluten-Free, Keto, and more).
    *   List your available kitchen appliances (Oven, Stovetop, Microwave, Air Fryer, etc.).
    *   Add personal details about your cooking journey, favorite cuisines, or dietary goals.

<img width="1007" alt="Screenshot 2025-02-17 at 14 21 06" src="https://github.com/user-attachments/assets/a60384fe-b52c-44df-852b-94871e3bf870" />
<img width="972" alt="Screenshot 2025-02-17 at 14 22 34" src="https://github.com/user-attachments/assets/026902c3-6752-4b62-a338-9671970287dd" />


  *Example of creating a new profile, highlighting dietary restrictions and kitchen appliances.*

<img width="970" alt="Screenshot 2025-02-17 at 14 23 55" src="https://github.com/user-attachments/assets/a4e16a25-dbe2-4571-b0b8-110109f4f039" />

  *Managing multiple profiles, each with unique settings.*

  You can create multiple profiles, each with unique settings. This is perfect for managing different dietary needs or experimenting with various cooking styles.

2.  **Chat with Auguste:**

    *   Engage in natural conversations with Auguste, your AI chef.
    *   Ask for cooking tips, recipe ideas, or general culinary advice.
    *   Auguste remembers your profile details and tailors responses accordingly.
    *   Conversation history is saved per profile, so you can easily refer back to previous interactions.

    <img width="1151" alt="Screenshot 2025-02-17 at 14 26 56" src="https://github.com/user-attachments/assets/7a0e6afd-ba13-4751-8484-6cebb534c249" />
    <img width="1152" alt="Screenshot 2025-02-17 at 14 27 15" src="https://github.com/user-attachments/assets/91c54843-3610-459b-9612-b2c38959efe1" />


    *Example conversation with Auguste, demonstrating personalized responses.*

3.  **Get Personalized Recipes:**

    *   Tell Auguste what ingredients you have on hand, or **upload an image of your food items!**
    *   Auguste will analyze the image to identify the ingredients and then craft a detailed recipe, including:
        *   A clear title.
        *   A complete ingredient list with quantities.
        *   Step-by-step instructions, explaining *why* each step is important.
        *   Precise measurements and cooking times.
        *   Helpful tips and variations.
    *   Recipes are generated with your profile's cooking skill, dietary restrictions, and available appliances in mind.

<img width="1153" alt="Screenshot 2025-02-17 at 14 51 35" src="https://github.com/user-attachments/assets/d842fa80-d030-4852-95c6-f31dc6b0e123" />
    *Uploading an image of ingredients for analysis.*

  <img width="1154" alt="Screenshot 2025-02-17 at 14 51 47" src="https://github.com/user-attachments/assets/b2e9ea67-e1fd-4f48-b6df-e1c68500d701" />

  *Auguste providing a detailed and custom recipe, taking the user's profile into account.*

## Technologies Used

ChefGPT is built using a modern web development stack:

*   **Frontend:** React, React Router, Framer Motion, Lucide React, Tailwind CSS, Vite
*   **Backend:** Netlify Functions, Google Gemini API, Moondream API
*   **Deployment:** Netlify

## Development (For Portfolio Purposes)

This section provides a high-level overview of the project's structure and includes illustrative code snippets. *(Note: The full codebase is not publicly available.)*

*   **Frontend:** The user interface is built with React, using a component-based architecture. State management is handled with React's built-in hooks and local storage for persisting user profiles and conversation history.  Key components include `AIChatAssistant`, `ProfileCreation`, and `Homepage`.

*   **Backend:** Netlify Functions handle API requests securely.  These functions interact with the Google Gemini API for AI chat and recipe generation, and the Moondream API for image analysis.

    **Example: Image Analysis with Moondream (Netlify Function):**

    ```javascript
    // netlify/functions/moondream-analysis.js
    exports.handler = async function (event) {
        try {
          const { imageBase64 } = JSON.parse(event.body);
          if (!imageBase64) {
            throw new Error("Image data is required");
          }
          const moondreamKey = process.env.MOONDREAM_API_KEY;

          const response = await fetch("https://api.moondream.ai/v1/caption", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              "X-Moondream-Auth": moondreamKey,
            },
            body: JSON.stringify({
              image_url: `data:image/jpeg;base64,${imageBase64}`,
              stream: false,
            }),
          });

          if (!response.ok) {
            throw new Error(`Moondream API failed with status ${response.status}`);
          }

          const moondreamData = await response.json();
          return {
            statusCode: 200,
            body: JSON.stringify({
              caption: moondreamData.caption || "No caption available",
            }),
          };
        } catch (error) {
          // ... error handling ...
        }
    };
    ```

    This snippet shows how the `moondream-analysis` Netlify Function receives a base64-encoded image, sends it to the Moondream API for captioning, and returns the result.

    **Example: AI Chat with Gemini (Netlify Function):**

    ```javascript
    // netlify/functions/ai-chat.js (partial)
    exports.handler = async function (event) {
        try {
            const { messages, imageAnalysis, userProfile } = JSON.parse(event.body);
            const geminiKey = process.env.GEMINI_API_KEY;
            let profileContext = '';
            if (userProfile) {
                // ... build profileContext string ...
            }
          const aiMessages = [
            {
              role: "system",
              content: `You are Auguste, a Michelin-star chef ...
    ${profileContext}
              // ... rest of the system prompt ...`,
            },
          ];
          // ... add user messages and image analysis to aiMessages ...

          const geminiResponse = await retryRequest(async () => {  // Using retry logic
              const response = await fetch(
                `https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${geminiKey}`,
                {
                  method: "POST",
                  // ... headers and body ...
                }
              );
              if (!response.ok) {
                throw new Error(`Gemini API failed with status ${response.status}`);
              }
              return response;
          });

          const geminiData = await geminiResponse.json();
          const responseText = geminiData.candidates?.[0]?.content?.parts?.[0]?.text || "No response content found";

          return {
            statusCode: 200,
            body: JSON.stringify({
              content: responseText,
            }),
          };
        } catch (error) {
          // ... error handling ...
        }
    };
    ```

This snippet demonstrates how the `ai-chat` function integrates user profiles, previous messages, and image analysis (from Moondream) into the prompt for the Gemini API.  It also shows the use of a `retryRequest` function (not shown here, but present in your code) to handle potential API rate limits.
 **Example: Profile Data Structure (React Component):**

    ```javascript
     //Simplified example from ProfileCreation.jsx or similar

        const [formData, setFormData] = useState({
            id: null,
            name: "",
            age: "",
            dietaryRestrictions: [],
            otherRestrictions: "",
            cookingLevel: "",
            appliances: [],
            description: ""
        });
    ```

    This snippet illustrates the structure used to manage user profile data within the React application.
*   **User Profiles:** User profile and conversation are stored using browser's local storage.


## Created By

ChefGPT is a project by Jordan Montée (AlikelDev) and Kamil Serrai (@[Klima42](https://github.com/Klima42)) for Alikearn Studio.

---
