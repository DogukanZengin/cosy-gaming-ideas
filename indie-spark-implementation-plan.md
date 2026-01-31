# 🔥 Indie Spark - Daily Email Workflow Implementation Plan

A comprehensive n8n workflow that delivers daily game development inspiration combining cozy aesthetics, unique mechanics, and scientific hooks.

---

## 📋 Overview

**Goal:** Wake up to a single email containing 3-5 curated game ideas that spark your creativity.

**Stack:**
- **n8n** (self-hosted) - Workflow automation
- **Gemini API** - Idea generation and filtering
- **SMTP** - Email delivery

**Schedule:** Daily at 7:00 AM (configurable)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INDIE SPARK WORKFLOW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────┐                                                               │
│  │ Schedule │ (7:00 AM Daily)                                               │
│  └────┬─────┘                                                               │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    PARALLEL DATA FETCHING                            │   │
│  ├─────────────────┬─────────────────┬─────────────────────────────────┤   │
│  │                 │                 │                                  │   │
│  │  ┌───────────┐  │  ┌───────────┐  │  ┌───────────┐                  │   │
│  │  │ Cozy      │  │  │ Game Jam  │  │  │ Nature    │                  │   │
│  │  │ Sources   │  │  │ + Patents │  │  │ + Books   │                  │   │
│  │  └───────────┘  │  └───────────┘  │  └───────────┘                  │   │
│  │                 │                 │                                  │   │
│  │  ┌───────────┐  │  ┌───────────┐  │  ┌───────────┐                  │   │
│  │  │ Mechanics │  │  │ Atlas     │  │  │ NASA APOD │                  │   │
│  │  │ (BGG)     │  │  │ Obscura   │  │  │           │                  │   │
│  │  └───────────┘  │  └───────────┘  │  └───────────┘                  │   │
│  │                 │                 │                                  │   │
│  └────────┬────────┴────────┬────────┴────────┬────────────────────────┘   │
│           │                 │                 │                             │
│           ▼                 ▼                 ▼                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         MERGE ALL DATA                               │   │
│  └────────────────────────────────┬────────────────────────────────────┘   │
│                                   │                                         │
│                                   ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    GEMINI AI AGENT NODE                              │   │
│  │  "Generate 3-5 experimental cozy game ideas from these ingredients" │   │
│  └────────────────────────────────┬────────────────────────────────────┘   │
│                                   │                                         │
│                                   ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      HTML EMAIL FORMATTER                            │   │
│  └────────────────────────────────┬────────────────────────────────────┘   │
│                                   │                                         │
│                                   ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         SEND EMAIL (SMTP)                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📦 Phase 1: Setup & Credentials (30 min)

### 1.1 Create Credentials in n8n

| Credential Type | Name | Required Fields |
|-----------------|------|-----------------|
| **HTTP Header Auth** | `Gemini API` | Header: `x-goog-api-key`, Value: Your API key |
| **SMTP** | `Email SMTP` | Host, Port, User, Password (Gmail/SendGrid/etc.) |
| **HTTP Header Auth** | `Reddit` | Optional - for higher rate limits |

### 1.2 Environment Variables

Add to your n8n docker-compose or .env:

```bash
# Gemini API
GEMINI_API_KEY=your_gemini_api_key_here

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password

# Timezone
GENERIC_TIMEZONE=Europe/Istanbul
```

---

## 📡 Phase 2: Data Source Nodes (1-2 hours)

### 2.1 Cozy Aesthetic Sources (High Weight: 50%)

#### Node: `Fetch r/CozyPlaces`
```
Type: HTTP Request
Method: GET
URL: https://www.reddit.com/r/CozyPlaces/hot.json?limit=10
Headers: User-Agent: "IndieSpark/1.0"
```

