# Pulse 360° Screenshot Gallery

Visual walkthrough of all portal features and capabilities in dark mode.

## Table of Contents
- [Dashboard & Home](#dashboard--home)
- [User Management](#user-management)
- [Group Management](#group-management)
- [Device Management](#device-management)
- [Security & Compliance](#security--compliance)
- [Azure Resources](#azure-resources)
- [Power Platform](#power-platform)
- [Microsoft Fabric](#microsoft-fabric)
- [Intune Management](#intune-management)
- [Audit & Monitoring](#audit--monitoring)
- [Reports & Analytics](#reports--analytics)

---

## Dashboard & Home

### Home / Login
![](home.png)

**Capabilities:**
- Secure authentication with Microsoft Entra ID
- Demo login for testing (russdemo/russadmin)
- Automatic token refresh and session management
- Multi-tenant tenant selection

---

## User Management

### Users List
![](users.png)

**Capabilities:**
- View all Microsoft 365 users
- Search and filter users
- Paginated results with customizable columns
- Quick access to user details
- View enabled/disabled status
- Job title and email information

### Create User
![](users__create-user.png)

**Capabilities:**
- Create new Microsoft 365 users
- Set display name, username, password
- Assign job titles and departments
- Configure account enabled/disabled status
- Automatic validation and error handling

### User Sign-Ins
![](user-signins.png)

**Capabilities:**
- Monitor user authentication activity
- View successful and failed sign-in attempts
- Filter by date range and user
- See location, device, and application details
- Detect suspicious sign-in patterns

### User Registration Details
![](user-registration-details.png)

**Capabilities:**
- View MFA registration status
- Track authentication methods registered
- Monitor security info registration
- Identify users without MFA

---

## Group Management

### Groups
![](groups.png)

**Capabilities:**
- View all Microsoft 365 and Security groups
- Filter by group type (Microsoft 365, Security, Distribution)
- View dynamic vs assigned membership
- Search and pagination
- Mail-enabled status
- Member count information

---

## Device Management

### Devices Overview
![](devices.png)

**Capabilities:**
- View all Intune-managed devices
- Device compliance status
- Operating system information
- Last check-in time
- Device ownership (Corporate/Personal)
- Management state

### Device Management Dashboard
![](device-management.png)

**Capabilities:**
- Centralized device management hub
- Quick access to all device features
- Compliance overview
- Configuration status
- Remote actions available
- Enrollment trends

### Device Actions
![](device-management_actions.png)

**Capabilities:**
- Remote device actions (restart, shutdown, sync)
- View action history and status
- Bulk device operations
- Action scheduling
- Success/failure tracking

### Compliance Status
![](device-management__compliance-status.png)

**Capabilities:**
- Device compliance policy status
- View compliant vs non-compliant devices
- Compliance policy details
- Remediation actions
- Grace period tracking

### Encryption Report
![](device-management__encryption-report.png)

**Capabilities:**
- BitLocker/FileVault status
- Encryption key recovery
- Encryption readiness assessment
- Compliance with encryption policies

### Enrollment Report
![](device-management__enrollment-report.png)

**Capabilities:**
- Device enrollment trends
- Enrollment method breakdown
- Failed enrollment troubleshooting
- Enrollment status tracking

### Policy Assignments
![](device-management__policy-assignments.png)

**Capabilities:**
- View assigned compliance policies
- Configuration profile assignments
- Group targeting information
- Assignment conflicts detection

### Reports
![](device-management__reports.png)

**Capabilities:**
- Comprehensive device reports
- Custom report generation
- Export to CSV/Excel
- Scheduled report delivery
- Historical trend analysis

### Windows Update Status
![](device-management__windows-update-status.png)

**Capabilities:**
- Windows Update compliance
- Feature update status
- Quality update deployment
- Update ring assignments
- Deferral period tracking

### Intune Audit Logs
![](device-management__intune-audit-logs.png)

**Capabilities:**
- Complete audit trail of Intune changes
- Track policy modifications
- User activity monitoring
- Change history for compliance

---

## Security & Compliance

### Security Alerts
![](security-alerts.png)

**Capabilities:**
- Real-time security threat alerts
- Microsoft Defender alerts
- Alert severity and status
- Investigation and remediation actions
- Threat intelligence integration

### Secure Score
![](secure-score.png)

**Capabilities:**
- Microsoft Secure Score dashboard
- Security posture assessment
- Improvement actions prioritized by impact
- Score trends over time
- Comparison with similar organizations

### Secure Score - Defender
![](secure-score__defender.png)

**Capabilities:**
- Microsoft Defender-specific recommendations
- Endpoint security score
- Threat protection improvements
- Automated investigation settings

### Risky Users
![](risky-users.png)

**Capabilities:**
- Identity Protection risk detection
- View users flagged for suspicious activity
- Risk level (High, Medium, Low)
- Risk state (At risk, Remediated, Dismissed)
- Remediation actions (Reset password, Block user)

### Conditional Access
![](conditional-access.png)

**Capabilities:**
- View and manage Conditional Access policies
- Policy status (Enabled, Disabled, Report-only)
- User and group assignments
- Cloud app assignments
- Grant and session controls

### Conditional Access Blocked Sign-Ins
![](conditional-access-blocked-signins.png)

**Capabilities:**
- View sign-ins blocked by Conditional Access
- Troubleshoot access issues
- Policy that blocked the sign-in
- User and application details
- Remediation recommendations

### MFA Sign-In Success
![](mfa-signin-success.png)

**Capabilities:**
- Monitor successful MFA authentications
- Verify MFA enforcement
- Authentication method used
- Location and device information

### MFA Sign-In Failure
![](mfa-signin-failure.png)

**Capabilities:**
- Investigate failed MFA attempts
- Identify user friction points
- Authentication method failures
- Fraud alert triggers

---

## Azure Resources

### Azure Resources Explorer
![](azure__resources.png)

**Capabilities:**
- Query Azure resources with KQL (Kusto Query Language)
- Filter by subscription, resource group, type
- Custom query builder
- Export results to CSV/JSON
- Save and share queries

### Azure Advisor
![](azure__advisor.png)

**Capabilities:**
- Personalized Azure best practice recommendations
- Cost optimization suggestions
- Security improvements
- Performance enhancements
- Operational excellence guidance

### Azure Advisor Score
![](azure__advisor-score.png)

**Capabilities:**
- Overall Azure health score
- Category-specific scores (Cost, Security, Reliability, Performance, Operational Excellence)
- Score trends over time
- Impact analysis

### Azure Recommendations
![](azure__recommendations.png)

**Capabilities:**
- Detailed recommendation list
- Priority-based sorting
- Estimated cost savings
- Implementation guidance
- Automated remediation options

### Azure Billing & Usage
![](azure-billing_and_usage.png)

**Capabilities:**
- Azure cost analysis and forecasting
- Subscription-level spending
- Resource group cost breakdown
- Budget alerts and thresholds
- Cost trend visualization

### Azure Cost Details
![](azure-cost1.png)

**Capabilities:**
- Granular cost details by service
- Daily/monthly cost trends
- Cost allocation by tags
- Anomaly detection
- Export billing data

### Azure Cost Analysis (Enhanced)
![](azure-cost2.png)

**Capabilities:**
- Rate limit handling with automatic retry
- Visual countdown timer during cooldown periods
- Multiple time period selection (7, 30, 60, 90 days)
- Cost forecasting with trend analysis
- Resource group and service-level breakdown
- Export to CSV for further analysis

---

## Power Platform

### Power Platform Hub
![](power-platform.png)

**Capabilities:**
- Centralized Power Platform management
- Quick access to environments, apps, flows
- Governance and compliance tools
- Licensing and capacity overview
- DLP policy management

### Environments
![](power-platform__environments.png)

**Capabilities:**
- View all Power Platform environments
- Environment type (Production, Sandbox, Developer, Teams, Trial)
- Database details (Dataverse, none)
- Storage capacity usage
- Security group assignments
- Environment lifecycle management

### Environments Expanded
![](power-platform__environments-expanded.png)

**Capabilities:**
- Detailed environment information
- Linked Teams environments
- Custom connector counts
- Flow and app inventories
- Admin center quick links
- Managed environment status

### Environments - Copilot Agents
![](power-platform__environments-agents.png)

**Capabilities:**
- View Copilot Studio agents per environment
- Agent status and configuration
- Conversation transcripts and analytics
- Agent performance metrics
- Integration with Dataverse
- Agent deployment tracking

### Environments - Governance
![](power-platform__environments-governance.png)

**Capabilities:**
- Managed environment indicators
- Governance policies applied
- Environment lifecycle status
- Compliance and security posture
- DLP policy assignments
- Admin role assignments

### Connectors
![](power-platform__connectors.png)

**Capabilities:**
- View all connectors in an environment
- Standard and custom connector inventory
- Connection references and dependencies
- Connector permissions and sharing
- API connection status
- Identify unused connectors

### Environment Details
![](power-platform_environment-details.png)

**Capabilities:**
- Complete environment configuration
- Security settings
- Database properties
- Dataverse organization details
- Environment features enabled
- Backup/restore options

### Environment Resources
![](power-platform_environment-details-resources.png)

**Capabilities:**
- Apps in environment
- Flows (Cloud flows, Desktop flows)
- Custom connectors
- Dataverse tables and relationships
- Solutions deployed
- Capacity consumption breakdown

### Flows
![](power-platform_flows.png)

**Capabilities:**
- List all Power Automate flows
- Flow status (On, Off, Suspended)
- Flow type (Cloud, Desktop, Automated, Scheduled, Manual)
- Last run status and timestamp
- Owner information
- Trigger and action count
- Environment filtering

### Flow Details
![](power-platform_flow-details.png)

**Capabilities:**
- Complete flow definition
- Trigger configuration
- Action details with inputs/outputs
- Connection references
- Run history (last 28 days)
- Flow dependencies

### Flow Details - Advanced
![](power-platform_flow-details2.png)

**Capabilities:**
- Flow diagram visualization
- Advanced troubleshooting
- Variable initialization values
- Request trigger schemas
- Error handling configuration
- Export flow definition

### DLP Policies
![](power-platform__dlp-policies.png)

**Capabilities:**
- Data Loss Prevention policies
- Connector classification (Business, Non-Business, Blocked)
- Environment scope
- Policy exceptions
- Audit and compliance reporting

### Tenant Capacity
![](power-platform__tenant-capacity.png)

**Capabilities:**
- Overall tenant capacity usage
- Database capacity (GB)
- File capacity (GB)
- Log capacity (GB)
- Add-on capacity purchased
- Capacity allocation by environment

### Licensing - Tenant Capacity
![](power-platform__licensing__tenant-capacity.png)

**Capabilities:**
- Detailed capacity breakdown
- Capacity sources (subscriptions, add-ons, trials)
- Capacity consumption trends
- Overage alerts
- Capacity purchase recommendations

### Apps
![](power-platform_apps.png)

**Capabilities:**
- Canvas and Model-driven apps
- App usage statistics
- Published vs draft versions
- Owner and co-owners
- Environment assignment
- App permissions
- App type identification

### Inventory Reports
![](power-platform__inventory-reports.png)

**Capabilities:**
- Cross-environment inventory
- App and flow discovery
- Orphaned resource identification
- Usage analytics
- Compliance reporting

### Inventory Dashboard
![](power-platform__inventory.png)

**Capabilities:**
- Comprehensive resource inventory across all environments
- Top resource owners by count
- Environment resource distribution
- Recently created resources (24 hours)
- Resource type breakdown
- Advanced KQL-based queries using Power Platform Resource Query API

### Admin Centers
![](power-platform__admin-centers.png)

**Capabilities:**
- Quick links to Power Platform Admin Center
- Power Apps Maker Portal
- Power Automate Portal
- Power BI Admin Portal
- Dataverse environment URLs

---

## Microsoft Fabric

### Fabric Hub
![](fabric.png)

**Capabilities:**
- Microsoft Fabric workspaces
- Data warehouses and lakehouses
- Capacity management
- Workspace permissions
- Spark pool status

### Create Workspace
![](fabric__workspaces__create.png)

**Capabilities:**
- Create new Fabric workspaces
- Configure capacity assignment
- Set workspace permissions
- Define workspace description
- License mode selection

---

## Intune Management

### Autopilot Profiles
![](intune__autopilot-profiles.png)

**Capabilities:**
- Windows Autopilot deployment profiles
- OOBE (Out-of-Box Experience) customization
- Device group assignments
- Hybrid Azure AD join vs Azure AD join
- Self-deploying mode configuration

### Compliance Policies
![](intune__compliance-policies.png)

**Capabilities:**
- Device compliance policy management
- Platform-specific policies (Windows, iOS, Android, macOS)
- Compliance rules and settings
- Grace period configuration
- Non-compliance actions

### Device Configurations
![](intune__device-configurations.png)

**Capabilities:**
- Configuration profiles for all platforms
- Device restrictions
- Email and Wi-Fi profiles
- VPN and certificate profiles
- Custom OMA-URI settings

### Intune Applications
![](intune__applications.png)

**Capabilities:**
- Complete Intune app inventory from beta and v1.0 endpoints
- Android Managed Store apps
- iOS/iPadOS apps
- Windows apps (Win32, MSI, Store)
- Microsoft 365 Apps
- App installation requirements and dependencies
- Detailed app metadata and properties

---

## Audit & Monitoring

### Audit Logs
![](audit-logs.png)

**Capabilities:**
- Microsoft 365 audit log search
- User and admin activity tracking
- File and folder activities
- Sharing and access changes
- Export audit data

### Audit Log Queries
![](audit-log-queries.png)

**Capabilities:**
- Saved audit log queries
- Scheduled query execution
- Query result history
- Email notifications
- Compliance investigation support

### Provisioning Logs
![](provisioning-logs.png)

**Capabilities:**
- User provisioning activity
- Azure AD Connect sync status
- App provisioning logs
- Error troubleshooting
- Attribute mapping validation

### Service Principal Sign-Ins
![](service-principal-signins.png)

**Capabilities:**
- Monitor application authentication
- Service principal sign-in activity
- OAuth token issuance
- API access patterns
- Automated process monitoring

---

## Reports & Analytics

### Application Insights Exceptions
![](application-insights-exceptions.png)

**Capabilities:**
- Real-time application error tracking
- Exception stack traces
- Error frequency and trends
- Affected user count
- Custom telemetry

### Application Insights Exceptions (Detailed)
![](application-insights-exceptions2.png)

**Capabilities:**
- Detailed exception analysis
- Request correlation
- Dependency failures
- Performance impact
- Automated alerting

### Log Analytics
![](log-analytics.png)

**Capabilities:**
- KQL query builder for Azure Monitor Logs
- Query multiple Log Analytics workspaces
- Custom dashboards
- Saved queries
- Export and share results

### Known Issues
![](known-issues.png)

**Capabilities:**
- Microsoft 365 known issues tracker
- Service health impact
- Issue status and resolution timeline
- Workarounds and mitigations
- Affected services

### Service Health
![](service-health.png)

**Capabilities:**
- Real-time Microsoft 365 service status
- Active incidents and advisories
- Service availability history
- Maintenance notifications
- Impact assessment

### Service Health Details
![](service-health2.png)

**Capabilities:**
- Detailed incident information
- Root cause analysis
- Remediation steps and timelines
- Affected services and features
- Communication history
- Issue resolution tracking

### Service Announcements
![](service-announcements.png)

**Capabilities:**
- Message Center posts
- Feature updates and rollouts
- Action required notifications
- Change management planning
- Roadmap visibility

### Teams User Activity
![](teams__user-activity.png)

**Capabilities:**
- Microsoft Teams usage analytics
- Active users count
- Chat, calls, and meetings activity
- Device usage breakdown
- Trend analysis

### Office 365 Active User Counts
![](office365-active-user-counts.png)

**Capabilities:**
- Microsoft 365 service usage
- Active users by service (Exchange, SharePoint, OneDrive, Teams, Yammer)
- Activation trends over time
- License utilization metrics
- 30, 90, and 180-day historical views

---

## Applications

### Applications
![](applications.png)

**Capabilities:**
- Azure AD app registrations
- Application IDs and secrets
- API permissions granted
- Redirect URIs
- Certificate management

### Enterprise Applications
![](enterprise-applications.png)

**Capabilities:**
- Enterprise apps (service principals)
- SSO configuration
- User assignments
- Provisioning status
- Access reviews

### Service Principals
![](service-principals.png)

**Capabilities:**
- All service principals in tenant
- Application permissions
- Owner information
- Certificate/secret expiration
- Sign-in activity

### Application Management - App Protection Status
![](application-management__app-protection-status.png)

**Capabilities:**
- Intune app protection policy status
- Managed app count
- Policy compliance
- User enrollment status

### Application Management - Intune Apps
![](application-management__intune-apps.png)

**Capabilities:**
- All Intune-managed applications
- App installation status
- Required vs available apps
- Platform-specific apps (iOS, Android, Windows)
- App configuration policies

---

## Other Features

### Licensing
![](licensing.png)

**Capabilities:**
- Microsoft 365 license assignment
- Available licenses by SKU
- Consumed licenses
- Service plan details
- License usage trends

### NVD (National Vulnerability Database)
![](nvd.png)

**Capabilities:**
- CVE (Common Vulnerabilities and Exposures) search
- CVSS scores and severity
- Affected products and versions
- Patch availability
- Threat intelligence integration

### Learn Catalog
![](learn-catalog.png)

**Capabilities:**
- Microsoft Learn training modules
- Role-based learning paths
- Certification preparation
- Hands-on labs
- Learning progress tracking

---

**Generated**: November 23, 2025  
**Total Screenshots**: 75+  
**Mode**: Dark Mode  
**Resolution**: Optimized for 1920x1080
