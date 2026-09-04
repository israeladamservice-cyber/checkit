<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Lead Scout & Outreach AI</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    * { box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: #f4f6f9; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 500px; margin: 0 auto; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
    h2 { font-size: 1.2rem; margin-top: 0; }
    label { font-size: 0.85rem; font-weight: 600; color: #555; display: block; margin-top: 12px; }
    input, select, textarea { width: 100%; padding: 10px; margin-top: 4px; border: 1px solid #ccc; border-radius: 6px; font-size: 0.95rem; }
    button { width: 100%; background: #0066ff; color: white; border: none; padding: 12px; border-radius: 6px; font-weight: bold; margin-top: 15px; cursor: pointer; }
    
    /* Floating Action Button */
    .fab {
      position: fixed;
      bottom: 25px;
      right: 25px;
      width: 56px;
      height: 56px;
      background-color: #0066ff;
      color: white;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 24px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.25);
      cursor: pointer;
      z-index: 1000;
    }

    .output-box { margin-top: 15px; background: #f0f4f8; padding: 12px; border-radius: 6px; border-left: 4px solid #0066ff; font-size: 0.9rem; white-space: pre-line; }
  </style>
</head>
<body>

<div class="container">
  <h2>Store Outreach Assistant</h2>
  
  <label for="platform">Platform</label>
  <select id="platform">
    <option value="Instagram">Instagram</option>
    <option value="TikTok">TikTok</option>
  </select>

  <label for="handle">Store Handle / Name</label>
  <input type="text" id="handle" placeholder="@storename">

  <label for="niche">Store Niche</label>
  <input type="text" id="niche" placeholder="e.g. Apparel, Skincare, Candles">

  <label for="bio">Profile Bio / Notes</label>
  <textarea id="bio" rows="3" placeholder="Paste store bio or observed pain points (e.g. slow site, bad product photos)..."></textarea>

  <button onclick="generateOutreach()">Generate Pitch Suggestion</button>

  <div id="resultBox" class="output-box" style="display:none;"></div>
  
  <button id="saveBtn" style="display:none; background: #28a745;" onclick="saveToSupabase()">Save Lead to Supabase</button>
</div>

<div class="fab" onclick="window.scrollTo({top: 0, behavior: 'smooth'})">+</div>

<script>
  const SUPABASE_URL = "YOUR_SUPABASE_URL";
  const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
  const supabase = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

  let currentGeneratedMessage = "";

  function generateOutreach() {
    const handle = document.getElementById('handle').value.trim();
    const niche = document.getElementById('niche').value.trim();
    const bio = document.getElementById('bio').value.trim();
    const platform = document.getElementById('platform').value;

    if (!handle) {
      alert("Please enter a store handle.");
      return;
    }

    // Welcoming, non-salesy outreach framework
    currentGeneratedMessage = `Hey ${handle}! 👋 Came across your ${niche || 'store'} on ${platform} and really liked your brand setup.\n\n` +
      `Noticed ${bio ? 'from your profile that ' + bio : 'your product catalog looks solid'}. ` +
      `Quick question—are you currently looking to upgrade your online store experience or boost conversions this month? ` +
      `Would love to share a quick 1-minute visual idea with you if you're open to it!`;

    const resultBox = document.getElementById('resultBox');
    resultBox.innerText = currentGeneratedMessage;
    resultBox.style.display = 'block';
    document.getElementById('saveBtn').style.display = 'block';
  }

  async function saveToSupabase() {
    const platform = document.getElementById('platform').value;
    const handle = document.getElementById('handle').value;
    const niche = document.getElementById('niche').value;
    const bio = document.getElementById('bio').value;

    const { data, error } = await supabase
      .from('store_leads')
      .insert([
        { 
          platform: platform, 
          handle: handle, 
          niche: niche, 
          bio_notes: bio, 
          suggested_message: currentGeneratedMessage 
        }
      ]);

    if (error) {
      alert('Error saving lead: ' + error.message);
    } else {
      alert('Lead successfully saved to Supabase database!');
    }
  }
</script>

</body>
</html>
