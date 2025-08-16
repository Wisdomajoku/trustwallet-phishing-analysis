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
  <p><em><img src="https://i.imgur.com/8LywfPS.png" alt="Screenshot of Thunderbird email" width="600">
</em></p>

  <h2>2. Header Analysis (Step-by-Step)</h2>
  <p>
    The full header was extracted and reviewed. For clarity, it was loaded into MXToolbox, 
    but manual interpretation was done to avoid over-reliance on automated tools.
  </p>
  <!-- Insert MXToolbox screenshot here -->
  <p><em><img src="https://i.imgur.com/w8kFj96.png" alt="Screenshot of header 1" width="600"></em></p>
<p><em><img src="https://i.imgur.com/j2qNpyC.png" alt="Screenshot of header 1" width="600"></em></p>
  
  <h3>Received Chain</h3>
  <p>
  <li>  Zendesk origin: zendesk.com → mta-out15.pod28.euc1.zdsys.com</li>
  <li>  Outbound IP: 188.172.138.40 (verified via SPF as alquiaga.com) </li>
 <li> Microsoft relays: DB1PEPF000509E4.mail.protection.outlook.com → DU2PR04CA0054.outlook.office365.com → CY5PR20MB4867.namprd20.prod.outlook.com → final mailbox delivery. </li> 
  </p>

  <h3>Authentication Results</h3>
  <ul>
    <li><strong>SPF:</strong> Pass (alignment issue, sender IP not fully authorized)</li>
    <li><strong>DKIM:</strong> Pass (but domain alignment questionable)</li>
    <li><strong>DMARC:</strong> Not aligned (failed to enforce policy)</li>
    <li><strong>ARC:</strong> No ARC headers present</li>
  </ul>

  <p>
    The DKIM pass initially looks good, but without proper DMARC alignment, it doesn’t mean 
    the email is legitimate. Attackers can exploit partial authentication to bypass filters.
  </p>

  <!-- BEGIN: EXPANDED AUTHENTICATION DETAILS (SOC VOICE) -->
  <div>
    <h4>Authentication — expanded SOC interpretation</h4>
    <p>Key header fields observed and what they mean to a SOC analyst:</p>
    <ul>
      <li><strong>Return-Path / Envelope From:</strong> <code>alquiaga@alquiaga.com</code> — this is the envelope sender used for SPF checks; it indicates which domain's SPF record was consulted.</li>
      <li><strong>DKIM signature:</strong> <code>d=zendesk.com; s=zendesk1</code> — mail was cryptographically signed by Zendesk keys. That proves the message body/headers were not altered after signing, but the signing domain (zendesk.com) does not match the impersonated brand (trustwallet.com).</li>
    </ul>
    <p><strong>SOC logic:</strong> SPF verifies the envelope sender's authorization (Return-Path domain). DKIM proves integrity and the signing domain. DMARC ties these together and enforces policy when alignment rules are met. If DMARC is not strict or absent, a message with valid SPF/DKIM for a non-brand domain can still impersonate a brand and be delivered. Authentication passing here is a signal that the infrastructure used to send the message is legitimate for that envelope domain — not that the content is trustworthy.</p>
  </div>
  <!-- END: EXPANDED AUTHENTICATION DETAILS -->

  <h2>3. Email Body and CTA</h2>
  <p>
    The message contained urgent language: <em>"Failure to complete this verification within 24 hours 
     may result in restricted access to your wallet"</em>. This is a typical phishing <strong>Call-To-Action (CTA)</strong> 
    meant to create fear and urgency. 
  </p>
  <p>
    The "Verify" link redirected to a non-TrustWallet domain:
    <code>https://www.fastresponseplumbingsf.com/best-plumber/</code>.
  </p>

  <!-- BEGIN: EXPANDED IOCs -->
  <h2>4. Indicators of Compromise (IOCs)</h2>
  <p>Below are expanded IOCs collected from both the header and the body. All items are presented and sanitized for safe sharing.</p>

  <h3>Header / Infrastructure IOCs</h3>
  <ul>
    <li><strong>Return-Path (Envelope From):</strong> <code>alquiaga@alquiaga.com</code></li>
    <li><strong>From (Header From / Display):</strong> <code>Trust-Wallet &lt;alquiaga@alquiaga.com&gt;</code> — display name impersonates Trust Wallet while the domain does not.</li>
    <li><strong>Sending IP (public MX/SMTP hop):</strong> <code>188.172.138.40</code> — seen as <code>mta-out10.pod28.euc1.zdsys.com</code> (Zendesk outbound MTA).</li>
    <li><strong>Other Zendesk hosts observed internally:</strong> <code>mta-out15.pod28.euc1.zdsys.com</code> (internal origin), HELO names consistent with Zendesk outbound pools.</li>
    <li><strong>Microsoft front-door relays:</strong> <code>DB1PEPF000509E4.mail.protection.outlook.com</code>, <code>DU2PR04CA0054.outlook.office365.com</code>, <code>CY5PR20MB4867.namprd20.prod.outlook.com</code>, <code>SN7PR20MB5311.namprd20.prod.outlook.com</code> — these are the receiving protection/relay hops.</li>
    <li><strong>DKIM:</strong> <code>d=zendesk.com; s=zendesk1</code> (signature verified)</li>
    <li><strong>SPF observation:</strong> Received-SPF indicates the envelope domain designated the Zendesk IP as permitted (per the Outlook check).</li>
  </ul>

  <h3>Body / Content IOCs</h3>
  <ul>
    <li><strong>Displayed Brand:</strong> "Trust Wallet" (used in display name and body text)</li>
    <li><strong>Subject line:</strong> <code>Important Notice: Trust Wallet Security Update Required</code></li>
    <li><strong>CTA / Redirect chain:</strong> initial redirect via Bing redirector then landing on <code>fastresponseplumbingsf.com</code> (defanged in the report).</li>
    <li><strong>Keywords used in lure:</strong> <code>verify</code>, <code>security</code>, <code>verify your wallet</code>, <code>24 hours</code>, <code>account locked</code>, <code>confirm</code>, <code>restricted access</code> — useful for detection rules.</li>
    <li><strong>Phishing theme:</strong> Possible credential/seed-phrase harvest under the guise of account protection.</li>
  </ul>

  <!-- END: EXPANDED IOCs -->
