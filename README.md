<h1 align="center">Bug Bounty Hunting Methodology 2025</h1>

<div align="center">
   
[![Edition](https://img.shields.io/badge/Edition-2025-blue)](#)
[![Author](https://img.shields.io/badge/Author-Amr%20Elsagaei-green)](#)
[![YouTube](https://img.shields.io/badge/AmrSecOfficial-c00?logo=youtube&logoColor=white)](https://www.youtube.com/@AmrSecOfficial)
[![Twitter](https://img.shields.io/badge/amrelsagaei-black?logo=X)](https://twitter.com/amrelsagaei)
[![Telegram](https://img.shields.io/badge/AmrSecOfficial-0088CC?logo=telegram&logoColor=white)](https://t.me/AmrSecOfficial)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-amrelsagaei-0073e3?logo=linkedin&labelColor=white)](https://www.linkedin.com/in/amrelsagaei)

</div>

<br>
<div style="text-align: center;">
    <a href="https://youtu.be/bAHNM8IInwE" target="_blank">
        <img src="https://blog.amrelsagaei.com/storage/posts/webp/bug-bounty-metho-2025.webp" alt="Bug Bounty Methodology 2025 Edition" style="width: 100%;">
    </a>
</div>
<div align="left">
Welcome to the <strong>Bug Bounty Methodology 2025 Edition</strong>! This methodology is a basic guide to help you kickstart your bug bounty journey. It outlines the essential steps to navigate your target effectively, but the real challenge lies in identifying high-impact vulnerabilities through your own skills and creativity. This methodology will be updated regularly as new and interesting techniques emerge to enhance your testing process.
</div>


<br>

## 📜 Table of Contents

| Section | Description |
|---------|-------------|
| 1. [Reconnaissance](#1-reconnaissance-and-subdomain-enumeration) | Subdomain Enumeration & Initial Scanning |
| 2. [Discovery](#2-discovery-and-probing) | HTTP Probing & Asset Discovery |
| 3. [Enumeration](#3-advanced-enumeration-techniques) | Advanced Techniques & Parameter Discovery |
| 4. [Testing](#4-vulnerability-testing) | Vulnerability Assessment |
| 5. [Two-Eye Approach](#5-the-two-eye-approach) | What is that? |
| 6. [POC Creation](#6-proof-of-concept-poc-creation) | Documentation & Evidence |
| 7. [Reporting](#7-reporting) | Final Documentation |

---

<br>


## **1. Reconnaissance and Subdomain Enumeration**

### **1.1 Passive Subdomain Enumeration**
**🛠️Tools:** [Subfinder](https://github.com/projectdiscovery/subfinder), [Amass](https://github.com/OWASP/Amass), [CRTSH](https://crt.sh/), [Github-Search](https://github.com/gwen001/github-search)

**Subfinder**
```bash
subfinder -d target.com -silent -all -recursive -o subfinder_subs.txt
```

**Amass (Passive Mode)**
```bash
amass enum -passive -d target.com -o amass_passive_subs.txt
```

**CRT.sh Query**
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_subs.txt
```

**Github Dorking**
```bash
github-subdomains -d target.com -t YOUR_GITHUB_TOKEN -o github_subs.txt
```

**Results Combination**
```bash
cat *_subs.txt | sort -u | anew all_subs.txt
```

### **1.2 Active Subdomain Enumeration**

**🛠️Tools:** [MassDNS](https://github.com/blechschmidt/massdns), [Shuffledns](https://github.com/projectdiscovery/shuffledns), [DNSX](https://github.com/projectdiscovery/dnsx), [SubBrute](https://github.com/TheRook/subbrute), [FFuF](https://github.com/ffuf/ffuf)

**MassDNS**
```bash
massdns -r resolvers.txt -t A -o S -w massdns_results.txt wordlist.txt
```

**Shuffledns**
```bash
shuffledns -d target.com -list all_subs.txt -r resolvers.txt -o active_subs.txt
```

**DNSX Resolution**
```bash
dnsx -l active_subs.txt -resp -o resolved_subs.txt
```

**SubBrute**
```bash
python3 subbrute.py target.com -w wordlist.txt -o brute_force_subs.txt
```

**FFuF Subdomain**
```bash
ffuf -u https://FUZZ.target.com -w wordlist.txt -t 50 -mc 200,403 -o ffuf_subs.txt
```

### **1.3 Handling Specific (Non-Wildcard) Targets**

**🛠️Tools:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [Katana](https://github.com/projectdiscovery/katana), [Hakrawler](https://github.com/hakluke/hakrawler)

**GAU**
```bash
gau target.example.com | anew gau_results.txt
```

**Waybackurls**
```bash
waybackurls target.example.com | anew wayback_results.txt
```

**Katana**
```bash
katana -u target.example.com -silent -jc -o katana_results.txt
```

**Hakrawler**
```bash
echo "https://target.example.com" | hakrawler -depth 2 -plain -js -out hakrawler_results.txt
```

### **Additional Advanced Techniques**

**🛠️Tools:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Reverse DNS**
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt
```

**ASN Enumeration**
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt
```

**Cloud Asset Enumeration**
```bash
cloud_enum -k target.com
```

**Results Validation**
```bash
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```

---

<br>


## **2. Discovery and Probing**

### **2.1 HTTP Probing**

**🛠️Tools:** [httpx](https://github.com/projectdiscovery/httpx), [httprobe](https://github.com/tomnomnom/httprobe)

**HTTPX Probing**
```bash
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -o live_websites.txt
```

**Custom Filtering**
```bash
cat live_websites.txt | grep -i "login\|admin" | tee login_endpoints.txt
```

### **2.2 JavaScript Analysis**

**🛠️Tools:** [LinkFinder](https://github.com/GerbenJavado/LinkFinder), [subjs](https://github.com/lc/subjs), [JSFinder](https://github.com/Threezh1/JSFinder), [GF](https://github.com/tomnomnom/gf)

**JS Extraction**
```bash
cat live_websites.txt | waybackurls | grep "\.js" | anew js_files.txt
```

**LinkFinder Analysis**
```bash
python3 linkfinder.py -i js_files.txt -o js_endpoints.txt
```

**Sensitive Pattern Search**
```bash
cat js_files.txt | gf aws-keys | tee aws_keys.txt
cat js_files.txt | gf urls | tee sensitive_urls.txt
```

**API Key Validation**
```bash
curl -X GET "https://api.example.com/resource" -H "Authorization: Bearer <extracted_key>"
```

### **2.3 Advanced Google Dorking**

**🛠️Tools:** [GitDorker](https://github.com/obheda12/GitDorker)

**Automated Dorking**
```bash
python3 GitDorker.py -tf <github_token.txt> -q target.com -d dorks.txt -o git_dorks_output.txt
```

**Admin/Login Files**
```bash
site:*.example.com inurl:"*admin | login" | inurl:.php | .asp
```

**Config Files**
```bash
site:*.example.com ext:env | ext:yaml | ext:ini
```

**Public Keys**
```bash
site:*.example.com inurl:"id_rsa.pub" | inurl:".pem"
```

### **2.4 URL Discovery**

**🛠️Tools:** [Katana](https://github.com/projectdiscovery/katana), [Gospider](https://github.com/jaeles-project/gospider), [Hakrawler](https://github.com/hakluke/hakrawler)

**Katana Crawling**
```bash
katana -list live_websites.txt -jc -o katana_urls.txt
```

**Gospider**
```bash
gospider -s "https://target.com" -d 2 -o gospider_output/
```

**Hakrawler**
```bash
echo "https://target.com" | hakrawler -depth 3 -plain -out hakrawler_results.txt
```

### **2.5 Archive Enumeration**

**🛠️Tools:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [ParamSpider](https://github.com/devanshbatham/ParamSpider)

**Archive URL Collection**
```bash
gau --subs target.com | anew archived_urls.txt
waybackurls target.com | anew wayback_urls.txt
```

**Parameter Extraction**
```bash
cat archived_urls.txt | grep "=" | anew parameters.txt
```

---

<br>


## **3. Advanced Enumeration Techniques**

### **3.1 Parameter Discovery**

**🛠️Tools:** [Arjun](https://github.com/s0md3v/Arjun), [ParamSpider](https://github.com/devanshbatham/ParamSpider), [FFuF](https://github.com/ffuf/ffuf)

**Arjun Parameter Discovery**
```bash
arjun -u "https://target.example.com" -m GET,POST --stable -o params.json
```

**ParamSpider Web Parameters**
```bash
python3 paramspider.py --domain target.com --exclude woff,css,js --output paramspider_output.txt
```

**FFuF Parameter Bruteforce**
```bash
ffuf -u https://target.com/page.php?FUZZ=test -w /usr/share/wordlists/params.txt -o parameter_results.txt
```

### **3.2 Cloud Asset Enumeration**

**🛠️Tools:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Cloud Bucket Enumeration**
```bash
cloud_enum -k target.com -b buckets.txt -o cloud_enum_results.txt
```

**S3 Bucket Access Test**
```bash
aws s3 ls s3://<bucket_name> --no-sign-request
```

**S3 Bucket Content Dump**
```bash
python3 AWSBucketDump.py -b target-bucket -o dumped_data/
```

### **3.3 Content Discovery**

**🛠️Tools:** [Feroxbuster](https://github.com/epi052/feroxbuster), [FFuF](https://github.com/ffuf/ffuf), [Dirsearch](https://github.com/maurosoria/dirsearch)

**Feroxbuster**
```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/common.txt -r -t 20 -o recursive_results.txt
```

**Dirsearch**
```bash
dirsearch -u https://target.com -w /usr/share/wordlists/content_discovery.txt -e php,html,js,json -x 404 -o dirsearch_results.txt
```

**FFuF Recursive**
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/content_discovery.txt -mc 200,403 -recursion -recursion-depth 3 -o ffuf_results.txt
```

### **3.4 API Enumeration**

**🛠️Tools:** [Kiterunner](https://github.com/assetnote/kiterunner), [Postman](https://www.postman.com/), [Burp Suite](https://portswigger.net/burp)

**Kiterunner**
```bash
kr scan https://api.target.com -w /usr/share/kiterunner/routes-large.kite -o api_routes.txt
```

### **3.5 ASN Mapping**

**🛠️Tools:** [Amass](https://github.com/OWASP/Amass), [Shodan](https://www.shodan.io/), [Censys](https://censys.io/)

**ASN Lookup**
```bash
amass intel -asn <ASN_Number> -o asn_ips.txt
```

**Shodan Enumeration**
```bash
shodan search "net:<ip_range>" --fields ip_str,port --limit 100
```

**Censys Asset Search**
```bash
censys search "autonomous_system.asn:<ASN_Number>" -o censys_assets.txt
```

---

<br>


## **4. Vulnerability Testing**

### **4.1 High-Priority Vulnerabilities**

**🐞CSRF Testing**
```bash
cat live_websites.txt | gf csrf | tee csrf_endpoints.txt
```

**🐞LFI Testing**
```bash
cat live_websites.txt | gf lfi | qsreplace "/etc/passwd" | xargs -I@ curl -s @ | grep "root:x:" > lfi_results.txt
```

**🐞RCE Testing**
```bash
curl -X POST -F "file=@exploit.php" https://target.com/upload
```

**🐞SQLi Testing**
```bash
ghauri -u "https://target.com?id=1" --dbs --batch
```

**🐞Sensitive Data Search**
```bash
cat js_files.txt | grep -Ei "key|token|auth|password" > sensitive_data.txt
```

**🐞Open Redirect Test**
```bash
cat urls.txt | grep "=http" | qsreplace "https://evil.com" | xargs -I@ curl -I -s @ | grep "evil.com"
```

---

<br>


## **5. The "Two-Eye" Approach 👀**
1. **First Eye:** Focus on testing every gathered subdomain, endpoint, or parameter for common vulnerabilities.
2. **Second Eye:** Identify “interesting” findings like exposed credentials, forgotten subdomains, or admin panels.

### **Actionable Steps:**
- If a vulnerability is identified, create a proof of concept (POC) and test its impact.
- If no vulnerabilities are found, pivot to deeper testing on unique subdomains or endpoints.



---

<br>


## **6. Proof of Concept (POC) Creation**

### **🎥Video POC**

Demonstrate vulnerabilities in action using screen recording tools like Greenshot or OBS Studio.

### **📸Screenshot POC**

Capture clear screenshots with annotations to explain each step.

- **🛠️Tool:** Greenshot.


---

<br>


## **7. Reporting**

### **📝Report Structure**

1. **Executive Summary**
   - Target Scope
   - Testing Timeline
   - Key Findings Summary
   - Risk Ratings

2. **Technical Details**
   - Vulnerability Title
   - Severity Rating
   - Affected Components
   - Technical Description
   - Steps to Reproduce
   - Impact Analysis
   - Supporting Evidence (POC)

3. **Remediation**
   - Detailed Recommendations
   - Mitigation Steps
   - Additional Security Controls
   - References & Resources

4. **Supporting Materials**
   - Video Demonstrations
   - Screenshots & Annotations
   - HTTP Request/Response Logs
   - Code Snippets
   - Timeline of Discovery

### **Best Practices**

- Write clear, concise descriptions
- Include detailed reproduction steps
- Provide actionable remediation advice
- Support findings with evidence
- Use professional formatting
- Highlight business impact
- Include verification steps

### **Report Format**

```markdown
# Vulnerability Report: [Title]

## Overview
- Severity: [Critical/High/Medium/Low]
- CVSS Score: [Score]
- Affected Component: [Component]

## Description
[Detailed technical description]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step n...]

## Impact
[Business and technical impact]

## Proof of Concept
[Screenshots, videos, code]

## Recommendations
[Detailed fix recommendations]

## References
[CVE, CWE, related resources]
```

---

<br>


## 📄 License
All Rights Reserved © 2025 **Amr Elsagaei (AmrSec)**.
