# Contributing to Raspberry Pi AI Server Guide

Thank you for your interest in improving this guide!

## 🎯 What We're Looking For

We welcome contributions that:
- ✅ Fix technical inaccuracies
- ✅ Improve clarity and readability
- ✅ Add troubleshooting steps
- ✅ Update software versions
- ✅ Expand hardware compatibility
- ✅ Improve security recommendations

We're **not** accepting:
- ❌ Promotions for commercial products
- ❌ Untested recommendations
- ❌ Content that compromises security

## 🚀 Quick Start

### 1. Fork and Clone
````bash
git clone https://github.com/YOUR_USERNAME/raspberry-pi-ai-server.git
cd raspberry-pi-ai-server
git remote add upstream https://github.com/ORIGINAL_OWNER/raspberry-pi-ai-server.git
````

### 2. Create a Branch
````bash
git checkout -b fix/improve-wireguard-setup
````

**Branch naming:**
- `fix/` - Bug fixes
- `docs/` - Documentation
- `feat/` - New features
- `security/` - Security enhancements

### 3. Make Changes

Edit files and test thoroughly.

### 4. Commit and Push
````bash
git add README.md
git commit -m "fix: correct WireGuard port forwarding instructions"
git push origin fix/improve-wireguard-setup
````

### 5. Open Pull Request

Fill out the PR template and wait for review.

## 📝 Style Guide

### Writing Style

**Do:**
- ✅ Use clear, concise language
- ✅ Write in second person
- ✅ Break complex steps into smaller ones
- ✅ Explain *why*, not just *how*

**Don't:**
- ❌ Use jargon without explanation
- ❌ Assume advanced knowledge
- ❌ Skip safety warnings

### Code Blocks

Always specify language:
````markdown
```bash
sudo apt update
```
```yaml
services:
  ollama:
    image: ollama/ollama:0.1.29
```
````

### Warnings
````markdown
**⚠️ Warning:** Never expose port 11434 to the internet.

**💡 Tip:** Use SSD for 5x faster loading.

**📝 Note:** Requires sudo privileges.
````

## 🧪 Testing Guidelines

Before submitting:
- [ ] Test all commands on real hardware
- [ ] Verify all links work
- [ ] Check code syntax
- [ ] Review for security issues
- [ ] No sensitive information included

## 🔒 Security Considerations

### Never Include:
- Real IP addresses (use 192.168.1.100, 203.0.113.0)
- Actual private keys
- Personal domains
- Real email addresses

### Good Examples:
````bash
PrivateKey = YOUR_PRIVATE_KEY_HERE
Endpoint = YOUR_PUBLIC_IP:51820
````

## 📋 Pull Request Template
````markdown
## Description
Brief description

## Type of Change
- [ ] Bug fix
- [ ] Documentation
- [ ] New feature
- [ ] Security improvement

## Testing Done
- [ ] Tested on Pi 4 (4GB)
- [ ] Verified all commands work
- [ ] Checked all links

## Environment
- Hardware: Raspberry Pi 4 (4GB)
- OS: Raspberry Pi OS Lite 64-bit
- Docker: 24.0.7
````

## 🐛 Reporting Issues

### Before Opening

1. Search existing issues
2. Try troubleshooting section
3. Test with latest versions

### Issue Template
````markdown
**Description**
Clear description

**Steps to Reproduce**
1. Step one
2. Step two

**Environment**
- Hardware: Pi 4 (4GB)
- OS: Raspberry Pi OS Lite
````

## 🙏 Recognition

Contributors are recognized in:
- GitHub contributor graph
- Release notes
- Special mentions for major features

## 📞 Getting Help

- Open a discussion
- Ask in issues with `question` label
- Review typically within 3-5 days

---

**Thank you for making this guide better! 🎉**