<section>
  <h2>Lessons Learned</h2>
  <ul>
    <li><strong>Header Analysis vs Content Scanning:</strong> Checking only the email body is not enough. Attackers may hide payloads in headers, links, or attachments. Best results come from analyzing both headers and content.</li>
    <li><strong>Authentication Alone Is Insufficient:</strong> SPF, DKIM, and DMARC may pass even if the email is malicious. A “PASS” does not always mean safe.</li>
    <li><strong>Received Chain Trust:</strong> The <code>Received</code> headers show the path the email took. This helps confirm if it came from the real source or a suspicious detour.</li>
    <li><strong>IOC Value:</strong> IPs, domains, HELO names, and URLs provide key clues. Blocking one IP is not enough—attackers rotate infrastructure quickly.</li>
    <li><strong>Operational Caution:</strong> Copying headers into tools can sometimes break formatting. Always handle raw headers carefully.</li>
  </ul>
</section>

<section>
  <h2>Remediation Steps</h2>
  <ul>
    <li><strong>Multi-Layered Filtering:</strong> Use SPF, DKIM, DMARC together with content scanning and behavior analysis.</li>
    <li><strong>Threat Intelligence Correlation:</strong> Check IOCs (IPs, domains, URLs) against threat feeds for blocking and alerts.</li>
    <li><strong>Segmentation of Indicators:</strong> Monitor domains and reputation, not just single IPs, to catch recurring patterns.</li>
    <li><strong>Awareness & SOC Procedures:</strong> Teach users that “PASS” ≠ safe. Analysts should always review full headers and escalate if needed.</li>
    <li><strong>Automation Support:</strong> Automate IOC extraction from headers and authentication results to save time and reduce errors.</li>
  </ul>
</section>

</body>
</html>