**Code Node to Process:**
```javascript
const posts = $input.first().json.data.children;
return posts.slice(0, 5).map(post => ({
  json: {
    type: 'cozy_aesthetic',
    title: post.data.title,
    image_url: post.data.url,
    source_url: `https://reddit.com${post.data.permalink}`,
    source: 'r/CozyPlaces'
  }
}));
```

#### Node: `Fetch r/TinyHouses` (backup cozy source)
```
Type: HTTP Request
Method: GET
URL: https://www.reddit.com/r/TinyHouses/hot.json?limit=5
```

### 2.2 Mechanics Sources (Medium Weight: 30%)

#### Node: `Fetch BGG Hot Games`
```
Type: HTTP Request
Method: GET
URL: https://boardgamegeek.com/xmlapi2/hot?type=boardgame
Response Format: XML
```

**Code Node to Extract Mechanics:**
```javascript
// Parse XML response and extract interesting mechanics
const games = $input.first().json;
const mechanics = [
  "Area Control", "Deck Building", "Worker Placement",
  "Push Your Luck", "Tile Laying", "Resource Management",
  "Hand Management", "Pattern Building", "Route Building"
];

// Pick 2-3 random mechanics for variety
const shuffled = mechanics.sort(() => Math.random() - 0.5);
return shuffled.slice(0, 3).map(mech => ({
  json: {
    type: 'mechanic',
    mechanic: mech,
    source: 'BoardGameGeek',
    source_url: 'https://boardgamegeek.com/'
  }
}));
```

### 2.3 Wildcard Sources (Low Weight: 20%)

#### Node: `Fetch Atlas Obscura`
```
Type: HTTP Request
Method: GET
URL: https://www.atlasobscura.com/random
(Or use their RSS: https://www.atlasobscura.com/feeds/latest)
```

#### Node: `Fetch NASA APOD`
```
Type: HTTP Request
Method: GET
URL: https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY
```

**Process NASA Response:**
```javascript
const apod = $input.first().json;
return [{
  json: {
    type: 'nature_science',
    title: apod.title,
    description: apod.explanation.substring(0, 200) + '...',
    image_url: apod.url,
    source_url: 'https://apod.nasa.gov/',
    source: 'NASA APOD'
  }
}];
```

#### Node: `Fetch Random Gutenberg Title`
```
Type: HTTP Request
Method: GET
URL: https://gutendex.com/books/?languages=en&sort=random
```

**Process Gutenberg:**
```javascript
const books = $input.first().json.results;
const randomBook = books[Math.floor(Math.random() * books.length)];
return [{
  json: {
    type: 'narrative_hook',
    title: randomBook.title,
    author: randomBook.authors[0]?.name || 'Unknown',
    source_url: `https://www.gutenberg.org/ebooks/${randomBook.id}`,
    source: 'Project Gutenberg'
  }
}];
```

### 2.4 Game Jam Sources

#### Node: `Fetch Itch.io Jams`
```
Type: HTTP Request
Method: GET
URL: https://itch.io/jams/upcoming.json
```

**Process Game Jams:**
```javascript
const jams = $input.first().json.jams || [];
const activeJams = jams.slice(0, 3);
return activeJams.map(jam => ({
  json: {
    type: 'game_jam',
    title: jam.title,
    theme: jam.theme || 'Open theme',
    url: jam.url,
    source: 'itch.io'
  }
}));
```

---

## 🤖 Phase 3: Gemini AI Agent Node (1 hour)

### 3.1 Create AI Agent Node

```
Node Type: HTTP Request
Method: POST
URL: https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent
Authentication: HTTP Header Auth (Gemini API)
```

### 3.2 System Prompt (paste into request body)

```json
{
  "contents": [{
    "parts": [{
      "text": "{{ $json.prompt }}"
    }]
  }],
  "generationConfig": {
    "temperature": 0.9,
    "maxOutputTokens": 2048
  }
}
```

### 3.3 Build the Prompt (Code Node before Gemini)

```javascript
const ingredients = $input.all().map(item => item.json);

const cozyItems = ingredients.filter(i => i.type === 'cozy_aesthetic');
const mechanics = ingredients.filter(i => i.type === 'mechanic');
const wildcards = ingredients.filter(i => i.type === 'nature_science' || i.type === 'narrative_hook');
const jams = ingredients.filter(i => i.type === 'game_jam');

const prompt = `You are a creative game designer assistant for a solo indie developer who makes cozy, experimental games.

TODAY'S INGREDIENTS:

🏡 COZY SETTINGS (pick 1-2):
${cozyItems.map(c => `- "${c.title}" [${c.source}]`).join('\n')}

🎲 MECHANICS TO CONSIDER:
${mechanics.map(m => `- ${m.mechanic}`).join('\n')}

🔬 WILDCARD INSPIRATION:
${wildcards.map(w => `- ${w.title}: ${w.description || ''}`).join('\n')}

🎮 ACTIVE GAME JAMS:
${jams.map(j => `- "${j.title}" - Theme: ${j.theme}`).join('\n')}

---

Generate EXACTLY 3 game ideas in this format:

## 💡 Idea 1: [Catchy Name]
**The Pitch:** [One sentence hook]
**Setting:** [Cozy location inspired by ingredients]
**Core Mechanic:** [What the player actually DOES]
**The Twist:** [What makes this experimental/unique]
**Solo Dev Scope:** [Why this is achievable for one person]

[Repeat for Ideas 2 and 3]

RULES:
- Keep it COZY - no combat, no stress, no time pressure
- Keep it EXPERIMENTAL - unusual mechanics or perspectives
- Keep it SMALL - scope for a solo dev in 1-3 months
- Make each idea DIFFERENT from the others
- Be specific, not generic`;

return [{
  json: {
    prompt: prompt,
    raw_ingredients: ingredients
  }
}];
```

### 3.4 Parse Gemini Response

```javascript
const response = $input.first().json;
const text = response.candidates[0].content.parts[0].text;
const ingredients = $('Build Prompt').first().json.raw_ingredients;

return [{
  json: {
    ideas: text,
    sources: ingredients,
    generated_at: new Date().toISOString()
  }
}];
```

---

## 📧 Phase 4: HTML Email Template (30 min)

### 4.1 HTML Template Node (Code Node)

```javascript
const data = $input.first().json;
const ideas = data.ideas;
const sources = data.sources;
const date = new Date().toLocaleDateString('en-US', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric'
});

