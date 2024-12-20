# แบบฝึกหัดที่ 14: การใช้งาน Cloud Armor
# Exercise 14: Using Cloud Armor

## โจทย์ | Problem
ตั้งค่าระบบป้องกัน DDoS และ application security ด้วย Cloud Armor โดยมีความต้องการดังนี้:

1. สร้าง:
   - Security policies
   - WAF rules
   - Custom rules
   - Rate limiting

2. ตั้งค่า:
   - IP allowlist/denylist
   - Geolocation rules
   - Preconfigured rules
   - Adaptive protection

## เฉลย | Solution

### 1. สร้าง Basic Security Policy:
```bash
# Create security policy
gcloud compute security-policies create main-policy \
    --description="Main security policy for web applications"

# Add default rule (deny all)
gcloud compute security-policies rules update 2147483647 \
    --security-policy=main-policy \
    --action=deny-403
```

### 2. ตั้งค่า IP Rules:
```bash
# Allow specific IP ranges
gcloud compute security-policies rules create 1000 \
    --security-policy=main-policy \
    --description="Allow internal IPs" \
    --action=allow \
    --src-ip-ranges="10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

# Block specific countries
gcloud compute security-policies rules create 2000 \
    --security-policy=main-policy \
    --description="Block specific countries" \
    --action=deny-403 \
    --src-ip-ranges=CN,RU \
    --expression="origin.region_code == 'CN' || origin.region_code == 'RU'"
```

### 3. ตั้งค่า Rate Limiting:
```bash
# Create rate limiting rule
gcloud compute security-policies rules create 3000 \
    --security-policy=main-policy \
    --description="Rate limiting" \
    --action=rate-based-ban \
    --rate-limit-threshold-count=100 \
    --rate-limit-threshold-interval-sec=60 \
    --ban-duration-sec=300 \
    --conform-action=allow \
    --exceed-action=deny-403 \
    --enforce-on-key=IP
```

### 4. ตั้งค่า WAF Rules:
```bash
# SQL injection protection
gcloud compute security-policies rules create 4000 \
    --security-policy=main-policy \
    --description="SQL injection protection" \
    --action=deny-403 \
    --expression="evaluatePreconfiguredWaf('sqli-v33-stable', ['owasp-crs-v030301-id942110-sqli','owasp-crs-v030301-id942120-sqli'])"

# XSS protection
gcloud compute security-policies rules create 4100 \
    --security-policy=main-policy \
    --description="XSS protection" \
    --action=deny-403 \
    --expression="evaluatePreconfiguredWaf('xss-v33-stable', ['owasp-crs-v030301-id941110-xss'])"

# Path traversal protection
gcloud compute security-policies rules create 4200 \
    --security-policy=main-policy \
    --description="Path traversal protection" \
    --action=deny-403 \
    --expression="evaluatePreconfiguredWaf('rce-v33-stable', ['owasp-crs-v030301-id930110-lfi'])"
```

### 5. สร้าง Custom Rules:
```bash
# Block specific user agents
gcloud compute security-policies rules create 5000 \
    --security-policy=main-policy \
    --description="Block bad user agents" \
    --action=deny-403 \
    --expression="request.headers['user-agent'].contains('BadBot') || request.headers['user-agent'].contains('Scraper')"

# Require specific headers
gcloud compute security-policies rules create 5100 \
    --security-policy=main-policy \
    --description="Require API key" \
    --action=deny-403 \
    --expression="!request.headers.contains('x-api-key')"

# Custom request validation
gcloud compute security-policies rules create 5200 \
    --security-policy=main-policy \
    --description="Validate request parameters" \
    --action=deny-403 \
    --expression="request.path.matches('/api/.*') && !request.headers.contains('content-type', 'application/json')"
```

### 6. ตั้งค่า Adaptive Protection:
```bash
# Enable adaptive protection
gcloud compute security-policies update main-policy \
    --enable-layer7-ddos-defense \
    --layer7-ddos-defense-rule-visibility=STANDARD \
    --adaptive-protection-auto-deploy=true

# Configure threshold
gcloud compute security-policies rules create 6000 \
    --security-policy=main-policy \
    --description="Adaptive protection" \
    --action=deny-403 \
    --adaptive-protection-config=threshold=90
```

### 7. Apply Policy to Backend Service:
```bash
# Create backend service with security policy
gcloud compute backend-services update web-backend-service \
    --security-policy=main-policy \
    --global

# Or update existing backend service
gcloud compute backend-services update web-backend-service \
    --security-policy=main-policy \
    --global
```

### 8. ตั้งค่า Logging และ Monitoring:
```bash
# Enable detailed logging
gcloud compute security-policies update main-policy \
    --log-level=VERBOSE

# Create monitoring alert
gcloud alpha monitoring policies create \
    --display-name="Cloud Armor Blocks" \
    --condition-display-name="High number of blocked requests" \
    --condition-filter='resource.type="http_load_balancer" AND metric.type="compute.googleapis.com/firewall/policy/action_count"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```bash
# Test IP blocking
curl -I http://your-lb-ip/
curl -I --header "X-Forwarded-For: 1.2.3.4" http://your-lb-ip/

# Test rate limiting
for i in {1..200}; do
    curl -I http://your-lb-ip/
done

# Test WAF rules
curl -X POST "http://your-lb-ip/login" \
    -d "username=admin' OR '1'='1"

# Test custom rules
curl -I -H "User-Agent: BadBot" http://your-lb-ip/

# View logs
gcloud logging read 'resource.type="http_load_balancer"' \
    --limit=10

# Check metrics
gcloud monitoring metrics list \
    --filter="metric.type=compute.googleapis.com/firewall/policy/action_count"
```

## เพิ่มเติม | Additional Notes
- ทำ regular security audits
- ปรับ rules ตาม traffic patterns
- ใช้ preview mode ก่อน deployment
- ตั้งค่า incident response plan
- พิจารณาใช้ Cloud Armor Edge Security
