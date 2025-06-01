## 🧩 Modules Overview

This documentation breaks down the key components of Blink's technical infrastructure. Each module is designed to reflect Blink’s core values: safety, respect, and meaningful connection.

### Covered Modules

- [x] Authentication
- [x] User Profile
- [ ] Matching Engine
- [ ] Conversation Flow
- [ ] Onboarding
- [ ] Moderation & Reporting
- [ ] Notifications
- [ ] Admin & Metrics
- [ ] Infrastructure & Deployment

---

## 🧭 Onboarding Module

The onboarding process is the bridge between account creation and profile activation. It ensures that all Blink users begin their journey with a thoughtful and emotionally safe setup that reflects their identity, preferences, and expectations.

Onboarding is triggered **immediately after first login** and is **mandatory** before access to the rest of the app is granted.

---

### 🧱 Onboarding Structure

The onboarding consists of **two main phases**, designed to progressively enrich the user's profile while preserving flexibility and user agency.

---

### 📋 Phase 1 — Basic Form (Required)

This fixed form gathers essential data for all new users. It ensures every profile contains the minimum viable information for compatibility and safety.

**Fields:**

| Field                | Type          | Notes                                                       |
|---------------------|---------------|-------------------------------------------------------------|
| Name                | Text          | First name only, validated for length and characters        |
| Date of Birth       | Date Picker   | Must be 18 or older, age is calculated and stored           |
| Gender              | Select        | Options: `man`, `woman`, `non-binary/other`                |
| Orientation         | Multi-select  | What the user is looking for: `men`, `women`, `everyone`    |
| Location            | Geolocation   | Retrieved via client-side API or manual selection           |
| Photos              | Uploader      | Minimum 1, up to 6, blurred by default, revealed gradually  |

**Technical Notes:**

- Data is saved via `POST /onboarding/basic-profile`
- Location uses reverse geocoding to store city and coordinates
- Blurring logic is enforced at the media layer, not client-side

---

### 🧠 Phase 2 — Interests & Description (Optional Pathways)

After completing the form, users are invited to **enrich their profile** with personality-revealing content. This step supports **two modes**:

---

#### 📝 Manual Completion

Users fill out:

- **About Me**: Free-text bio  
- **Hobbies & Interests**: Tag-based selector  
- **Looking For**: Values, intentions, priorities  
- **Dealbreakers**: Optional red flags or preferences  

**Endpoint:**  
`POST /onboarding/manual-profile`

---

#### 🤖 Conversational Agent (AI Mode)

Users engage with an **interactive agent** (chatbot-style) that helps them describe:

- Personality traits  
- Life philosophy  
- Relationship expectations  
- Humor, lifestyle, quirks  
- Short profile bio (auto-generated + editable)

**Functionality:**

- Agent uses NLP to infer key traits from conversation  
- Generates initial values for interests and profile description  
- User can approve, modify, or restart this process  

**Endpoint:**  
`POST /onboarding/ai-profile`

**Backend Behavior:**

- Uses OpenAI-like model (e.g. GPT) for intent & tone analysis  
- Suggestions are stored as editable drafts  
- Agent supports fallback to manual at any time

---

### 🧾 Data Structure Example (Merged Profile Payload)
```json
{
  "name": "Alex",
  "birthdate": "1998-04-21",
  "gender": "non-binary",
  "orientation": ["women", "men"],
  "location": {
    "city": "Barcelona",
    "lat": 41.3851,
    "lng": 2.1734
  },
  "photos": ["img1.jpg", "img2.jpg"],
  "bio": "Curious, creative and a firm believer in slow conversations.",
  "interests": ["cinema", "travel", "spirituality", "cooking"],
  "lookingFor": ["authenticity", "long-term", "deep connection"],
  "dealbreakers": ["smoking", "ghosting"]
}
```
### ✅ Validation Rules

- `name`: 2–30 characters, no symbols  
- `birthdate`: must result in age ≥ 18  
- `photos`: at least one required  
- `bio`: optional but encouraged (auto-generated if AI used)  
- All fields are sanitized and validated server-side  

---

### 🧠 Future Enhancements

- [ ] Add support for voice-based conversational onboarding  
- [ ] Emoji & tone indicators for self-expression  
- [ ] AI feedback on clarity and empathy of profile  
- [ ] Onboarding completion score / progress indicator  
- [ ] Localized prompts based on region and culture  

---

### 🛠 Developer Notes

- Onboarding status is stored in the `users` table under `onboarding_complete: boolean`
- Users are locked out of match/discovery flows until onboarding is marked complete
- Conversational agent data is stored in a temporary `draft_profile` table before confirmation
- Photo blurring state is handled via metadata (`is_blurred: true`) per image

---