// Build source links
const sourceLinks = sources
  .filter(s => s.source_url)
  .map(s => `<li><a href="${s.source_url}">${s.title || s.mechanic || s.source}</a> (${s.source})</li>`)
  .join('\n');

const html = `
<!DOCTYPE html>
<html>
<head>
  <style>
    body {
      font-family: 'Georgia', serif;
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
      background: #faf8f5;
      color: #2d2d2d;
    }
    .header {
      text-align: center;
      border-bottom: 2px solid #e8dfd5;
      padding-bottom: 20px;
      margin-bottom: 30px;
    }
    .header h1 {
      color: #5d4e37;
      margin: 0;
      font-size: 28px;
    }
    .header .date {
      color: #8b7355;
      font-size: 14px;
      margin-top: 5px;
    }
    .ideas {
      background: white;
      border-radius: 12px;
      padding: 25px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.05);
    }
    .ideas h2 {
      color: #6b8e6b;
      border-bottom: 1px solid #e8dfd5;
      padding-bottom: 10px;
    }
    .ideas h3 {
      color: #5d4e37;
      margin-top: 25px;
    }
    .sources {
      margin-top: 40px;
      padding: 20px;
      background: #f5f0e8;
      border-radius: 8px;
    }
    .sources h3 {
      margin-top: 0;
      color: #8b7355;
      font-size: 16px;
    }
    .sources ul {
      padding-left: 20px;
    }
    .sources a {
      color: #6b8e6b;
    }
    .footer {
      text-align: center;
      margin-top: 30px;
      color: #a99b8a;
      font-size: 12px;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>🌿 Indie Spark</h1>
    <div class="date">${date}</div>
  </div>

  <div class="ideas">
    ${ideas.replace(/##/g, '<h2>').replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')}
  </div>

  <div class="sources">
    <h3>📚 Today's Inspiration Sources</h3>
    <ul>
      ${sourceLinks}
    </ul>
  </div>

  <div class="footer">
    <p>Made with ☕ by your n8n workflow</p>
    <p>Reply to this email to save ideas you like!</p>
  </div>
</body>
</html>
`;

