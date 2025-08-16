<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
</head>
<body>

  <h1>Trust Wallet Phishing Email Analysis</h1>
  <p>
    This project documents a real-world phishing email sample that was carefully analyzed, sanitized, 
    and redacted for educational purposes. The email pretended to come from Trust Wallet, 
    urging the user to click a verification link within 24 hours. 
  </p>
  <p>
    The objective of this analysis is to demonstrate how email header investigation, 
    combined with content review, reveals the true source and intent of such attacks.
  </p>

  <h2>1. Email Preview (Thunderbird)</h2>
  <p>
    Below is the screenshot of how the phishing email appeared in Thunderbird. 
    At first glance, it looked convincing with Trust Wallet branding and urgent language.
  </p>
  <!-- Insert Thunderbird screenshot here -->
  <p><em>[Insert Thunderbird screenshot here]</em></p>

  <h2>2. Header Analysis (Step-by-Step)</h2>
  <p>
    The full header was extracted and reviewed. For clarity, it was loaded into MXToolbox, 
    but manual interpretation was done to avoid over-reliance on automated tools.
  </p>
  <!-- Insert MXToolbox screenshot here -->
  <p><em>[Insert MXToolbox screenshot here]</em></p>

  <h3>Received Chain</h3>
  <p>
    The <code>Received:</code> fields tell the real journey of the email. 
    While the visible sender claimed to be Trust Wallet, the hops showed delivery 
    through an unrelated domain/server: <strong>fastresponseplumbingsf.com</strong>. 
    This mismatch is a strong indicator of fraud.
  </p>

  <h3>Authentication Results</h3>
  <ul>
    <li><strong>SPF:</strong> SoftFail (alignment issue, sender IP not fully authorized)</li>
    <li><strong>DKIM:</strong> Pass (but domain alignment questionable)</li>
    <li><strong>DMARC:</strong> Not aligned (failed to enforce policy)</li>
    <li><strong>ARC:</strong> No ARC headers present</li>
  </ul>

  <p>
    The DKIM pass initially looks good, but without proper DMARC alignment, it doesn’t mean 
    the email is legitimate. Attackers can exploit partial authentication to bypass filters.
  </p>

  <h2>3. Email Body and CTA</h2>
  <p>
    The message contained urgent language: <em>"Verify My Wallet within 24 hours 
    or risk losing access"</em>. This is a typical phishing <strong>Call-To-Action (CTA)</strong> 
    meant to create fear and urgency. 
  </p>
  <p>
    The "Verify" link redirected to a non-TrustWallet domain:
    <code>https://www.fastresponseplumbingsf.com/best-plumber/</code>.
  </p>

  <h2>4. Indicators of Compromise (IOCs)</h2>
  <ul>
    <li>Malicious URL: <code>fastresponseplumbingsf.com</code></li>
    <li>Sender mismatch: Email claimed Trust Wallet branding, but header showed external domain</li>
    <li>Urgent CTA: "Verify within 24 hours" (psychological trigger)</li>
  </ul>

  <h2>5. Lessons Learned</h2>
  <ol>
    <li>Scanning only the <strong>email body</strong> is not enough — the header reveals the true sender.</li>
    <li>Relying solely on <strong>authentication results</strong> is insufficient — attackers can pass DKIM while failing alignment.</li>
    <li>Header analysis + content analysis together provide stronger evidence.</li>
    <li>Blocking only the sending IP is not enough — attackers rotate infrastructure constantly.</li>
    <li>All-round analysis (headers, content, URLs, reputation checks) is the best defense.</li>
  </ol>

  <h2>6. Notes & Caution</h2>
  <ul>
    <li>When copying headers into online tools like MXToolbox, be careful. Extra spaces or broken formatting can change the results.</li>
    <li>This sample has been fully sanitized and redacted. Sensitive identifiers are removed for safe sharing.</li>
  </ul>

  <h2>7. Repository</h2>
  <p>
    This analysis is maintained as a GitHub project. 
    You can find the repository here: 
    <a href="https://github.com/your-username/phishing-email-analysis">
      github.com/your-username/phishing-email-analysis
    </a>
  </p>

</body>
</html>
