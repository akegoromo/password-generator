
```markdown
# Password Generator

English | [日本語](./README.md)

A client-side password generator with fine-grained control over character composition, exclusion rules, and consecutive character constraints. Built with vanilla JavaScript and cryptographically secure random number generation.

**🔗 Live Demo:** https://pswd-gen.pages.dev  
*(Note: The tool's UI is in Japanese, but modern browsers will automatically offer to translate it. The intuitive UI makes it easy to use even without translation.)*

## Design Philosophy

This tool implements the following core principles:

- **Zero-trust client-side architecture**: All processing occurs in the browser with no server communication
- **Cryptographically secure randomness**: Uses `window.crypto.getRandomValues()` instead of `Math.random()`
- **Strict policy enforcement**: Post-generation validation ensures 100% compliance with all constraints
- **Bulletproof UI design**: Dropdown-only interface eliminates input validation complexity

## Technical Features

**Password Generation Rules:**
- Configurable length (8-32 characters, default: 13)
- Must start with lowercase letter
- Must contain uppercase, lowercase, and digits
- Minimum 2 digits required
- Optional symbol inclusion with customizable exclusions
- Configurable consecutive character limits (uppercase/lowercase independently)

**Implementation Highlights:**
- Single HTML file with no build process
- Fisher-Yates shuffle for unbiased character distribution
- Retry mechanism with constraint validation (max 1000 attempts)
- Responsive design with mobile optimization

## Core Implementation

### Cryptographically Secure Character Selection

```javascript
function getRandomChar(charSet) {
    if (charSet.length === 0) return '';
    const array = new Uint32Array(1);
    window.crypto.getRandomValues(array);
    return charSet[array[0] % charSet.length];
}

Strict Validation Loop

Generated passwords undergo rigorous post-generation validation:

function validatePassword(pwd, config, upperPool, lowerPool, numberPool) {
    // 1. Must start with lowercase
    if (!lowerPool.includes(pwd[0])) return false;
    
    // 2. Minimum digit count verification
    let numberCount = 0;
    for (let char of pwd) {
        if (numberPool.includes(char)) numberCount++;
    }
    if (numberCount < 2) return false;
    
    // 3. Character type requirements
    let hasUpper = pwd.split('').some(c => upperPool.includes(c));
    let hasLower = pwd.split('').some(c => lowerPool.includes(c));
    if (!hasUpper || !hasLower) return false;
    
    // 4. Consecutive character limits
    return !hasExcessiveConsecutive(pwd, upperPool, config.maxConsecutiveUpper) &&
           !hasExcessiveConsecutive(pwd, lowerPool, config.maxConsecutiveLower);
}

Architecture Decision: Why Single File?

    Zero dependencies: No package.json, no build tools, no external libraries
    Maximum portability: Works on any static hosting platform
    Instant deployment: Direct upload to Cloudflare Pages/GitHub Pages
    Clear boundaries: All logic contained in one reviewable file

Usage
For End Users

    Access the live demo URL
    Configure generation parameters via dropdowns
    Generate 30-60 passwords simultaneously
    Copy all passwords to clipboard with one click

For Developers

Local Development:

git clone https://github.com/akegoromo/password-generator.git
cd password-generator
# Open index.html in browser or serve with any static server
python -m http.server 8000  # Example with Python

Deployment:

    Upload index.html to any static hosting service
    No build configuration required
    Works with Cloudflare Pages, GitHub Pages, Netlify, Vercel

Customization: All generation parameters are externalized in the JavaScript configuration object. Modify these values to change behavior:

// Example customization points in index.html
const CONFIG = {
    passwordLength: 13,        // Default length
    includeSymbols: true,      // Symbol usage
    excludedSymbols: "!$%&=",  // Excluded symbols
    maxConsecutiveUpper: 4,    // Max consecutive uppercase (3+1)
    maxConsecutiveLower: 4,    // Max consecutive lowercase (3+1)
    excludedChars: "0OI1l"     // Ambiguous characters
};

Security Considerations
What This Tool Provides

✅ Cryptographically secure random number generation
✅ Client-side processing (no network transmission)
✅ Configurable character exclusions
✅ Strict constraint compliance
✅ Open source code for security review
What This Tool Does NOT Provide

❌ Password strength assessment against real-world attacks
❌ Protection against clipboard monitoring
❌ Guarantee of compliance with specific security policies
❌ Password storage or management
❌ Network security or transmission protection
Browser Compatibility

Requirements:

    ES6+ JavaScript support
    Web Crypto API (window.crypto.getRandomValues)
    CSS Grid and Flexbox support

Tested on:

    Chrome 90+, Firefox 88+, Safari 14+, Edge 90+

Performance

    Generates 60 passwords in <100ms on modern hardware
    Zero network requests after initial load
    Minimal memory footprint (<1MB total)
    Typical constraint satisfaction in <10 retry attempts

Disclaimer

⚠️ IMPORTANT: NO WARRANTY OR LIABILITY

This software is provided "AS IS" without warranty of any kind, express or implied.

USER RESPONSIBILITY:

    Password evaluation: You are solely responsible for determining whether generated passwords meet your security requirements
    Risk assumption: You assume all risks associated with password usage, storage, and management
    Policy compliance: You must verify passwords comply with your organization's security policies
    Security assessment: The author makes no claims about password strength against specific attack vectors

LIMITATION OF LIABILITY: The author(s) shall NOT be liable for any damages, losses, or security breaches arising from:

    Use of passwords generated by this tool
    Use of code copied or modified from this repository
    Deployment of modified versions of this tool
    Any direct, indirect, incidental, or consequential damages

By using this tool or code, you acknowledge and accept these terms.
License

MIT License

Copyright (c) 2026 akegoromo

See LICENSE file for details.

