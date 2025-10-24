
# Broken Access Control: How I Found (and Exploited) an EdTech Platform's Video Authentication Bypass

So here's the thing. I didn't set out to find a vulnerability. I wasn't following some fancy checklist or reading vulnerability research papers. I was just... curious.

"What if we could do this? Can we access these video URLs directly?" That kind of curious.

And yeah, turns out we could. And we definitely shouldn't have been able to.

This is the story of how a simple curiosity led me to discover a critical Broken Access Control vulnerability that exposed paid course content to anyone willing to look in the right place.

## What Happened

I was just exploring. Testing. Poking around in the browser's Network tab because I had this random thought: *what if I could grab the video URLs that the platform uses internally?*

I wasn't trying to hack anything. I was just asking questions. And the platform answered them in the worst possible way.

What I found: A critical Broken Access Control vulnerability in a popular EdTech platform. Users could extract long-lived, direct URLs to video streams. Those URLs never expire (well, they're set to expire over a year later). Anyone with those URLs could download paid course videos using basic tools like ffmpeg. No authentication. No subscription needed. Just grab the link and download.

The impact? Huge. Intellectual property theft. Revenue loss. Creators losing control of their content. Anyone with a basic account could grab these URLs and share them freely.

## How I Found It

This is where it gets interesting. Most people learn about vulnerabilities first, then go hunting for them. I did it backwards.

I was exploring the platform's network traffic. Just clicking around. Noticed the .m3u8 video playlist files being loaded. Thought: "Can I see these URLs?" 

Opened Developer Tools. Went to the Network tab. Started playing a video.

### Step-by-Step Reproduction

**Step 1:** Log into the platform with any valid account that has access to a paid course

**Step 2:** Navigate to a course with video content

**Step 3:** Open Browser Developer Tools (F12) → Go to the "Network" tab

**Step 4:** Filter the requests by typing `.m3u8` in the filter bar

**Step 5:** Play the video. A new entry appears in the Network tab showing the video playlist request

**Step 6:** Right-click on that .m3u8 request → Copy the full URL

<img width="1919" height="1010" alt="Screenshot 2025-10-22 171552" src="https://github.com/user-attachments/assets/5d35985b-bb3f-4029-a33b-60adf70c1b55" />


I copied that URL. Just to see what it would do.

## Testing It Locally

Here's where it got real.

I went to my Linux machine. Opened a terminal. Had this wild idea: *what if I just tried to download this video using ffmpeg with the URL I copied?*

```bash
ffmpeg -i "PASTE_THE_CAPTURED_URL_HERE" -c copy downloaded_video.mp4
```

<img width="697" height="559" alt="Screenshot 2025-10-22 171820" src="https://github.com/user-attachments/assets/b5242932-39ba-4bfb-a827-4a28893205dc" />


I pressed Enter. Expecting it to fail. Expecting some error message saying "nope, you're not authenticated."

Instead? It started downloading.

<img width="1919" height="1079" alt="Screenshot 2025-10-22 171612" src="https://github.com/user-attachments/assets/8171b51d-f4fc-4da0-9cba-568f3248b7ee" />

The video downloaded. Successfully. Outside the platform. Without being logged in. Without any authentication checks.



That moment. That "wait, this actually worked?" moment hit different.

## Why This Is Critical

**Broken Access Control.** That's the CWE classification. The platform uses signed URLs to grant access, but the expiration timestamp is set so far in the future that it basically doesn't work as a security measure.

### The Technical Details

- **Vulnerability Type:** Insecure Direct Object Reference (IDOR) / Broken Access Control
- **Affected Asset:** All proprietary video content hosted on the platform for paid courses
- **Severity:** High
- **CVSS 3.1 Score:** 8.2 (High)
- **CWE References:**
  - CWE-639: Authorization Bypass Through User-Controlled Key
  - CWE-284: Improper Access Control
  - CWE-200: Exposure of Sensitive Information to an Unauthorised Actor

The signed URL acts like a permanent key. Once you have it, you own it. You can share it. Use it anywhere. And there's nothing stopping you.

### The Real Impact

- **Intellectual Property Theft:** Content creators lose control of their intellectual property
- **Revenue Loss:** The platform loses revenue (why pay for a course when you can get the free link?)
- **Subscription Evasion:** Users can share these links, allowing non-paying individuals to access premium content for free
- **Brand Damage:** The inability to protect creators' content damages the platform's reputation

## What Should Be Done

I reported this to the platform. They've acknowledged it and are working on fixes. Here's what they should do:

### 1. Implement Short-Lived Signed URLs

The expiration time for signed URLs must be drastically reduced. A URL should only be valid for 5-10 minutes. Just long enough for the video player to start playing. New, short-lived URLs should be generated for subsequent chunks or session renewals.

### 2. Enforce Server-Side Session Validation

Don't rely solely on the signed URL. For every request for video content, the server should also validate the user's current session cookie and authorisation token. The server needs to check: "Is this person still logged in? Do they still have permission?"

### 3. Implement Digital Rights Management (DRM)

For maximum protection, use solutions like Google Widevine, Apple FairPlay, or Microsoft PlayReady. These encrypt the content and the player can only decrypt it after obtaining a license from the server. This makes it nearly impossible to simply download videos.

### 4. Monitor for Abnormal Activity

Implement logging and monitoring to detect suspicious behaviour. Unusual spike in requests from one IP? One user account downloading tons of videos? That's a red flag.

## What I Learned

This whole thing taught me something fundamental about security.

You don't need to be an expert to find vulnerabilities. You don't need to read textbooks on attack vectors. Sometimes you just need curiosity. And a willingness to ask dumb questions.

Most people are told: "Learn security theory → Then go find bugs."

I skipped the theory part. I found the bug first. Then realized what it meant.

That's actually more valuable in some ways. Because I wasn't looking for a specific type of vulnerability. I was just exploring. And when I found something weird, I investigated it.

If you're reading this thinking "I'm not a security expert, so I can't do this," that's the wrong mindset. The best discoveries often come from people who don't know what they're supposed to be looking for.

Just be curious. Test things. Ask questions. And be responsible when you find something.

---

## Connect With Me

If this interested you, check out more of my journey:

- **YouTube:** [https://youtube.com/@akashmadanu3994](https://youtube.com/@akashmadanu3994)
  Learn more about cybersecurity discoveries and ethical hacking.

- **LinkedIn:** [https://www.linkedin.com/in/akash-madanu/](https://www.linkedin.com/in/akash-madanu/)
  Follow my professional journey and insights.

- **Instagram:** [https://www.instagram.com/cyb3rwithakash/](https://www.instagram.com/cyb3rwithakash/)
  Daily cybersecurity content and updates.

---

*This vulnerability was reported through responsible disclosure. The platform has been notified and is actively working on fixes. This writeup is shared to help other researchers and security teams understand the importance of proper access control mechanisms.*
