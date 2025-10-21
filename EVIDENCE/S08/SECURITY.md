# Security Policy

## Reporting a Vulnerability

**Private Disclosure Process**

We take the security of our software seriously. If you believe you have found 
a security vulnerability, please disclose it via our security email:

**idderezovskiy@edu.hse.ru**

Please do not report security vulnerabilities through public GitHub issues.

**What to Include**

When reporting a vulnerability, please include:
- Project version
- Vulnerability description
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

**Response Timeline**

We will acknowledge receipt of your vulnerability report within 48 hours
and send a more detailed response within 72 hours indicating next steps.

## Security Updates

Security updates will be delivered through:
- GitHub security advisories
- Version tags in this repository
- Release notes with security mentions

---

## Secret Rotation Procedure

### Памятка по ротации секретов

**Ответственные лица:**
- Lead DevOps Engineer - первичная ответственность
- Security Officer - надзор и аудит
- Team Lead - резервная ответственность

#### Экстренная ротация при утечке

**1. Немедленные действия:**
```bash
# 1. Отозвать компрометированные секреты в системах:
- GitHub Secrets → Settings → Secrets and variables → Actions
- CI/CD variables (GitLab/GitHub)
- Cloud provider IAM keys
- Database credentials

# 2. Сгенерировать новые ключи:
openssl rand -base64 32  # для случайных токенов
aws iam create-access-key  # для AWS ключей
