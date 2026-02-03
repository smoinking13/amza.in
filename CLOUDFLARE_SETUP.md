# CLOUDFLARE SETUP FOR AMZA.IN
# Complete Configuration Guide for Enterprise Security & Performance

## ═══════════════════════════════════════════════════════════
## DOMAIN: amza.in
## Date: February 2026
## Configuration Type: Production Enterprise
## ═══════════════════════════════════════════════════════════

## 1. DNS CONFIGURATION
## ═══════════════════════════════════════════════════════════

### Primary DNS Records

Type    Name    Content                 TTL     Proxy Status
────────────────────────────────────────────────────────────
A       @       YOUR_SERVER_IP          Auto    Proxied (Orange Cloud)
A       www     YOUR_SERVER_IP          Auto    Proxied (Orange Cloud)
AAAA    @       YOUR_IPv6_ADDRESS       Auto    Proxied (Orange Cloud)
AAAA    www     YOUR_IPv6_ADDRESS       Auto    Proxied (Orange Cloud)

### Email Records (if using email)
MX      @       mail.amza.in            Auto    DNS Only
TXT     @       "v=spf1 include:_spf.google.com ~all"    Auto    DNS Only

### Subdomain Records (if needed)
CNAME   api     @                       Auto    Proxied (Orange Cloud)
CNAME   cdn     @                       Auto    Proxied (Orange Cloud)

### Verification Records
TXT     @       "Site verification codes"       Auto    DNS Only

## 2. SSL/TLS CONFIGURATION
## ═══════════════════════════════════════════════════════════

### SSL/TLS Encryption Mode
Setting: Full (Strict)
- Encrypts traffic between visitors and Cloudflare
- Encrypts traffic between Cloudflare and origin server
- Requires valid SSL certificate on origin server

### Edge Certificates
- Universal SSL: Enabled
- Certificate Status: Active
- Type: Cloudflare Universal SSL
- Validity: Auto-renewed

### Origin Server Certificate
Generate a Cloudflare Origin Certificate for your server:
1. Navigate to SSL/TLS > Origin Server
2. Create Certificate
3. Validity: 15 years
4. Hostnames: amza.in, *.amza.in
5. Download and install on your origin server

Certificate files to install on origin:
- Origin Certificate (certificate.pem)
- Private Key (private.key)

### Always Use HTTPS
Setting: ON
- Redirects all HTTP requests to HTTPS

### Minimum TLS Version
Setting: TLS 1.2
- Ensures strong encryption

### Opportunistic Encryption
Setting: ON
- Enables TLS for additional hostnames

### TLS 1.3
Setting: ON
- Uses latest TLS protocol

### Automatic HTTPS Rewrites
Setting: ON
- Automatically rewrites insecure URLs to HTTPS

### Certificate Transparency Monitoring
Setting: ON
- Monitors certificate issuance

## 3. FIREWALL RULES
## ═══════════════════════════════════════════════════════════

### WAF (Web Application Firewall)

#### Managed Rules - OWASP Core Ruleset
Action: Block
Sensitivity: Medium
Enable all OWASP rules

#### Managed Rules - Cloudflare Managed Ruleset
Action: Block
Enable all Cloudflare rules

### Custom Firewall Rules

#### Rule 1: Block Known Bad Bots
Expression:
(cf.client.bot) and not (cf.verified_bot)
Action: Block

#### Rule 2: Rate Limiting - API Protection
Expression:
(http.request.uri.path contains "/api/") and
(rate("requests", 1m) > 100)
Action: Block

#### Rule 3: Geoblocking (if needed)
Expression:
(ip.geoip.country in {"CN" "RU"}) and
(http.request.uri.path contains "/admin")
Action: Block

#### Rule 4: SQL Injection Protection
Expression:
(http.request.uri.query contains "union" or
 http.request.uri.query contains "select" or
 http.request.uri.query contains "script")
Action: Block

#### Rule 5: Allow Only Valid User Agents
Expression:
(http.user_agent eq "") or
(http.user_agent contains "curl") or
(http.user_agent contains "wget")
Action: Block

### Rate Limiting Rules