return [{
  json: {
    html: html,
    subject: `🌿 Indie Spark - ${date}`,
    plain_text: ideas
  }
}];
```

### 4.2 Send Email Node

```
Node Type: Send Email (SMTP)
To: your_email@gmail.com
Subject: {{ $json.subject }}
HTML: {{ $json.html }}
Text: {{ $json.plain_text }}
```

---

## 🔧 Phase 5: Error Handling & Reliability (30 min)

### 5.1 Add Error Workflow

Create a separate "Error Handler" workflow:

```
Error Trigger → Slack/Discord Notification
             → Log to file/database
```

### 5.2 Add Fallbacks for Each Data Source

```javascript
// In each HTTP Request node, enable "Continue On Fail"
// Then add IF node to check for errors

// Example fallback for Reddit:
if ($json.error || !$json.data) {
  return [{
    json: {
      type: 'cozy_aesthetic',
      title: 'A quiet cabin in the woods',
      source: 'Fallback',
      source_url: ''
    }
  }];
}
```

### 5.3 Rate Limiting

Add `Wait` nodes between API calls if needed:
```
HTTP Request → Wait (1 second) → Next HTTP Request
```

---

## ✅ Implementation Checklist

### Week 1: Foundation
- [ ] Set up Gemini API credentials
- [ ] Configure SMTP for email
- [ ] Create basic workflow skeleton
- [ ] Test Schedule Trigger

### Week 2: Data Sources
- [ ] Implement Reddit fetching (r/CozyPlaces, r/TinyHouses)
- [ ] Implement NASA APOD
- [ ] Implement Gutenberg random book
- [ ] Implement Itch.io game jams
- [ ] Add BGG mechanics extraction
- [ ] Test each source individually

### Week 3: AI & Email
- [ ] Build prompt construction node
- [ ] Configure Gemini API call
- [ ] Parse Gemini response
- [ ] Create HTML email template
- [ ] Test full workflow end-to-end

### Week 4: Polish & Reliability
- [ ] Add error handling
- [ ] Add fallback data sources
- [ ] Create error notification workflow
- [ ] Test failure scenarios
- [ ] Enable production schedule

---

## 🎛️ Configuration Options

### Adjust Idea Count
In the Gemini prompt, change "Generate EXACTLY 3 game ideas" to your preferred number.

### Adjust Weirdness Level
Modify the temperature in Gemini config:
- `0.7` = More focused, predictable ideas
- `0.9` = Balanced (recommended)
- `1.2` = Maximum chaos/creativity

### Change Schedule
Edit the Schedule Trigger node:
- Daily: `0 7 * * *` (7 AM every day)
- Weekdays only: `0 7 * * 1-5`
- Twice daily: `0 7,19 * * *` (7 AM and 7 PM)

---

## 🚀 Quick Start

1. **Import the workflow JSON** (I can generate this for you)
2. **Add your credentials** (Gemini API, SMTP)
3. **Test with Manual Trigger** first
4. **Enable the Schedule** once everything works
5. **Wake up to inspiration!** ☕🎮

---

## 💡 Future Enhancements

- **Feedback Loop:** Reply "⭐" to save ideas to a database
- **Weekly Digest:** Compile best ideas from the week
- **Mood Selector:** Choose between "cozy", "weird", "narrative" focus
- **Image Generation:** Use DALL-E/Midjourney for concept art
- **Notion Integration:** Auto-save ideas to a Notion database

---

*Ready to build? Let me know if you want me to generate the complete n8n workflow JSON file!*
