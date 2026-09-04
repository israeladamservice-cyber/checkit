<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Live Store Outreach</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    * { box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: #f4f6f9; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 480px; margin: 0 auto; background: white; padding: 24px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.06); }
    h2 { font-size: 1.25rem; margin-top: 0; color: #111; }
    label { font-size: 0.85rem; font-weight: 600; color: #555; display: block; margin-top: 14px; }
    input, select { width: 100%; padding: 12px; margin-top: 6px; border: 1px solid #d1d5db; border-radius: 8px; font-size: 0.95rem; }
    button { width: 100%; background: #0066ff; color: white; border: none; padding: 12px; border-radius: 8px; font-weight: bold; margin-top: 18px; cursor: pointer; font-size: 0.95rem; }
    button:disabled { background: #99c2ff; cursor: not-allowed; }
    .card { margin-top: 20px; padding: 14px; border-radius: 8px; background: #f8fafc; border: 1px solid #e2e8f0; display: none; }
    .card h3 { margin: 0 0 8px 0; font-size: 0.85rem; color: #475569; text-transform: uppercase; letter-spacing: 0.05em; }
    .pitch-box { font-size: 0.92rem; line-height: 1.5; color: #1e293b; white-space: pre-line; background: #fff; padding: 10px; border-radius: 6px; border: 1px solid #cbd5e1; }
    .status { font-size: 0.85rem; color: #0066ff; margin-top: 10px; text-align: center; font-weight: 600; }
    .error-box { color: #dc2626; background: #fef2f2; padding: 10px; border-radius: 6px; font-size: 0.85rem; margin-top: 10px; display: none; }
  </style>
</head>
<body>

<div class="container">
  <h2>Live Store Outreach</h2>
  
  <label for="platform">Platform</label>
  <select id="platform">
    <option value="Instagram">Instagram</option>
  </select>

  <label for="username">Instagram Username</label>
  <input type="text" id="username" placeholder="e.g. nike">

  <button id="analyzeBtn" onclick="analyzeAndPitch()">Analyze Live Profile</button>
  <div id="statusText" class="status"></div>
  <div id="errorText" class="error-box"></div>

  <!-- Real Bio Card -->
  <div id="bioCard" class="card">
    <h3>Profile Bio Detected</h3>
    <div id="bioText" style="font-size:0.88rem; color:#334155; font-weight: 500;"></div>
  </div>

  <!-- Generated Pitch Card -->
  <div id="pitchCard" class="card">
    <h3>Generated Pitch</h3>
    <div id="pitchText" class="pitch-box"></div>
    <button id="saveBtn" style="background: #10b981; margin-top:12px;" onclick="saveToSupabase()">Save Lead to Supabase</button>
  </div>
</div>

<script>
  const SUPABASE_URL = "YOUR_SUPABASE_URL"; 
  const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";

  let supabase = null;
  if (SUPABASE_URL !== "YOUR_SUPABASE_URL" && SUPABASE_ANON_KEY !== "YOUR_SUPABASE_ANON_KEY") {
    supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
  }

  let currentProfileData = {};

  async function analyzeAndPitch() {
    const username = document.getElementById('username').value.trim().replace('@', '');
    const analyzeBtn = document.getElementById('analyzeBtn');
    const statusText = document.getElementById('statusText');
    const errorText = document.getElementById('errorText');

    if (!username) {
      alert("Please enter a username.");
      return;
    }

    analyzeBtn.disabled = true;
    errorText.style.display = 'none';
    statusText.innerText = `Analyzing live page for @${username}...`;

    try {
      // Scrape open web metadata directly without third-party API keys
      const targetUrl = encodeURIComponent(`https://www.instagram.com/${username}/`);
      const res = await fetch(`https://api.allorigins.win/get?url=${targetUrl}`);

      if (!res.ok) throw new Error("Could not reach Instagram profile page.");

      const data = await res.json();
      const htmlText = data.contents;

      // Extract description tag
      let extractedBio = "Active Instagram brand page.";
      const metaMatches = htmlText.match(/<meta property="og:description" content="([^"]*)"/i) || 
                          htmlText.match(/<meta name="description" content="([^"]*)"/i);

      if (metaMatches && metaMatches[1]) {
        extractedBio = metaMatches[1];
      }

      // Display scraped profile summary
      document.getElementById('bioText').innerText = extractedBio;
      document.getElementById('bioCard').style.display = 'block';

      // Build outreach pitch
      const pitch = buildPitchFromBio(username, extractedBio);
      document.getElementById('pitchText').innerText = pitch;
      document.getElementById('pitchCard').style.display = 'block';

      currentProfileData = {
        platform: "Instagram",
        handle: `@${username}`,
        bio_notes: extractedBio,
        suggested_message: pitch
      };

      statusText.innerText = "Fetch Successful!";
    } catch (err) {
      statusText.innerText = "";
      errorText.innerText = "Error: " + err.message;
      errorText.style.display = 'block';
    } finally {
      analyzeBtn.disabled = false;
    }
  }

  function buildPitchFromBio(handle, bio) {
    const bioLower = bio.toLowerCase();
    let niche = "e-commerce brand";

    if (bioLower.includes("apparel") || bioLower.includes("clothing") || bioLower.includes("wear") || bioLower.includes("fashion")) {
      niche = "apparel brand";
    } else if (bioLower.includes("skin") || bioLower.includes("beauty") || bioLower.includes("cosmetics")) {
      niche = "beauty store";
    }

    return `Hey @${handle}! 👋\n\n` +
      `Came across your Instagram profile while researching active ${niche}s.\n\n` +
      `Are you currently looking to upgrade your web store design to increase conversion rates for profile visitors this month?\n\n` +
      `Would love to send over a quick 1-minute visual design concept if you're open to checking it out!`;
  }

  async function saveToSupabase() {
    if (!supabase) {
      alert("Please enter your Supabase URL and Key inside the code to save data.");
      return;
    }

    const { error } = await supabase
      .from('store_leads')
      .insert([currentProfileData]);

    if (error) {
      alert('Error saving to Supabase: ' + error.message);
    } else {
      alert('Saved directly to Supabase!');
    }
  }
</script>

</body>
</html>