#### API Rate Limit
Path: /api/*
Requests: 100 per minute per IP
Action: Block for 1 hour

#### Contact Form Rate Limit
Path: /contact.html
Requests: 10 per minute per IP
Action: Challenge

#### General Site Rate Limit
Path: /*
Requests: 200 per minute per IP
Action: Challenge

## 4. SECURITY HEADERS
## ═══════════════════════════════════════════════════════════

### Transform Rules - HTTP Response Headers

#### Add Security Headers
Create rule to add the following headers:

X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self' https:; script-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com https://fonts.googleapis.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https:;

## 5. PAGE RULES
## ═══════════════════════════════════════════════════════════

### Page Rule 1: Cache Everything (Static Assets)
URL Pattern: amza.in/assets/*
Settings:
- Cache Level: Cache Everything
- Edge Cache TTL: 1 month
- Browser Cache TTL: 1 month

### Page Rule 2: Security Level High (Admin Area)
URL Pattern: amza.in/admin*
Settings:
- Security Level: High
- Cache Level: Bypass
- Disable Apps
- Disable Performance

### Page Rule 3: Force HTTPS
URL Pattern: http://*amza.in/*
Settings:
- Always Use HTTPS: On

### Page Rule 4: WWW to Non-WWW Redirect
URL Pattern: www.amza.in/*
Settings:
- Forwarding URL: 301 Redirect to https://amza.in/$1

### Page Rule 5: Optimize Homepage
URL Pattern: amza.in/
Settings:
- Cache Level: Standard
- Browser Cache TTL: 4 hours
- Auto Minify: JS, CSS, HTML

## 6. SPEED OPTIMIZATION
## ═══════════════════════════════════════════════════════════

### Auto Minify
- JavaScript: ON
- CSS: ON
- HTML: ON

### Brotli Compression
Setting: ON
- Better compression than gzip

### Early Hints
Setting: ON
- Speeds up page loading

### HTTP/2
Setting: ON
- Multiplexing support

### HTTP/3 (QUIC)
Setting: ON
- Latest protocol

### 0-RTT Connection Resumption
Setting: ON
- Faster subsequent connections

### WebSockets
Setting: ON
- For real-time features

### Rocket Loader
Setting: OFF
- Can cause issues with custom JS

### Mirage
Setting: ON
- Lazy loads images

### Polish
Setting: Lossless
- Optimizes images

### Image Resizing
Setting: ON
- Serves responsive images

## 7. CACHING CONFIGURATION
## ═══════════════════════════════════════════════════════════

### Caching Level
Setting: Standard
- Respects origin cache headers

### Browser Cache TTL
Setting: 4 hours
- Balance between freshness and performance

### Cache Everything Page Rules
Applied to: /assets/*
- Cache all static resources

### Bypass Cache for Dynamic Content
Applied to:
- /contact.html (POST requests)
- Any pages with user-specific content

### Purge Cache
Method: Selective purge by URL or tag
- Use after deployments

## 8. BOT PROTECTION
## ═══════════════════════════════════════════════════════════

### Bot Fight Mode
Setting: ON
- Challenges suspicious bots

### Super Bot Fight Mode
Setting: Enabled (if on paid plan)
- Advanced bot detection

### Verified Bots
- Allow: Googlebot, Bingbot, etc.
- Configure in Bot Management

### Challenge Passage
Setting: 30 minutes
- Time verified users can browse

## 9. DDOS PROTECTION
## ═══════════════════════════════════════════════════════════

### DDoS Protection
Status: Automatic (Always On)
- L3/L4 DDoS protection included
- L7 DDoS protection automatic

### Advanced DDoS Protection
Configure sensitivity: Medium-High
- Protects against application-layer attacks

### HTTP DDoS Attack Protection
- Automatic detection and mitigation
- No configuration needed

## 10. ANALYTICS & MONITORING
## ═══════════════════════════════════════════════════════════

### Cloudflare Analytics
- Monitor traffic patterns
- Track security threats
- Review cache performance

### Security Events
- Review blocked requests
- Analyze attack patterns
- Adjust rules as needed

### Performance Metrics
- Track page load times
- Monitor bandwidth usage
- Review cache hit ratio

## 11. WORKERS (OPTIONAL ADVANCED)
## ═══════════════════════════════════════════════════════════

### Use Cases for Cloudflare Workers
- A/B testing
- Personalization
- API routing
- Advanced security logic
- Server-side rendering

### Example Worker for Security Headers
```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const response = await fetch(request)
  const newHeaders = new Headers(response.headers)
  
  newHeaders.set('X-Content-Type-Options', 'nosniff')
  newHeaders.set('X-Frame-Options', 'SAMEORIGIN')
  newHeaders.set('X-XSS-Protection', '1; mode=block')
  newHeaders.set('Strict-Transport-Security', 'max-age=31536000')
  
  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers: newHeaders
  })
}
```

## 12. DEPLOYMENT CHECKLIST
## ═══════════════════════════════════════════════════════════

### Pre-Launch
- [ ] DNS records configured
- [ ] SSL/TLS Full (Strict) enabled
- [ ] Origin certificate installed
- [ ] Firewall rules tested
- [ ] Security headers configured
- [ ] Page rules active
- [ ] Cache purged

### Post-Launch
- [ ] Test all pages (HTTP → HTTPS redirect)
- [ ] Verify SSL certificate
- [ ] Check security headers (securityheaders.com)
- [ ] Test SSL configuration (ssllabs.com)
- [ ] Monitor Cloudflare Analytics
- [ ] Review firewall events
- [ ] Check cache hit ratio

### Ongoing Maintenance
- [ ] Review security events weekly
- [ ] Update firewall rules as needed
- [ ] Monitor performance metrics
- [ ] Purge cache after deployments
- [ ] Review and optimize page rules monthly

## 13. EMERGENCY PROCEDURES
## ═══════════════════════════════════════════════════════════

### Under Attack Mode
When experiencing DDoS attack:
1. Navigate to Security > Settings
2. Enable "I'm Under Attack Mode"
3. All visitors see challenge page
4. Disable after attack subsides

### Rate Limit Adjustments
If legitimate traffic is blocked:
1. Review firewall events
2. Adjust rate limit thresholds
3. Whitelist specific IPs if needed

### Cache Purge
After emergency fixes:
1. Navigate to Caching
2. Select "Purge Everything"
3. Confirm purge
4. Wait 30 seconds for propagation

## 14. SUPPORT & RESOURCES
## ═══════════════════════════════════════════════════════════

### Cloudflare Support
- Community: community.cloudflare.com
- Documentation: developers.cloudflare.com
- Status Page: www.cloudflarestatus.com

### Security Testing Tools
- SSL Labs: ssllabs.com/ssltest
- Security Headers: securityheaders.com
- DNS Check: dnschecker.org

## 15. COST OPTIMIZATION
## ═══════════════════════════════════════════════════════════

### Free Plan Features
✓ Unlimited DDoS protection
✓ Universal SSL
✓ CDN
✓ Basic WAF
✓ Page Rules (3)

### Pro Plan ($20/month) - Recommended
✓ Everything in Free
✓ WAF custom rules
✓ More page rules (20)
✓ Image optimization
✓ Mobile optimization
✓ Email support

### Business Plan ($200/month) - Enterprise
✓ Everything in Pro
✓ 100% uptime SLA
✓ PCI compliance
✓ Priority support
✓ Advanced DDoS

## NOTES
## ═══════════════════════════════════════════════════════════

- Replace YOUR_SERVER_IP with actual server IP address
- Replace YOUR_IPv6_ADDRESS with actual IPv6 (if available)
- Adjust security rules based on traffic patterns
- Monitor and optimize settings continuously
- Keep Cloudflare account credentials secure
- Enable 2FA on Cloudflare account

## IMPLEMENTATION PRIORITY
## ═══════════════════════════════════════════════════════════

CRITICAL (Implement First):
1. DNS Configuration
2. SSL/TLS Full (Strict)
3. Always Use HTTPS
4. Basic Firewall Rules
5. Security Headers

HIGH PRIORITY:
6. Page Rules
7. Caching Configuration
8. Bot Protection
9. Rate Limiting

MEDIUM PRIORITY:
10. Speed Optimization
11. Analytics Setup
12. Advanced Firewall Rules

OPTIONAL:
13. Cloudflare Workers
14. Load Balancing
15. Argo Smart Routing

═══════════════════════════════════════════════════════════
END OF CLOUDFLARE CONFIGURATION
═══════════════════════════════════════════════════════════
