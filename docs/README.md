# Pulse 360° Documentation Index

**Complete documentation guide for IT administrators and developers**  
**Last Updated:** November 17, 2025

Welcome to the Pulse 360° documentation. This index helps you find the right documentation for your needs.

---

## 📚 Documentation Overview

| Document | Audience | Purpose | When to Use |
|----------|----------|---------|-------------|
| [API Documentation](./API-DOCUMENTATION.md) | IT Administrators | Complete API reference with authentication, permissions, and setup | Setting up the application, configuring permissions, understanding authentication flows |
| [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | Developers & IT Admins | Quick lookup of all API endpoints | Troubleshooting API calls, finding endpoint documentation, understanding rate limits |
| [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | IT Administrators | Step-by-step production deployment guide | Deploying to production, ensuring proper configuration, security review |
| [Troubleshooting Guide](./TROUBLESHOOTING.md) | IT Administrators | Common issues and solutions | Diagnosing authentication problems, permission errors, API failures |
| [Quick Start Guide](./quick-start.md) | Developers | Get up and running quickly | First-time setup, local development |
| [Application Installation](./application-installation.md) | IT Administrators | Detailed installation and deployment | Installing on Azure, Kubernetes, or other hosting |
| [Application Walkthrough](./application-walkthrough.md) | End Users | Feature tour and usage guide | Learning application features, training new users |

---

## 🚀 Getting Started Paths

### For IT Administrators Setting Up for the First Time

**Follow this sequence**:

1. **[Quick Start Guide](./quick-start.md)** (10 minutes)
   - Understand basic requirements
   - Set up development environment
   - Verify basic functionality

2. **[API Documentation](./API-DOCUMENTATION.md)** (1-2 hours)
   - Create Azure app registration
   - Configure API permissions
   - Set up Power Platform access
   - Configure environment variables

3. **[Deployment Checklist](./DEPLOYMENT-CHECKLIST.md)** (Follow before production)
   - Verify all configurations
   - Set up security (Key Vault)
   - Plan deployment
   - Execute production deployment

4. **[Troubleshooting Guide](./TROUBLESHOOTING.md)** (As needed)
   - Diagnose any issues encountered
   - Resolve permission errors
   - Fix authentication problems

### For Developers Contributing to the Project

**Follow this sequence**:

1. **[Quick Start Guide](./quick-start.md)** (10 minutes)
   - Clone repository
   - Install dependencies
   - Run local development server

2. **[API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md)** (Reference)
   - Understand which APIs are used where
   - Find endpoint documentation
   - Learn about rate limits

3. **[Copilot Instructions](.github/copilot-instructions.md)** (For AI-assisted development)
   - Understand project structure
   - Learn coding conventions
   - Review authentication patterns

4. **[API Documentation](./API-DOCUMENTATION.md)** (Deep dive)
   - Understand authentication flows
   - Learn token acquisition patterns
   - Implement new API integrations

### For End Users Learning the Application

**Follow this sequence**:

1. **[Application Walkthrough](./application-walkthrough.md)** (30 minutes)
   - Tour of all features
   - Learn navigation
   - Understand key capabilities

2. **Screenshots** (Visual reference)
   - See example screens
   - Understand expected output
   - Compare with your environment

---

## 📖 Documentation by Topic

### Authentication & Security

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **Setting up Azure AD authentication** | [API Documentation](./API-DOCUMENTATION.md) | Authentication Architecture |
| **Creating app registration** | [API Documentation](./API-DOCUMENTATION.md) | Application Registration Setup |
| **Configuring OAuth flows** | [API Documentation](./API-DOCUMENTATION.md) | Microsoft Entra ID |
| **Managing secrets securely** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Security & Secrets Management |
| **Troubleshooting login issues** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Authentication Issues |
| **Understanding token types** | [API Documentation](./API-DOCUMENTATION.md) | Token Types |

### API Permissions & Roles

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **Microsoft Graph permissions** | [API Documentation](./API-DOCUMENTATION.md) | Microsoft Graph API → Required API Permissions |
| **Azure RBAC roles** | [API Documentation](./API-DOCUMENTATION.md) | Azure Resource Manager API → Required API Permissions |
| **Power Platform roles** | [API Documentation](./API-DOCUMENTATION.md) | Power Platform API → Required Power Platform Roles |
| **Dataverse security roles** | [API Documentation](./API-DOCUMENTATION.md) | Dataverse API → Required Permissions |
| **Granting admin consent** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | API Permissions & Consent |
| **Troubleshooting permission errors** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Permission Errors |

### API Integration

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **List of all API endpoints** | [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | All sections |
| **Microsoft Graph endpoints** | [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | Users, Groups, Applications, etc. |
| **Azure Resource Manager endpoints** | [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | Azure Resources, Costs & Billing |
| **Power Platform endpoints** | [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | Power Platform Environments |
| **Dataverse endpoints** | [API Endpoints Reference](./API-ENDPOINTS-REFERENCE.md) | Dataverse (Power Platform) |
| **Rate limits and throttling** | [API Documentation](./API-DOCUMENTATION.md) | Rate Limits (per API section) |
| **API error codes** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Common Error Codes |

### Configuration & Deployment

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **Environment variables** | [API Documentation](./API-DOCUMENTATION.md) | Environment Configuration |
| **Local development setup** | [Quick Start Guide](./quick-start.md) | All sections |
| **Production deployment** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Production Deployment |
| **Azure App Service deployment** | [Application Installation](./application-installation.md) | Deployment sections |
| **Docker/Kubernetes deployment** | [Application Installation](./application-installation.md) | Container sections |
| **Secret management** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Security & Secrets Management |

### Troubleshooting

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **Cannot sign in** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Authentication Issues |
| **403 Forbidden errors** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Permission Errors |
| **Power Platform not loading** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Power Platform Issues |
| **Token errors** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Token Problems |
| **API connection failures** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | API Connection Failures |
| **Rate limiting** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Common Error Codes |
| **Environment configuration** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Environment Configuration |

### Operations & Maintenance

| Topic | Primary Document | Section |
|-------|------------------|---------|
| **Monitoring setup** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Monitoring & Logging |
| **Creating alerts** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Configure Alerts |
| **Secret rotation** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Rotate Secrets |
| **Ongoing maintenance tasks** | [Deployment Checklist](./DEPLOYMENT-CHECKLIST.md) | Ongoing Maintenance |
| **Debugging techniques** | [Troubleshooting Guide](./TROUBLESHOOTING.md) | Debugging Tools & Techniques |

---

## 🔍 Quick Reference Tables

### Environment Variables

| Variable | Required | Purpose | Where Documented |
|----------|----------|---------|------------------|
| `NEXTAUTH_URL` | Yes | Application URL | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |
| `NEXTAUTH_SECRET` | Yes | Session encryption | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |
| `AZURE_CLIENT_ID` | Yes | App registration ID | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |
| `AZURE_CLIENT_SECRET` | Yes | App secret | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |
| `AZURE_TENANT_ID` | Yes | Tenant ID | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |
| `AZURE_SUBSCRIPTION_ID` | No | For Azure Resource Graph | [API Docs](./API-DOCUMENTATION.md#environment-configuration) |

Full list: [API Documentation → Environment Configuration](./API-DOCUMENTATION.md#environment-configuration)

### Required Permissions Summary

| API | Permission Type | Key Permissions | Documented In |
|-----|----------------|-----------------|---------------|
| Microsoft Graph | Delegated | User.Read.All, Directory.Read.All, AuditLog.Read.All | [API Docs](./API-DOCUMENTATION.md#microsoft-graph-api) |
| Azure Resource Manager | Delegated | user_impersonation | [API Docs](./API-DOCUMENTATION.md#azure-resource-manager-api) |
| Azure RBAC | Role Assignment | Reader, Security Reader | [API Docs](./API-DOCUMENTATION.md#required-api-permissions-application) |
| Power Platform | Role | Power Platform Administrator | [API Docs](./API-DOCUMENTATION.md#power-platform-api) |
| Dataverse | Security Role | System Administrator | [API Docs](./API-DOCUMENTATION.md#dataverse-api) |

Full list: [API Documentation → Required Permissions Summary](./API-DOCUMENTATION.md#required-permissions-summary)

### Common Error Codes

| Code | Meaning | Quick Fix | Documented In |
|------|---------|-----------|---------------|
| 401 | Unauthorized | Check token is present and valid | [Troubleshooting](./TROUBLESHOOTING.md#common-error-codes) |
| 403 | Forbidden | Grant required permission | [Troubleshooting](./TROUBLESHOOTING.md#common-error-codes) |
| 429 | Rate Limit | Implement retry with backoff | [Troubleshooting](./TROUBLESHOOTING.md#common-error-codes) |
| AADSTS50011 | Reply URL mismatch | Add redirect URI to app registration | [Troubleshooting](./TROUBLESHOOTING.md#authentication-issues) |
| AADSTS65001 | Consent not granted | Grant admin consent | [Troubleshooting](./TROUBLESHOOTING.md#authentication-issues) |

Full list: [Troubleshooting Guide → Common Error Codes](./TROUBLESHOOTING.md#common-error-codes)

---

## 🔗 External Resources

### Microsoft Official Documentation

- **Microsoft Graph**: [https://learn.microsoft.com/en-us/graph/](https://learn.microsoft.com/en-us/graph/)
- **Azure Resource Manager**: [https://learn.microsoft.com/en-us/azure/azure-resource-manager/](https://learn.microsoft.com/en-us/azure/azure-resource-manager/)
- **Power Platform**: [https://learn.microsoft.com/en-us/power-platform/](https://learn.microsoft.com/en-us/power-platform/)
- **Microsoft Entra ID**: [https://learn.microsoft.com/en-us/azure/active-directory/](https://learn.microsoft.com/en-us/azure/active-directory/)
- **MSAL Node**: [https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-node](https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-node)

### Tools & Utilities

- **Azure Portal**: [https://portal.azure.com](https://portal.azure.com)
- **Microsoft 365 Admin**: [https://admin.microsoft.com](https://admin.microsoft.com)
- **Power Platform Admin**: [https://admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)
- **Microsoft Entra Admin**: [https://entra.microsoft.com](https://entra.microsoft.com)
- **Graph Explorer**: [https://developer.microsoft.com/graph/graph-explorer](https://developer.microsoft.com/graph/graph-explorer)
- **JWT Decoder**: [https://jwt.ms](https://jwt.ms)

### Framework Documentation

- **Next.js**: [https://nextjs.org/docs](https://nextjs.org/docs)
- **NextAuth.js**: [https://next-auth.js.org](https://next-auth.js.org)
- **React**: [https://react.dev](https://react.dev)
- **Tailwind CSS**: [https://tailwindcss.com/docs](https://tailwindcss.com/docs)

---

## 📝 Documentation Maintenance

### Document Ownership

| Document | Owner | Last Updated | Review Frequency |
|----------|-------|--------------|------------------|
| API Documentation | IT Operations Lead | Nov 1, 2025 | Quarterly |
| API Endpoints Reference | Development Lead | Nov 1, 2025 | Quarterly |
| Deployment Checklist | DevOps Lead | Nov 1, 2025 | Quarterly |
| Troubleshooting Guide | Support Lead | Nov 1, 2025 | Monthly |
| Quick Start Guide | Development Lead | - | As needed |
| Application Installation | DevOps Lead | - | Quarterly |
| Application Walkthrough | Product Owner | - | Quarterly |

### Suggesting Documentation Updates

If you find errors or have suggestions for improving documentation:

1. **Open an issue** on GitHub with:
   - Document name
   - Section with issue
   - Description of problem or suggestion
   - Your contact information (optional)

2. **Submit a pull request** with:
   - Updated markdown file(s)
   - Description of changes
   - Reason for update

3. **Email documentation team** at: docs@company.com

### Documentation Standards

All documentation follows these standards:

- **Format**: Markdown (.md files)
- **Location**: `/docs` directory
- **Naming**: UPPERCASE-WITH-HYPHENS.md
- **Linking**: Relative links between docs
- **Updates**: Include "Last Updated" date
- **Review**: Quarterly review cycle
- **Accessibility**: Plain language, clear headings, tables for structured data

---

## 📞 Support

### Getting Help with Documentation

**Documentation unclear?**
- Check the [Troubleshooting Guide](./TROUBLESHOOTING.md) first
- Search for keywords in all documentation
- Review external Microsoft documentation links
- Contact documentation team: docs@company.com

**Technical issues?**
- Follow diagnostic steps in [Troubleshooting Guide](./TROUBLESHOOTING.md)
- Check Application Insights for errors
- Review Azure service health
- Contact IT support: support@company.com

**Security concerns?**
- Review [Security & Secrets Management](./DEPLOYMENT-CHECKLIST.md#security--secrets-management)
- Contact security team immediately: security@company.com
- For client secret exposure, rotate immediately

### Training Resources

**For IT Administrators**:
- Schedule onboarding session with IT Operations team
- Review all documentation in sequence (2-4 hours)
- Practice deployment in staging environment
- Shadow experienced administrator during production deployment

**For Developers**:
- Review Quick Start Guide
- Explore codebase with IDE
- Read Copilot Instructions for AI-assisted development
- Pair program with experienced developer

**For End Users**:
- Watch recorded demo (if available)
- Read Application Walkthrough
- Schedule training session with admin team
- Practice in staging environment before production

---

## ✅ Documentation Checklist

Use this checklist to ensure you've reviewed necessary documentation:

### For Initial Setup
- [ ] Read Quick Start Guide
- [ ] Review API Documentation (Authentication section)
- [ ] Review API Documentation (Environment Configuration)
- [ ] Review API Documentation (Application Registration Setup)
- [ ] Review Deployment Checklist (Pre-Deployment)

### For Production Deployment
- [ ] Complete entire Deployment Checklist
- [ ] Review Security & Secrets Management section
- [ ] Review Troubleshooting Guide (skim for awareness)
- [ ] Bookmark API Endpoints Reference for quick lookup
- [ ] Set up monitoring per Deployment Checklist

### For Troubleshooting
- [ ] Review Troubleshooting Guide relevant section
- [ ] Check API Documentation for permission requirements
- [ ] Verify configuration in Environment Configuration section
- [ ] Use Debugging Tools & Techniques
- [ ] Consult Common Error Codes

### For Ongoing Operations
- [ ] Review Ongoing Maintenance section monthly
- [ ] Check for documentation updates quarterly
- [ ] Update team training materials as needed
- [ ] Contribute improvements to documentation

---

## 🆕 What's New

### November 1, 2025
- ✨ **New**: Comprehensive API Documentation with step-by-step setup
- ✨ **New**: API Endpoints Reference for quick lookup
- ✨ **New**: Production Deployment Checklist
- ✨ **New**: IT Administrator Troubleshooting Guide
- 📝 **Updated**: README with documentation references
- 📝 **Updated**: Environment variable security guidance

### Future Planned Updates
- Video walkthrough series
- Interactive troubleshooting wizard
- Automated deployment scripts
- Environment-specific configuration templates
- API usage analytics and optimization guide

---

## 📄 License & Legal

This documentation is part of the Pulse 360° project and is subject to the same license terms. See [LICENSE](../LICENSE) file for details.

**Confidentiality**: This documentation may contain sensitive information about your organization's Microsoft environment. Treat it as confidential and do not share outside your organization without proper authorization.

---

**Documentation maintained by**: IT Operations & Development Team  
**Last comprehensive review**: November 1, 2025  
**Next scheduled review**: February 1, 2026

For questions about this documentation index or suggestions for improvement, contact: docs@company.com
